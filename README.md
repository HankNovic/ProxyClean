# ProxyClean

公开 SOCKS5 代理列表仓库，可用于协议学习、网络调试及测试环境。文件中的节点不保证持续可用或安全，不能据此承诺固定更新频率、探测地点或适合特定生产用途。

## 文件下载与 API

| 文件 | 用途 | 下载地址 |
|---|---|---|
| SOCKS5.txt | 供代理客户端识别、导入的发布列表 | https://raw.githubusercontent.com/HankNovic/ProxyClean/refs/heads/main/SOCKS5.txt |
| SOCKS5_RAW.txt | 便于逐行读取、转换的文本列表 | https://raw.githubusercontent.com/HankNovic/ProxyClean/refs/heads/main/SOCKS5_RAW.txt |

[公共节点 API v1 使用文档](docs/API.md)说明如何认证、读取三个 GET 接口、筛选和固定快照分页、处理到期、拒绝访问、限流及暂不可用错误，并提供 curl/Python 示例。

文件下载地址不是 API 地址。若需要 API，请向服务提供者获取实际基础地址、独立 Key 和访问限制；本仓库不表示已有某个对所有人开放的 API 域名或自助申请渠道。文件下载方式不因 API 文档更新而改变。

## 读取文件

文件是代理清单，不是包含账号、节点期限或全部历史数据的数据库。不同发布内容可能使用裸地址、SOCKS5 URI 或行尾注释；请以实际文件内容为准，不要把文件名中的 RAW 理解为全部未处理的来源数据。

可见的形式例如：

~~~text
203.0.113.10:1080
203.0.113.10:1080 #US
socks5://203.0.113.10:1080 #US
socks5://[2001:db8::40]:1080 #JP
~~~

这些是说明用地址，不是真实可用节点。读取时去除空行和 # 注释；只导入代理地址，不将国家注释当成密码或地址的一部分。IPv6 地址需保留方括号。

不要假定两份文件永远使用相同的前缀或注释格式，也不要仅凭列表顺序推断延迟排名。国家信息可能未知或不准确。

API 的 /api/v1/exports 则是没有国家注释的 socks5:// 地址列表，支持条件请求；JSON 接口还提供每个节点的到期时间。具体行为以[API 文档](docs/API.md)为准，不混用文件格式与接口响应格式。

## 使用代理结果

先将下面的说明用地址替换为你实际取得的节点，并检查其是否仍可用。API Key 只用于读取 API，**不是代理服务器的用户名或密码**，不要发送给代理节点。

### curl

~~~bash
curl --max-time 15 --socks5-hostname 203.0.113.10:1080 https://api.ipify.org
~~~

如果希望域名由本机解析，可改用 curl 的 --socks5 参数。上面的请求是使用代理访问目标站点，不是 API 请求。

### 环境变量

只在需要使用代理的终端中设置，完成后取消，不要把未知代理设置成整个系统的长期出口：

~~~bash
export ALL_PROXY='socks5://203.0.113.10:1080'
curl --max-time 15 https://api.ipify.org
unset ALL_PROXY
~~~

### 其他客户端

在支持 SOCKS5 的客户端中填写节点的 IP 和端口；URI 列表可按客户端支持的方式导入。为真实请求设置超时并处理失败，不要因为某次检测或一次导入成功就永久信任节点。文件及 API 列表都不是所有客户端通用的完整订阅配置。

## 更新、时效与归属

- 文件内容和更新时间以仓库中的实际文件及提交记录为准；本文不承诺每小时更新、固定排序、探测地区或特定目标可达性。
- 文件不会在你下载后自动失效；下载时间不能替代节点检测或 API 中的 expires_at。节点可能很快失效，重新获取列表也不能保证未来可用。
- 需要按国家、ASN、延迟筛选，或判断单节点期限时，使用带这些字段的 API 响应；详见文档中的空结果、固定分页空洞、410、429 与 503 处理。
- 位置/ASN 派生信息使用 [DB-IP](https://db-ip.com/) 数据时，应保留其 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) 归属说明；地理信息不代表连接来源的安全保证。

## 相关公开代理项目

感谢公开分享代理列表的项目。以下链接供查阅，不表示每份当前文件均包含各项目的节点，也不构成质量或可用性背书：

- [proxifly/free-proxy-list](https://github.com/proxifly/free-proxy-list)
- [TheSpeedX/PROXY-List](https://github.com/TheSpeedX/PROXY-List)
- [roosterkid/openproxylist](https://github.com/roosterkid/openproxylist)
- [hookzof/socks5_list](https://github.com/hookzof/socks5_list)
- [gfpcom/free-proxy-list](https://github.com/gfpcom/free-proxy-list)
- [dpangestuw/Free-Proxy](https://github.com/dpangestuw/Free-Proxy)

## 风险与使用须知

公开免费代理不保证来源真实性、安全性、合规性或稳定性，可能带来隐私泄露、账号风控和数据泄露风险。不要传输敏感信息；遵守所在地法律和目标平台服务条款，不用于违法违规行为。不建议直接作为高稳定性、高合规要求的生产出口。
