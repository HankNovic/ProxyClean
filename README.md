# ProxyClean

本仓库用于公开可用 SOCKS5 代理数据及相关说明。

## 公开内容

仓库中维护的 `good_socks.txt` 文件，提供一份**实时清洗、持续保活、按延迟排序**的国内可用 Socks5 代理列表。

所有代理均来自公开免费源，本项目只负责聚合、筛选和保活，不出售、不提供任何付费服务。

> 测试出口接口：`https://api.ipify.org`  
> 测试节点：**中国 福建 厦门 移动**

## 代理池特点

- **国内可用优先**：在中国大陆电信环境下实测连通，通过后才会写入 `good_socks.txt`。
- **多源聚合**：从多个公开免费 Socks5 列表抓取，去重后集中清洗。
- **并发测速**：通过并发请求测试代理连通性和延迟，并按延迟从低到高排序。
- **持续保活**：在两次大规模拉取之间，定期对现有代理做包活检测，只保留仍存活的节点。
- **自动滚动更新**：过期/失效代理会被自然淘汰，`good_socks.txt` 始终尽量保持为“当前时刻可用”的集合。

生成和保活的逻辑由仓库内脚本自动完成，普通使用者只需要关注 **如何使用 `good_socks.txt`** 即可。

## 代理来源（致谢引用）

目前主要聚合和清洗以下公开 Socks5 源（排名不分先后）：

- [proxifly/free-proxy-list](https://github.com/proxifly/free-proxy-list)
- [TheSpeedX/PROXY-List](https://github.com/TheSpeedX/PROXY-List)
- [roosterkid/openproxylist](https://github.com/roosterkid/openproxylist)
- [hookzof/socks5_list](https://github.com/hookzof/socks5_list)
- [gfpcom/free-proxy-list](https://github.com/gfpcom/free-proxy-list)

如果你是上述项目的维护者，感谢你对社区的贡献。  
如有收录/引用上的需求或问题，可以在本仓库提 Issue 进行沟通。

> 说明：本项目不会对任何单个源做“质量背书”，仅基于当前探测结果进行技术层面的存活筛选。

---

## 代理池文件格式

代理池文件为 `good_socks.txt`，位于仓库根目录，格式为：

```text
ip:port
```

- 每行一个 Socks5 代理。
- IP 和端口之间使用英文冒号 `:` 分隔。
- 不包含用户名/密码（即匿名/公开免费代理）。

在客户端或程序中使用时，协议一般选择 `socks5` 或 `socks5h`。

---

## 如何使用本项目的代理池

以下示例假设你已经拿到 `good_socks.txt`，并手动选择或程序中读取其中一行，例如：

```text
1.2.3.4:1080
```

### 在命令行中使用（以 curl 为例）

```bash
curl --socks5 1.2.3.4:1080 https://api.ipify.org
```

如果你希望域名解析也走代理，可以使用：

```bash
curl --socks5-hostname 1.2.3.4:1080 https://api.ipify.org
```

### 在 Linux 下使用 Socks5 代理

在大多数 Linux 发行版上，可以通过**环境变量**或工具自身的配置来使用 `good_socks.txt` 里的 Socks5 代理。

- **方式一：使用环境变量（影响当前终端会话）**

```bash
export ALL_PROXY="socks5://1.2.3.4:1080"
# 或者分别设置 HTTP / HTTPS
export http_proxy="socks5://1.2.3.4:1080"
export https_proxy="socks5://1.2.3.4:1080"

# 之后直接使用 curl / wget 等命令即可：
curl https://api.ipify.org
```

- **方式二：git 等命令行工具单独配置**

```bash
git config --global http.proxy "socks5://1.2.3.4:1080"
git config --global https.proxy "socks5://1.2.3.4:1080"
```

如果需要取消代理配置，可执行：

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### 在 Python 中使用（requests 示例）

```python
import requests

proxy = "1.2.3.4:1080"
proxies = {
    "http": f"socks5://{proxy}",
    "https": f"socks5://{proxy}",
}

resp = requests.get("https://api.ipify.org", proxies=proxies, timeout=10)
print(resp.text)
```

> 注意：在使用 `requests` 前需要安装 `requests[socks]` 或 `pysocks` 等相关依赖。

### 在常见客户端中使用

- **Clash / Clash.Meta**：在代理节点中新增一个 `type: socks5` 的节点，`server` 填 IP，`port` 填端口，然后在规则或直连/代理配置中使用该节点。
- **v2rayN / Qv2ray 等**：添加 Socks 代理节点或本地转发，指向 `ip:port`。
- **浏览器插件（如 SwitchyOmega）**：配置一个 Socks5 代理情景模式，填入代理 IP 和端口即可。

> 强烈建议：将 `good_socks.txt` 中的代理视为 **易波动资源**，在实际业务中做失败重试、轮询、健康检查等逻辑。

---

## 更新频率与可用性说明

- **拉取频率**：脚本会定期（默认约每小时）从多个源重新拉取完整代理列表，并重新清洗。
- **保活检测**：在两次大规模拉取之间，对已有代理做周期性包活检测，连续成功多次才视为真正存活。
- **可用性**：`good_socks.txt` 只能代表**当前或最近一段时间**的探测结果，不能保证长期稳定可用。

因此：

- 适合用作**临时性抓取 / 测试环境 / 学习研究**的代理池；
- 不建议直接用作关系到账号安全、资金安全、合规要求较高的生产环境出口。

---

## 风险声明与使用须知

- 代理来源于公开网络，不保证来源真实性、合规性与安全性。
- 使用代理可能带来隐私泄露、账号风控、数据泄露等安全风险，请务必自行评估。
- 请遵守所在地区的法律法规，不得将本项目及其产生的代理池用于任何违法违规用途。
- 建议仅将本项目用于协议学习、网络调试与研究测试等用途。

如果你希望自行搭建稳定、可控的代理环境，请优先选择自建节点或合规的商业服务。