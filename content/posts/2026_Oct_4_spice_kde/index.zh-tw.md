+++
title = "基於 KDE 的虛擬機在 Spice 中可能會遇到的問題"
date = 2026-10-04
draft = false
categories = ["Linux"]
+++

如果您在虛擬機器（VM）中執行桌面環境時，無法共用剪貼簿，這篇文章或許能幫上忙。

# 背景說明

這是我在基於 virt-manager/QEMU 的虛擬機器中執行 [Kali Linux](https://www.kali.org/) 時發現的問題。在 virt-manager 中設定桌面環境時，若需要在宿主機與虛擬機之間共用剪貼簿，需依據 [SPICE 官方文件](https://www.spice-space.org/spice-user-manual.html#agent)執行以下步驟：

1. 新增一個 VirtIO serial device

   <div style="text-align: center;">
       <img src="/posts/2026_oct_4_spice_kde/images/virtio_serial.png">
   </div>

2. 新增一個 Spicevmc channel

   <div style="text-align: center;">
       <img src="/posts/2026_oct_4_spice_kde/images/spice_channel.png">
   </div>

雖然有些教學會建議在宿主機端安裝 `spice-vdagent`，但對較新的發行版（例如 Ubuntu 26.04+ 或 Debian 13+）來說，基本上沒有這個必要。

然而，若您的客體 VM 使用的是 KDE，可能會觀察到一些異常狀況。在我遇到的情況中，從宿主機（Debian 13）複製文字到虛擬機（Kali 2026.2，基於 Debian 13）能正常運作，但反過來從虛擬機複製複製到宿主機上時，卻無法運作。這似乎是由[不同桌面環境中的 Wayland Compositor）實作方式差異](https://bugzilla.redhat.com/show_bug.cgi?id=2016563#c32)所導致。要徹底解決這個問題需要 對 KDE 進行 Patch，目前尚未修復。

# 替代解決方案（Workarounds）

我的建議如下：

* **切換至其他桌面環境：** 在我的測試中，至少 GNOME 與 XFCE 都沒有出現這個問題。
* **改用其他虛擬化平台：** 例如 [VirtualBox](https://www.virtualbox.org/)。不過在 Linux 上執行 VirtualBox 會依賴 KVM，會與 QEMU 產生衝突，可能會給使用者帶來額外的麻煩。
