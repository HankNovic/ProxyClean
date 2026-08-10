# ProxyClean

<!-- hy-mt2-i18n:start -->
[中文](./README.md) | [English](./README_en.md) | **日本語** | [Español](./README_es.md)
<!-- hy-mt2-i18n:end -->


公開されており、継続的に更新される SOCKS5 プロキシデータのウェアハウスです。

## データ概要

- プロキシプールファイル：`SOCKS5.txt`
- オリジナルデータファイル：`SOCKS5_RAW.txt`
- オリジナルの直接リンク（利用可能なプール）：https://raw.githubusercontent.com/HankNovic/ProxyClean/refs/heads/main/SOCKS5.txt
- オリジナルの直接リンク（オリジナルプール）：https://raw.githubusercontent.com/HankNovic/ProxyClean/refs/heads/main/SOCKS5_RAW.txt
- 接続性検証：SOCKS5ハンドシェイク + SOCKS5 CONNECTターゲットテスト
- デフォルトのCONNECTテストターゲット：`1.1.1.1:53`、`8.8.8.8:53`
- テストノード：**中国 福建省 厦門市 中国電信**

説明：
- リストにあるプロキシは、公開されている無料のソースから取得されています。
- このリポジトリは単にデータの集約、クリーニング、アクティブ状態の維持、公開を行うもので、有料サービスは一切提供していません。

## プロキシプールの特徴

- **国内で利用可能なものを優先**：中国本土のネットワーク環境下で接続性を検証し、合格したもののみが利用可能なプールに追加されます。  
- **複数ソースからの統合クリーニング**：複数の公開済みSOCKS5ソースを集約し、一元化して解析・重複削除・フィルタリングを行います。  
- **並行検証によるランキング付け**：接続性と遅延の検証を同時に並行して実行し、最近利用可能で遅延の安定性が高いノードを優先して出力します。  
- **実際の利用可能性の確認**：ハンドシェイクに合格した後、さらにSOCKS5 CONNECTによるターゲット検証を行い、ハンドシェイクとCONNECTの両方に合格した場合のみ成功と判断します。  
- **統一されたエラー処理戦略**：ハンドシェイクとCONNECTで共通のソフト/ハードエラー分類および重複削除の処理を行い、一時的なネットワークの不安定さによる誤判定を減らします。  
- **継続的なアクティブ維持と更新**：定期的な全量取得の間にアクティブ状態のチェックを行い、機能しなくなったノードを自動的に除外します。  
- **継続的なメンテナンス**：`SOCKS5.txt`は継続的に更新され、できるだけ最新の利用可能状況を反映させます。

## プロキシの出所（謝辞）

現在、主に以下の公開されている SOCKS5 ソースを集約しています（順序はありません）：

- [proxifly/free-proxy-list](https://github.com/proxifly/free-proxy-list)
- [TheSpeedX/PROXY-List](https://github.com/TheSpeedX/PROXY-List)
- [roosterkid/openproxylist](https://github.com/roosterkid/openproxylist)
- [hookzof/socks5_list](https://github.com/hookzof/socks5_list)
- [gfpcom/free-proxy-list](https://github.com/gfpcom/free-proxy-list)
- [dpangestuw/Free-Proxy](https://github.com/dpangestuw/Free-Proxy)

> 説明：このリポジトリは各単独のソースの品質を保証するものではなく、実際の検出結果に基づいてのみ技術的な選別を行っています。

## ファイル形式

`SOCKS5.txt` と `SOCKS5_RAW.txt` はどちらもテキスト形式を採用しています：

IP:ポート

- 1行に1つのSOCKS5プロキシが記載されています。
- IPとポートは英字のコロン`:`で区切ります。
- デフォルトではユーザー名/パスワードは含まれていません。
- クライアントは通常、`socks5`または`socks5h`プロトコルを使用します。

## 使用方法

### 1) プロキシの読み込み

`SOCKS5_RAW.txt`から行単位で読み込みます。例えば：

1.2.3.4:1080

### 2) コマンドラインでのテスト（curl）

```bash
curl --socks5 1.2.3.4:1080 https://api.ipify.org
```

ドメイン名の解決もプロキシ経由で行う場合：

```bash
curl --socks5-hostname 1.2.3.4:1080 https://api.ipify.org
```

### 3) Linuxの環境変数を利用する方法

```bash
export ALL_PROXY="socks5://1.2.3.4:1080"
export http_proxy="socks5://1.2.3.4:1080"
export https_proxy="socks5://1.2.3.4:1080"

curl https://api.ipify.org
```

### 4) Gitの個別プロキシ設定

```bash
git config --global http.proxy "socks5://1.2.3.4:1080"
git config --global https.proxy "socks5://1.2.3.4:1080"
```

設定の解除：

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

`requests` の SOCKS プロキシ機能を利用するには、まず `requests[socks]` または `pysocks` をインストールしてください。

### 6) よく使われるクライアント

- **Clash / Clash.Meta**：`type: socks5` のノードを新規追加し、`server` と `port` を入力します。  
- **v2rayN / Qv2ray**：Socks ノードまたはローカル転送を新規追加し、`ip:port` を指定します。  
- **SwitchyOmega などのブラウザプラグイン**：新しい SOCKS5 シナリオモードを作成し、IP/ポートを入力します。

実際の利用にあたっては、失敗時の再試行やポーリング、ヘルスチェックの仕組みを追加することを推奨します。

## アップデートと利用可能性

- 全量取得とクリーニング：デフォルトでは約1時間ごとに1回実行されます。
- 間隔をあけたライフチェック：全量取得が2回行われる間に定期的に接続状態の確認が行われます。
- ネットワーク監視機能：オンライン検出スイッチがあり、ネットワーク検出に異常が発生した場合はその回の検証を一時停止し、復旧後に自動的に再開します。
- 履歴の削除：保持日数（30/15/7/1日）に基づいてタイムアウト処理が行われ、履歴統計や実行ログの保持方針が適用されます。
- 利用可能性の説明：`SOCKS5.txt`は現在または最近の検出結果を示すものであり、長期的な安定性は保証されません。

適用シナリオ：一時的なデータ取得、テスト環境、学習や研究。  
高いコンプライアンス要件や高い安定性が求められる本番環境での直接使用は推奨されません。

## リスク告知と利用上の注意事項

- プロキシは公開ネットワークから取得されており、その出所の真実性、コンプライアンス、安全性は保証されません。
- プロキシを使用すると、**プライバシー漏洩、アカウントのリスク管理、データ漏洩**などのリスクが生じる可能性がありますので、ご自身でリスクを評価してください。
- 所在する地域の法律規制および対象プラットフォームの利用規約を厳守し、違法または不正な目的での使用は禁止されています。
- 協定の学習、ネットワークのデバッグ、研究テストにのみ使用することを推奨します。
