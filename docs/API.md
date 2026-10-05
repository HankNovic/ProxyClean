# 公共节点 API v1 使用文档

本 API 提供 SOCKS5 节点列表、单节点信息和纯文本导出。你可以按国家、ASN、检测延迟筛选节点，读取节点的检测时间及到期时间，或将导出列表交给自己的代理客户端。

**本仓库的 GitHub/raw 地址是文件下载地址，不是 API 地址。** 是否提供 API、可访问的基础地址、Key、调用额度及允许的来源 IP，请向服务提供者确认。本文不表示存在某个已经上线的公共域名或自助申请渠道。[仓库文件下载方式](../README.md)不变，也不要求为了读文档申请 Key。

## 1. 获取地址与 Key

1. 向提供者取得 API 基础地址，例如将占位地址换成你的实际 HTTPS 地址。
2. 取得为你签发的 API Key，并确认期限、额度和来源 IP 要求。Key 不能从本文、节点文件或 GitHub 地址推导出来。
3. 每次请求都在请求头中发送下面的认证信息，不放到 URL、查询参数或正文中：

~~~text
Authorization: Bearer <API_KEY>
~~~

使用为本 API 签发的 Key，而不是其他系统的登录凭据。原有 Key 继续使用同一认证格式；只要仍获允许且未到期、停用或撤销，不因文档更新而要求换 Key。需要补发、续期或调整权限时联系提供者。

以下 Bash 设置会从终端隐藏输入 Key，避免把明文写进命令历史；不要开启 shell 命令跟踪，不要打印认证头或分享带凭据的截图。示例地址、节点均用于说明，不能直接作为可用服务。

~~~bash
export API_BASE_URL='https://api.example.invalid'
read -r -s -p 'API Key: ' PROXY_KEY
printf '\n'
export PROXY_KEY
~~~

所有接口只用 GET，无需请求体。使用准确的服务地址，不将带 Key 的请求自动重定向到其他地址。不要并发猛刷或反复重试被拒绝的 Key。

## 2. GET /api/v1/proxies — 节点列表

### 查询参数

| 参数 | 类型 / 默认值 | 行为 |
|---|---|---|
| page | 整数 / 1 | 从 1 开始；小于 1 返回 422。超过页数返回 200 和空 items，不是 404。 |
| limit | 整数 / 100 | 至少 1，且不超过提供者设置的上限。上限若低于 100，必须显式传入允许的 limit，否则默认请求也会 422。 |
| country | 可选字符串 | 精确匹配返回的国家码，如 US；区分大小写，不自动转换。空字符串等同不筛选。 |
| asn | 可选字符串 | 精确匹配，如 64500；不要自行添加 AS 前缀。空字符串等同不筛选。 |
| max_latency | 可选数字 | 检测延迟上限，单位毫秒，包含等于上限的节点。请传有限数值；负数不按范围错误拒绝，而会得到空匹配。 |
| snapshot_id | 可选整数 | 固定已返回的 snapshot。不存在或已回收的标识返回 410，包括 0、负数；非整数返回 422。 |

多个筛选条件同时满足才返回。分页期间保持相同的 limit 和筛选条件，不要依赖未列出的参数。

~~~bash
curl --fail-with-body --get -H "Authorization: Bearer $PROXY_KEY" \
  --data-urlencode 'page=1' --data-urlencode 'limit=1' \
  "$API_BASE_URL/api/v1/proxies"
~~~

筛选时可在此命令加入 --data-urlencode 'country=US'、--data-urlencode 'asn=64500' 或 --data-urlencode 'max_latency=800'。

### 成功响应

HTTP 200，application/json。下面的地址、时间和快照编号仅说明响应格式；请使用实际响应中的值。

~~~json
{
  "snapshot": 1,
  "generated_at": 1791155241.417055,
  "items": [
    {
      "id": "203.0.113.10:1080",
      "ip": "203.0.113.10",
      "port": 1080,
      "protocol": "socks5",
      "country": "US",
      "region": null,
      "city": null,
      "asn": "64500",
      "discovered_at": 1791155141.4130058,
      "verified_at": 1791155231.4130058,
      "latency_ms": 120.0,
      "valid_since": 1791155221.4130058,
      "valid_seconds": 20.156752109527588,
      "cumulative_valid_seconds": 300.1567521095276,
      "expires_at": 1791155841.4130058
    }
  ],
  "total": 4,
  "page": 1
}
~~~

| 字段 | 含义 |
|---|---|
| snapshot | 结果版本的整数标识；后续请求使用 snapshot_id 参数。0 表示尚无发布快照，不可用作固定分页标识。 |
| generated_at | 所选快照的生成时间，不是每次读取时间；snapshot=0 时是本次生成空响应的时间。 |
| total | 所选结果经筛选后的数量，不是本页条数；固定快照时还可能包含现在已失效的项，详见下一节。 |
| page | 本次请求的页码。 |
| items | 本页节点对象数组，可以为空。响应不附带 limit 或下一页 URL。 |

### 固定分页与过期：不要遇到空页就停止

不传 snapshot_id 时，读取当前结果，先去除到期节点、再筛选分页；不同请求间结果和位置可能变化。

传入 snapshot_id 时，使用该快照的字段和原始位置：**先筛选并按原位置切页，再移除不在当前结果中或该旧记录已到期的节点，不从后页补齐。** 同一个节点即使重新出现在当前结果中，旧快照记录自己的 expires_at 也不会被延长。固定页中的延迟、位置等字段仍属于旧快照，不保证等于最新单节点响应。

可靠的遍历方式：

1. 不固定快照请求一次，取得 snapshot；若为 0，本次没有节点，结束。
2. 用该标识和同样筛选条件，**重新请求固定快照的第 1 页**。用这次固定响应的 total 计算总页数，不沿用首次非固定响应的 total：前者可能包含已到期的原位置，因此可能更大。
3. 遍历到 ceil(total / limit) 页。某页 items 少于 limit，甚至整页为空，都不表示没有后续节点。
4. 收到 410 时丢弃旧快照标识，重新从第 1 步开始；应限制重新开始的次数，不能无限重试同一个失效标识。

历史快照仅暂时保留，不承诺固定保留时长。正常空结果是 200：既可能 snapshot=0，也可能是非零快照且 items=[]；暂时无法提供可用结果是 503，不能当成空列表处理。

### 节点字段与时间

| 字段 | 类型 / 含义 |
|---|---|
| id | 字符串，完整节点标识，例如 203.0.113.10:1080 或 [2001:db8::40]:1080。单节点查询原样使用此标识并进行 URL 编码。 |
| ip / port | IP 字符串 / 整数端口；IPv6 的 ip 本身不含方括号。 |
| protocol | 当前为 socks5。 |
| country / region / city / asn | 字符串或 null。未知信息用 null 表示，不要将其当成某个国家或 ASN。 |
| discovered_at | 首次发现时间。 |
| verified_at | 这条结果依据的一组成功检测中最早的时间，不是本次 API 读取时间。 |
| latency_ms | 节点检测耗时；涉及多项检测时为其中最大值。单位毫秒，不是 API 响应耗时或带宽，不含排队和失败重试时间。 |
| valid_since / valid_seconds | 当前连续有效观测段的起点 / 截至读取时的时长。 |
| cumulative_valid_seconds | 包含当前观测段的累计有效时长。 |
| expires_at | 这条记录的到期时间；超过此时刻不要继续当作有效缓存使用。 |

时间戳是 Unix 秒，可含小数；时长为秒，延迟为毫秒。有效时长来自周期检测的观测，不代表间隔中每一秒都可用，也不保证未来可用。位置数据可能不准确；显示或再分发位置/ASN 派生数据时保留 [DB-IP](https://db-ip.com/) 的 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 归属说明。

## 3. GET /api/v1/proxies/{node_id} — 单节点

返回当前结果中仍未到期的一个节点，HTTP 200，JSON 对象字段与列表的一项相同。不存在、已移出当前结果或已到期返回 404；不能用此接口读取历史版本。

~~~bash
curl --fail-with-body -H "Authorization: Bearer $PROXY_KEY" \
  "$API_BASE_URL/api/v1/proxies/203.0.113.10:1080"
~~~

请换成列表实际返回的 id；在程序中对整个 id 使用 URL 路径编码，例如 Python 的 quote(node_id, safe='')。IPv6 标识中的方括号和冒号也应编码。不需要请求体或查询参数。

## 4. GET /api/v1/exports — 文本导出

HTTP 200，text/plain; charset=utf-8，每行一个 socks5:// 节点地址，以换行结尾；没有结果时正文为空。没有筛选参数，想按国家或延迟筛选请使用列表接口。此导出没有国家注释，也没有每个节点的 expires_at，不要将它等同于仓库中的注释版文件。

~~~bash
curl --fail-with-body -H "Authorization: Bearer $PROXY_KEY" \
  "$API_BASE_URL/api/v1/exports"
~~~

内容格式：

~~~text
socks5://203.0.113.20:1080
socks5://203.0.113.30:1080
socks5://[2001:db8::40]:1080
~~~

200 响应带有 ETag 和 Cache-Control: private, no-cache。下次可将收到的完整 ETag（包括引号）原样放入 If-None-Match 请求头：内容没变化返回 304，正文为空、仍带 ETag；使用你为该地址保存的原正文，不要用空的 304 正文覆盖它。只有导出支持此条件请求。

条件请求仍需有效 Key，也计入额度，可能返回 401、403、429 或 503。304 不延长 Key 或节点期限；导出不含逐节点到期时间，使用缓存前需向服务重新校验，不能无限离线复用。需要自己管理节点缓存期限时使用 JSON 列表的 expires_at。

## 5. 错误、访问限制与重试

先判断 HTTP 状态码，再读取错误正文；不要假设所有错误都有同一个 JSON 字段。以下是接口返回的形状，提供者的网络入口也可能返回非 JSON 错误。

| 状态 | 常见正文 | 调用方应如何处理 |
|---|---|---|
| 401 | {"error":"invalid_api_key"} | Key 缺失、错误、到期、停用或撤销，或当前不再允许该调用者访问。检查 Bearer 头，联系提供者；不要自动循环重试。 |
| 403 | {"error":"ip_not_allowed"}；来源信息无效时可能为 {"detail":"invalid_forwarded_address"} | 检查实际出口 IP 是否获准，必要时联系提供者；不要伪造来源头绕过限制。 |
| 404 | {"detail":"Not Found"} | 单节点已不可用；未知路径或非 GET 方法也可能返回此状态。检查路径并重新获取列表。 |
| 410 | {"detail":"快照已回收，请重新分页"} | 停止使用当前 snapshot_id，重新开始分页。 |
| 413 | {"error":"request_too_large"} | 缩短路径、查询或请求头，修正请求后再发。 |
| 422 | {"detail":"分页参数超限"} 或 {"detail":[…]} | 修正类型、页码或每页上限；校验详情内容可能因参数而异。 |
| 429 | {"error":"rate_limited"}，Retry-After: 60 | 等待至少响应头给出的秒数，减少并发/调用频率；避免多客户端同时重试。 |
| 503 | {"error":"replica_unavailable"}，Retry-After: 15，Cache-Control: no-store | 服务暂时不能提供可用结果。保留“暂不可用”状态，不写成空列表、不缓存该错误；按响应头延迟并有限重试，持续失败联系提供者。 |

请求预算：查询字符串不超过 2048 字节；请求头名称和值合计不超过 8192 字节；路径不超过 256 个字符。其他网络入口可能有更严格的限制。不要发送与这三个 GET 无关的大请求。

提供者可设置单 Key、共享调用额度、来源 IP 及整体请求限制；向提供者确认实际额度。429 不一定表示仅你自己的 Key 超额；不要通过多 Key 绕过共享额度。对于 429/503，遵循 Retry-After，可加随机抖动并限制次数；对于网络超时也应退避，而非无间隔重试。

每次响应才是这次调用是否获准的依据，过去的 200 不代表以后仍有权限。服务不可用时可能先返回 503，不能仅凭此状态认定 Key 有效或无效。不要通过缓存旧结果来掩盖拒绝访问或持续故障。

## 6. Python 标准库示例：正确遍历固定快照

先设置上述 API_BASE_URL、PROXY_KEY。API_PAGE_SIZE 可设为提供者允许的每页数量，默认 20。本例拒绝自动重定向；429/503 最多重试两次，410 最多重新开始一次；不会把空的中间页误当成结束，也不打印 Key。最终只输出尚未到期的节点。

~~~python
import json
import os
import time
from urllib.error import HTTPError
from urllib.parse import urlencode
from urllib.request import HTTPRedirectHandler, Request, build_opener


class NoRedirect(HTTPRedirectHandler):
    def redirect_request(self, request, response, code, message, headers, new_url):
        return None


base_url = os.environ['API_BASE_URL'].rstrip('/')
api_key = os.environ['PROXY_KEY']
page_size = int(os.environ.get('API_PAGE_SIZE', '20'))
opener = build_opener(NoRedirect())


def read_page(parameters):
    request = Request(
        base_url + '/api/v1/proxies?' + urlencode(parameters),
        headers={'Authorization': 'Bearer ' + api_key},
    )
    for attempt in range(3):
        try:
            with opener.open(request, timeout=15) as response:
                return json.load(response)
        except HTTPError as error:
            if error.code not in (429, 503) or attempt == 2:
                raise
            time.sleep(max(1, int(error.headers.get('Retry-After', '15'))))


def collect_nodes():
    for restart in range(2):
        try:
            parameters = {'page': 1, 'limit': page_size}
            first = read_page(parameters)
            if first['snapshot'] == 0:
                return []
            parameters['snapshot_id'] = first['snapshot']
            pinned = read_page(parameters)
            nodes = list(pinned['items'])
            pages = (pinned['total'] + page_size - 1) // page_size
            for number in range(2, pages + 1):
                parameters['page'] = number
                nodes.extend(read_page(parameters)['items'])
            return [node for node in nodes if node['expires_at'] > time.time()]
        except HTTPError as error:
            if error.code != 410 or restart == 1:
                raise


if __name__ == '__main__':
    try:
        print(json.dumps(collect_nodes(), ensure_ascii=False, indent=2))
    except HTTPError as error:
        raise SystemExit('HTTP ' + str(error.code) + '：请按文档处理错误，不要无限重试')
~~~

采集结果不保证代理安全或持续在线。将节点用于实际请求前仍需考虑目标可达性、超时和隐私风险，不要通过不可信免费代理传输敏感信息。
