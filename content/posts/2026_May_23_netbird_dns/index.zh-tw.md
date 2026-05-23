+++
title = "NetBird 引起的 DNS 解析問題"
date = 2026-05-23
draft = false
categories = ["Debian", "Linux", "Netbird"]
+++

如果您使用 NetBird 作為 VPN 解決方案，且偶爾會遇到 DNS 解析問題，本文或許能有所幫助。

# 背景

我使用 [NetBird](https://netbird.io/) 來作為我的 HomeLab 的連線方案已經有一段時間了。我選擇它的原因有以下幾點：

- 自主權 — 我可以自行部署並完全掌控所有元件。
- 開箱即用 — NetBird 提供了優秀的網頁版 GUI 來管理用戶端與憑證。我可以透過設計良好的網頁介面控制一切，不需要與設定檔或 CLI 頻繁互動。
- 自訂 DNS — 為了讓我的 HomeLab 服務支援 HTTPS/TLS，我想將公開網域綁定並映射到 HomeLab 上的服務的 IP。然而，像 CloudFlare 這類的 DNS 提供商會限制公開網域與私有 IP 以預防 [DNS rebinding](https://en.wikipedia.org/wiki/DNS_rebinding)。因此，另一種替代方法是將私有 IP 綁定到私有 DNS 紀錄，並透過 CNAME 將它們映射到公開 DNS 紀錄。這種方法需要 VPN 支援自訂私有 DNS。雖然有些人可能完全依賴私有 DNS 和私有 IP，但這意味著他們必須自己管理憑證，因為一般像 [Let's Encrypt](https://letsencrypt.org/) 這類現代 SSL 發行機構都需要 DNS 驗證，而 Public Domain 對此是不可或缺的（特別是對於免費的解決方案而言）。

其他 VPN 解決方案（例如 [ZeroTier](https://www.zerotier.com/) 和 [TailScale](https://tailscale.com/)）雖然也支援上述部分功能，但它們的 Control Plane 基本上不是開源的，且無法自建。

### 部署架構

我的 NetBird 及相關服務部署架構如下：

<div style="text-align: center;">
    <img src="/posts/2026_may_23_netbird_dns/images/overview.drawio.svg">
</div>

- 伺服器是根據 [Self-Hosting Quickstart](https://docs.netbird.io/selfhosted/selfhosted-quickstart) 指南，部署在雲端執行的一台虛擬機器上。這使得我的裝置（筆記型電腦和智慧型手機）可以從外部公開存取該伺服器。
- 服務與 DNS 伺服器則是以容器形式執行在我的本地伺服器中。透過 [sidecar containers](https://docs.netbird.io/get-started/install/docker#docker-run-command) 與 NetBird 服務連線。

# 問題

我的筆記型電腦（基於 Debian Trixie）偶爾會失去 DNS 解析能力。具體情況是，瀏覽器和 CURL/DIG 等 CLI 工具無法存取外部公開服務（例如 Google），但是 ping 卻可以正常運作。

# 調查 / 解決方案

當問題在我的筆記型電腦上發生時，DNS 伺服器正指向 VPN 內部私有 DNS 服務的 IP。這是我筆記型電腦上的預設行為，因為我將 NetBird Client 設定為系統啟動時自動執行。經檢查後，我發現是 Local Server 上的 VPN 服務遇到了問題，導致無法處理 DNS 查詢。只要單純停止 NetBird 用户端，我筆記型電腦上的 DNS 解析功能就會恢復正常。不過，若要徹底解決該問題，我仍需要重新啟動 DNS 伺服器。

一個有趣的點是，這種情況只發生在 Linux 上，Android 和 Windows 則沒有問題。這看起來像是這些平台上的 Client 端採用了不同的策略來覆寫 DNS 設定。目前不確定這些行為未來是否會改變，因此現階段看來，在需要時才啟動 Client 端，而不是讓它一直保持開啟，似乎是比較好的做法。