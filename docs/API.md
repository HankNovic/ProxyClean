# 公共节点 API v1 使用文档

本文面向调用方，无需登录管理后台即可阅读。**阅读公开文档不等于 API 匿名开放**：下面三个接口都需要服务提供方签发的公共 API Key。

本仓库提供代理列表文件，不是 API 服务地址。请向服务提供方获取 API 基础地址和 Key；不要把 GitHub/raw 文件地址当作下面的 API 地址。仓库现有列表下载方式不变。

## 认证与安全

- 请求头：`Authorization: Bearer <API_KEY>`。
- 公共 API Key 与管理控制台登录凭据不是同一种凭据。不要向这些接口发送管理员令牌。
- 缺失、错误、过期或已撤销的 Key 均返回 HTTP 401。
- 默认每 Key 每分钟60次；实际额度由服务提供方设定。HTTP 429 的 Retry-After: 60 表示请等待60秒再重试。条件请求同样需要认证并计入调用次数。
- 使用 HTTPS；不要把 Key 放在 URL、公开仓库、Issue、截图或日志中。以下域名、Key和节点均为示例，不是可直接使用的服务或凭据。

~~~bash
BASE_URL='https://api.example.invalid'
PROXY_KEY='<API_KEY>'
curl --fail-with-body -H "Authorization: Bearer $PROXY_KEY" \
  "$BASE_URL/api/v1/proxies?page=1&limit=20"
~~~

## GET /api/v1/proxies

获取有效节点列表。查询参数如下；不需要请求体。

| 参数 | 类型/默认值 | 说明 |
|---|---|---|
| page | 整数，默认1 | 从1开始；小于1返回422 |
| limit | 整数，默认100 | 每页数量，至少1，不得超过服务上限；默认上限100，超限返回422 |
| country | 可选字符串 | 国家码精确匹配，例如US；接口不自动转大写 |
| asn | 可选字符串 | ASN精确匹配，例如64500，不加AS前缀 |
| max_latency | 可选数值 | 延迟上限，单位毫秒，包含等于上限的节点 |
| snapshot_id | 可选整数 | 固定首次响应的snapshot以继续分页；已不存在返回410 |

~~~bash
curl --fail-with-body --get -H "Authorization: Bearer $PROXY_KEY" \
  --data-urlencode 'country=US' --data-urlencode 'asn=64500' \
  --data-urlencode 'max_latency=800' --data-urlencode 'page=1' \
  --data-urlencode 'limit=20' "$BASE_URL/api/v1/proxies"
~~~

成功返回 HTTP 200、JSON。示意（数值仅用于说明格式）：

~~~json
{
  "snapshot": 12,
  "generated_at": 1700000000,
  "total": 1,
  "page": 1,
  "items": [{
    "id": "203.0.113.10:1080",
    "ip": "203.0.113.10",
    "port": 1080,
    "protocol": "socks5",
    "country": "US",
    "region": null,
    "city": null,
    "asn": "64500",
    "discovered_at": 1699990000,
    "verified_at": 1700000000,
    "latency_ms": 120,
    "valid_since": 1699999980,
    "valid_seconds": 20,
    "cumulative_valid_seconds": 300,
    "expires_at": 1700000300
  }]
}
~~~

- snapshot：快照标识；generated_at：生成时间；page：请求页；total：所选快照经过国家/ASN/延迟筛选后的数量，**不是本页items长度**。
- 后续页同时保持原筛选条件，并传入首次响应的snapshot_id。固定快照按原位置切页，再过滤目前失效/过期的节点，不把后页节点补到前页；因此items可能少于limit，甚至为空，total也不一定等于当前仍可用数量。
- 快照不是永久分页游标。目前最多保留最近21份快照；收到410应丢弃旧游标并从第一页获取新快照，不要无限重试旧ID。
- 当前没有结果时items为空数组；没有任何快照时snapshot为0。不要将0作为后续固定快照游标。

### 节点字段和有效性口径

| 字段 | 类型/含义 |
|---|---|
| id / ip / port | 节点标识、IP、端口；单节点查询使用返回的完整id |
| protocol | 当前为socks5 |
| country / region / city / asn | 地理与ASN信息；未知时可能为null，不能据此编造位置 |
| discovered_at | 首次发现时间 |
| verified_at | 满足发布条件的所需成功证据中最早的检测时间 |
| latency_ms | 所需成功证据中最大的端到端耗时，单位毫秒；不是本次API响应耗时，不包含排队与失败重试 |
| valid_since / valid_seconds | 当前连续有效观测段的起点 / 已观察时长 |
| cumulative_valid_seconds | 累计有效观测时长，单位秒 |
| expires_at | 当前证据的失效时间 |

所有时间戳均为Unix秒，可以包含小数；时长为秒，延迟为毫秒。有效时长是周期检测的观测估计，不保证间隔内每秒可用，也不保证未来可用。不要缓存节点超过expires_at；每次接口读取会过滤过期结果。

## GET /api/v1/proxies/{node_id}

查询仍在当前发布结果中的一个节点，成功HTTP 200，JSON对象字段与列表的单项相同；不存在或已失效返回404。

~~~bash
curl --fail-with-body -H "Authorization: Bearer $PROXY_KEY" \
  "$BASE_URL/api/v1/proxies/203.0.113.10:1080"
~~~

请使用实际列表返回的id，并在客户端按URL路径正确编码，不将示例节点当作真实可用节点。

## GET /api/v1/exports

返回HTTP 200、text/plain; charset=utf-8，逐行socks5://IP:PORT，非空结果以换行结束；无节点时正文为空。这是API导出格式，不改变本仓库既有列表文件格式。

~~~text
socks5://203.0.113.10:1080
socks5://203.0.113.20:1080
~~~

~~~bash
curl --fail-with-body -D response-headers.txt \
  -H "Authorization: Bearer $PROXY_KEY" "$BASE_URL/api/v1/exports"
curl --fail-with-body -H "Authorization: Bearer $PROXY_KEY" \
  -H 'If-None-Match: "<PREVIOUS_ETAG>"' "$BASE_URL/api/v1/exports"
~~~

首次保存响应ETag；后续原样发送If-None-Match（包括双引号）。内容未变返回304、空正文及ETag。200响应包含Cache-Control: private, no-cache；再次使用缓存前应重新验证，不把旧导出视为持续可用证明。

## 错误响应

| HTTP状态 | 响应/处理 |
|---|---|
| 401 | {"error":"invalid_api_key"}；检查是否使用公共Key、是否过期/撤销；不要改用管理员凭据 |
| 404 | {"detail":"Not Found"}；单节点未在当前发布结果中 |
| 410 | {"detail":"快照已回收，请重新分页"}；重新从第一页获取快照 |
| 422 | 分页越界时detail为“分页参数超限”；类型解析失败时detail为字段错误数组；修正参数，不重试相同无效请求 |
| 429 | {"error":"rate_limited"}；遵循Retry-After: 60，避免并发重试风暴 |
| 500 | 服务端异常；联系服务提供方，避免无限重试 |

HTTP受理、有效结果及未来可用性是不同概念。客户端应先检查状态码和Content-Type，再解析正文；304没有JSON正文。

## Python（标准库）示例

~~~python
import json
import os
import urllib.request

base_url = os.environ['PROXY_API_BASE_URL'].rstrip('/')
request = urllib.request.Request(
    base_url + '/api/v1/proxies?page=1&limit=20',
    headers={'Authorization': 'Bearer ' + os.environ['PROXY_API_KEY']},
)
with urllib.request.urlopen(request, timeout=15) as response:
    result = json.load(response)
for node in result['items']:
    print(node['id'], node['latency_ms'], node['expires_at'])
~~~

请妥善处理HTTP错误和超时；不要输出请求认证头。API结果仅是当时的检测观测，使用免费代理时不要传输敏感信息。
