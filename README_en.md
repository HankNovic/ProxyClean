# ProxyClean

<!-- hy-mt2-i18n:start -->
[中文](./README.md) | **English** | [日本語](./README_ja.md) | [Español](./README_es.md)
<!-- hy-mt2-i18n:end -->


A publicly compiled and continuously updated repository of SOCKS5 proxy data.

## Data Overview

- Proxy pool file: `SOCKS5.txt`
- Raw data file: `SOCKS5_RAW.txt`
- Direct link to the available pool: https://raw.githubusercontent.com/HankNovic/ProxyClean/refs/heads/main/SOCKS5.txt
- Direct link to the raw pool: https://raw.githubusercontent.com/HankNovic/ProxyClean/refs/heads/main/SOCKS5_RAW.txt
- Connectivity verification: SOCKS5 handshake + SOCKS5 CONNECT target test
- Default CONNECT test targets: `1.1.1.1:53`, `8.8.8.8:53`
- Test node: **Xiamen, Fujian, China – China Telecom**

Notes:  
- The proxies listed come from public and free sources.  
- This repository is solely for aggregation, cleaning, maintenance, and public release; it does not offer any paid services.

## Features of the Proxy Pool

- **Prioritize domestically available proxies**: Conduct connectivity tests in the Chinese mainland network environment, and only proxies that pass these tests are included in the available pool. 
- **Multi-source aggregation and cleaning**: Combine multiple public SOCKS5 sources, then parse them uniformly, remove duplicates, and filter the results. 
- **Concurrent verification and sorting**: Execute connectivity and latency tests in parallel, outputting nodes that are currently available and exhibit more stable latency performance first. 
- **Actual availability validation**: After a successful handshake, additional SOCKS5 CONNECT target tests are performed; a proxy is considered valid only if it passes both tests. 
- **Unified error handling strategy**: Use the same soft/hard error classification and duplicate removal mechanisms for handshakes and CONNECT tests to reduce false positives caused by short-term network fluctuations. 
- **Ongoing keep-alive updates**: Perform keep-alive checks between periodic full data pulls to automatically eliminate invalid proxies. 
- **Continuous maintenance**: The `SOCKS5.txt` file is updated regularly to reflect the most up-to-date availability status.

## Proxy Sources (Acknowledgments)

Currently, it primarily aggregates the following public SOCKS5 sources (in no particular order):

- [proxifly/free-proxy-list](https://github.com/proxifly/free-proxy-list)
- [TheSpeedX/PROXY-List](https://github.com/TheSpeedX/PROXY-List)
- [roosterkid/openproxylist](https://github.com/roosterkid/openproxylist)
- [hookzof/socks5_list](https://github.com/hookzof/socks5_list)
- [gfpcom/free-proxy-list](https://github.com/gfpcom/free-proxy-list)
- [dpangestuw/Free-Proxy](https://github.com/dpangestuw/Free-Proxy)

> Note: This repository does not endorse the quality of any individual source; it only performs technical filtering based on actual testing results.

## File Format

Both `SOCKS5.txt` and `SOCKS5_RAW.txt` use plain text format:

IP:Port

- One SOCKS5 proxy per line.
- Use a colon `:` to separate the IP address from the port.
- Username/passwords are not included by default.
- Clients typically use the `socks5` or `socks5h` protocol.

## Usage

### 1) Reading Proxies

Read line by line from `SOCKS5_RAW.txt`, for example:

1.2.3.4:1080

### 2) Command-line testing (curl)

```bash
curl --socks5 1.2.3.4:1080 https://api.ipify.org
```

To route domain name resolution through the proxy as well:

```bash
curl --socks5-hostname 1.2.3.4:1080 https://api.ipify.org
```

### 3) Using Linux environment variables

```bash
export ALL_PROXY="socks5://1.2.3.4:1080"
export http_proxy="socks5://1.2.3.4:1080"
export https_proxy="socks5://1.2.3.4:1080"

curl https://api.ipify.org
```

### 4) Separate Git Proxy Configuration

```bash
git config --global http.proxy "socks5://1.2.3.4:1080"
git config --global https.proxy "socks5://1.2.3.4:1080"
```

Unconfigure:

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### 5) Python (requests)

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

> Before using the SOCKS proxy capabilities of `requests`, install `requests[socks]` or `pysocks`.

### 6) Common Clients

- **Clash / Clash.Meta**: Add a new `type: socks5` node and fill in the `server` and `port`. 
- **v2rayN / Qv2ray**: Add a Socks node or local forwarding, pointing to `ip:port`. 
- **Browser extensions like SwitchyOmega**: Create a new SOCKS5 profile mode and enter the IP/port.

It is recommended to incorporate retry on failure, polling, and health check mechanisms in actual use.

## Updates and Availability

- Full pull and cleaning: Occurs approximately once per hour by default.  
- Interval keep-alive checks: Periodic viability verification is performed between full pulls.  
- Network guardian: An online detection switch; validation for the current round can be paused if network detection fails, and it will resume automatically upon recovery.  
- Historical cleanup: Timed deletion based on retention days (30/15/7/1), aligning with the retention policies for historical statistics and operation logs.  
- Availability note: `SOCKS5.txt` only represents current or recent detection results and does not guarantee long-term stability.

Use cases: temporary data fetching, testing environments, learning and research.  
It is not recommended for direct use in production environments with high compliance and stability requirements.

## Risk Statement and Usage Guidelines

- Proxies are sourced from the public internet, and their authenticity, compliance, and security cannot be guaranteed. 
- Using proxies may pose risks such as **privacy leakage, account risk control, and data breaches**; please assess these risks on your own. 
- Please strictly comply with local laws and regulations as well as the terms of service of the target platform, and do not use them for illegal or unauthorized purposes. 
- It is recommended to use them solely for protocol learning, network debugging, and research testing.
