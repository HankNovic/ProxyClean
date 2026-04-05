# ProxyClean

公开整理并持续更新的 SOCKS5 代理数据仓库。

## 数据概览

- 代理池文件：`SOCKS5.txt`
- 原始数据文件：`SOCKS5_RAW.txt`
- 原始直链（可用池）：https://raw.githubusercontent.com/HankNovic/ProxyClean/refs/heads/main/SOCKS5.txt
- 原始直链（原始池）：https://raw.githubusercontent.com/HankNovic/ProxyClean/refs/heads/main/SOCKS5_RAW.txt
- 连通性验证：SOCKS5 握手 + SOCKS5 CONNECT 目标探测
- 默认 CONNECT 探测目标：`1.1.1.1:53`、`8.8.8.8:53`
- 测试节点：**中国 福建 厦门 移动**

说明：
- 列表中的代理来自公开免费来源。
- 本仓库仅做聚合、清洗、保活与公开，不提供任何付费服务。

## 代理池特点

- **国内可用优先**：在中国大陆网络环境下进行连通性验证，通过后才进入可用池。
- **多源聚合清洗**：汇总多个公开 SOCKS5 源，统一解析、去重、筛选。
- **并发验证排序**：并发执行连通与延迟验证，输出优先近期可用、延迟表现更稳定的节点。
- **真实可用性校验**：握手通过后可追加 SOCKS5 CONNECT 目标探测，只有握手与 CONNECT 同时通过才判定成功。
- **统一错误策略**：握手与 CONNECT 共用软/硬错误分类与去重聚合，降低短时网络抖动带来的误判。
- **持续保活更新**：在周期性全量拉取之间进行保活检查，自动淘汰失效节点。
- **滚动维护**：`SOCKS5.txt` 持续更新，尽量反映近期可用状态。

## 代理来源（致谢）

当前主要聚合以下公开 SOCKS5 源（排名不分先后）：

- [proxifly/free-proxy-list](https://github.com/proxifly/free-proxy-list)
- [TheSpeedX/PROXY-List](https://github.com/TheSpeedX/PROXY-List)
- [roosterkid/openproxylist](https://github.com/roosterkid/openproxylist)
- [hookzof/socks5_list](https://github.com/hookzof/socks5_list)
- [gfpcom/free-proxy-list](https://github.com/gfpcom/free-proxy-list)
- [tomcat1235.nyc.mn/proxy_list](https://tomcat1235.nyc.mn/proxy_list)

> 说明：本仓库不对任一单独来源做质量背书，仅依据实际探测结果进行技术筛选。

## 文件格式

`SOCKS5.txt` 与 `SOCKS5_RAW.txt` 均采用纯文本格式：

```text
ip:port
```

- 每行一个 SOCKS5 代理。
- 使用英文冒号 `:` 分隔 IP 与端口。
- 默认不包含用户名/密码。
- 客户端通常使用 `socks5` 或 `socks5h` 协议。

## 使用方式

### 1) 读取代理

从 `SOCKS5_RAW.txt` 按行读取，例如：

```text
1.2.3.4:1080
```

### 2) 命令行测试（curl）

```bash
curl --socks5 1.2.3.4:1080 https://api.ipify.org
```

如需让域名解析也走代理：

```bash
curl --socks5-hostname 1.2.3.4:1080 https://api.ipify.org
```

### 3) Linux 环境变量方式

```bash
export ALL_PROXY="socks5://1.2.3.4:1080"
export http_proxy="socks5://1.2.3.4:1080"
export https_proxy="socks5://1.2.3.4:1080"

curl https://api.ipify.org
```

### 4) Git 单独代理配置

```bash
git config --global http.proxy "socks5://1.2.3.4:1080"
git config --global https.proxy "socks5://1.2.3.4:1080"
```

取消配置：

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### 5) Python（requests）

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

> 使用 `requests` 的 SOCKS 代理能力前，请安装 `requests[socks]` 或 `pysocks`。

### 6) 常见客户端

- **Clash / Clash.Meta**：新增 `type: socks5` 节点，填写 `server` 与 `port`。
- **v2rayN / Qv2ray**：新增 Socks 节点或本地转发，指向 `ip:port`。
- **SwitchyOmega 等浏览器插件**：新建 SOCKS5 情景模式并填入 IP/端口。

建议在实际使用中加入失败重试、轮询与健康检查机制。

## 更新与可用性

- 全量拉取与清洗：默认约每小时进行一次。
- 间隔保活检测：在两次全量之间进行周期性存活验证。
- 网络守护：在线探测开关；网络探测异常时可暂停当轮验证，恢复后自动继续。
- 历史清理：按保留天数（30/15/7/1）进行超时清理，覆盖历史统计与运行日志保留策略。
- 可用性说明：`SOCKS5.txt` 仅代表当前或近期探测结果，不保证长期稳定。

适用场景：临时抓取、测试环境、学习研究。  
不建议直接用于高合规、高稳定性要求的生产出口。

## 风险声明与使用须知

- 代理来源于公开网络，不保证来源真实性、合规性与安全性。
- 使用代理可能带来隐私泄露、账号风控、数据泄露等风险，请自行评估。
- 请严格遵守所在地法律法规及目标平台服务条款，不得用于违法违规用途。
- 建议仅用于协议学习、网络调试与研究测试。
