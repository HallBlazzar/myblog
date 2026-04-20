+++
title = "我如何為筆記型電腦選擇作業系統？"
date = 2026-04-19
draft = false
categories = ["Debian", "Linux"]
+++

{{<notice tip>}}
**TL;DR：** 如果你是一名正在為筆記型電腦/工作站選擇 Linux 桌面環境而苦惱的開發者，Debian 基本上是最可靠的方案。否則，建議從 [DistroWatch](https://distrowatch.com/dwres.php?resource=popularity) 中挑選並測試，找出最適合你的一個。
{{</notice>}}

這是一個關於我如何為新筆記型電腦選擇作業系統的故事，也是一段為期 2 + 3 個月的旅程。

去年 11 月（2025 年），我趁黑五優惠花了約 1,200 美元買了一台 Dell Pro 14 筆記型電腦（64 GB RAM, AMD Ryzen 5 220），用來替換我的 Surface Laptop 3（32 GB RAM, i7-1065G7）。Surface Laptop 3 對開發者來說其實很不錯，但 Windows 11 和近期的桌面應用程式（加上 WSL2）對它來說似乎太沉重。即便重新安裝整個系統，它還是會隨著時間運行得越來越慢。因此，在去年年底，我決定更換它。

第一個問題是：我應該使用哪個作業系統？

事實上，我腦中浮現的第一個念頭是選擇 Windows 以外的選項。我使用 Windows 已經超過 20 年了。我仍然認為它擁有最好的使用者體驗。使用者不需要花太多時間去理解如何使用，所有的操作都很直觀。此外，我日常使用的大多數現代應用程式都是基於 Windows 的，例如 Notion Desktop、[Groupy](https://www.stardock.com/products/groupy/) 和 Notepad++。然而，我還是決定離開它。

最顯著的原因是系統資源消耗。在 Surface Laptop 3 上，Windows 佔用了約 8 GB 的記憶體來運行大量用途不明的 Background Processes。當我運行應用程式時，情況變得更糟。在日常工作中，我習慣同時運行超過 100 個標籤頁的 Firefox、多個基於 Jetbrains IDE Instances 以及 WSL。在這種情況下，工作管理員無法完全反映出的記憶體與 CPU 使用狀況。例如，當我看到系統 CPU 和記憶體使用率已達到近 90% 時，工作管理員的 Process 中所記錄到的資源消耗總數往往對應不起來。即使我嘗試關閉所有應用程式與 WSL，系統資源消耗仍然高於剛開機時的狀態。

就我記憶所及，自 Windows 10 以來，Windows 本身為了安全性與系統資源最佳化等不同目的，增加了大量的 Background Process。但自從 Windows 11 引入 AI 相關功能和預裝的桌面小程式後，辨識資源消耗的來源變得更加困難。網路上已經有大量關於如何最佳化 Windows 以降低資源消耗的討論，包括修改 Registry 和安裝第三方工具（而這些隨時可能被隨機的 Windows Update 覆蓋）。然而，我對作業系統最基本的要求是運行**我需要的東西**。作業系統本身的任何額外功能都是加分項，但不應成為這件事的阻礙，尤其是缺乏簡單且官方支援的方法來直接關閉或移除它們。

我熱愛 Windows，但該是改變的時候了。

除了 Windows，Linux 似乎是最可行的選擇。大多數軟硬體的支援仍然是針對 Windows、MacOS 和 Linux。雖然還有像 [ReactOS](https://reactos.org/) 這樣的 Open Source 作業系統，但我對於將他們是否能夠實際使用在生產環境仍抱持懷疑。但在我個人經驗中，基於 Linux 的桌面環境仍不像 Windows 或 MacOS 那樣穩定。例如，Ubuntu 在使用一段時間（幾天或幾週）後通常會開始跳出錯誤訊息，像是某些不明的升級失敗或損壞的系統組件；Fedora 更新頻繁，容易導致某些系統配置或軟體損壞（源於 System Library 或 Kernel）。然而，既然我已經決定選擇這個方案，我能做的就是盡力測試不同的 Linux 發行版和桌面環境，找出最適合我日常工作的一款。

我的需求非常簡單：
* **最少配置**：我希望作業系統本身能夠開箱即用，而不是需要大量特殊的設定。
* **不要有太多「驚喜」**：作業系統不應該存在一些被記錄在 GitHub Issues 或 Reddit/StackOverflow 上的限制，而這些限制會阻礙我運行的軟體。

基於這些原則，我的研究分為兩個領域：桌面環境與發行版。

# 桌面環境 (Desktop Environments)

Linux 有許多現代桌面環境，例如 [GNOME](https://www.gnome.org/)、[KDE](https://kde.org/)、[Cinnamon](https://projects.linuxmint.com/cinnamon/) 和 [XFCE](https://www.xfce.org/)。但由於我日常使用基於 Jetbrains 的 IDE，GNOME 和 KDE 對我來說成為了最可行的選擇。

## GNOME

GNOME 為提供相當流暢的使用者體驗。我不需要進行太多的自訂就能讓它運行得非常出色。此外，GNOME 擁有豐富的 Extensions 來增加額外功能，例如在 System Tray 顯示系統資訊（[Vitals](https://extensions.gnome.org/extension/1460/vitals/)）以及顯示 Caps Lock 和 Num Lock 狀態（[Lock Keys](https://extensions.gnome.org/extension/36/lock-keys/)）。

然而，我認為缺點也來自於它的設計。首先，GNOME 的可自訂部分（從 Settings 中）相對受限。使用者大多需要依賴 Extensions （基於 [JS](https://gjs.guide/extensions/development/creating.html)）來變更。但由於 Extension 大多是獨立的開源項目，很容易遇到過時（不支援最新 GNOME）或 Extension 之間衝突的情況。因此，我的建議是盡可能保持最少的 Extension ，並確保每個 Extension 都盡可能簡單（功能單一）。接受它提供的使用方式比要求它按自己的意願運作要容易得多。

## KDE

與 GNOME 相比，KDE 提供了更多自訂選項，如 System Tray 的數量和外觀。使用者幾乎可以對外觀進行任何更改。此外，KDE 的 Extension 也與 GNOME 一樣豐富。

然而，我沒有選擇 KDE 是因為 [SDDM](https://github.com/sddm/sddm)。這個 Display Manager 讓使用者體驗產生了割裂感。我的經驗是 KDE 現代且流暢，但 SDDM 卻很原始。SDDM 的主題和使用者體驗與 KDE 本身並不相容。此外，最奇怪的部分是 SDDM 的多顯示器支援。當登錄 KDE 環境時，登錄視窗會顯示在所有螢幕上，但只有其中一個有效。網路上有一些關於解決方案的討論，但我不想花時間在這上面。

目前，KDE 提供了一個 SDDM 的替代方案：[Plasma Login Manager](https://github.com/KDE/plasma-login-manager)，它似乎解決了這些問題。但由於我在它發布前就已經選擇了 GNOME，並且它尚未被包含在 Debian 中，我目前仍會繼續使用 GNOME，並可能在未來考慮 KDE。

## XFCE / LXDE

簡單且輕量。但由於 KDE 和 GNOME 在我的新筆記型電腦上不會對系統負載造成影響，我想我不需要考慮它們。

## Cinnamon

美觀且流暢，同時提供了類似 Windows 的使用者體驗和 UI。但 Extension 的支援不如 GNOME 和 KDE 豐富。此外，如果使用者想獲得最佳體驗，發行版的選擇大多僅限於 Ubuntu 系（例如 [Mint](https://linuxmint.com/) 和 [Ubuntu Cinnamon](https://ubuntucinnamon.org/)），而這並非我的首選。

## COSMIC

與 GNOME 類似，但仍處於早期階段且存在不少 Bug。

# 發行版 (Distributions)

我測試了 [DistroWatch](https://distrowatch.com/dwres.php?resource=popularity) 上排名靠前的各種發行版。最終，我的體會是：排名並不代表該作業系統適合所有場景。更多是顯示了哪些發行版獲得了更多的關注和討論。使用者仍需要親自安裝並佈署軟體進行測試，才能知道一個發行版是否能滿足其需求。

## Debian

老實說，我原本沒有把 Debian 考慮在內。與其他發行版相比，Debian 非常「無聊」。它不提供最新的套件包，也沒有像 Arch [AUR](https://aur.archlinux.org/) 那樣的內建第三方存儲庫或滾動更新，也沒有最新或自訂的 Kernel。然而，使用者可以毫無「意外」地運行軟體。特定版本中包含的套件包和函式庫非常穩定且始終有效。在嘗試了不同的發行版後，我發現它正是我真正需要的。系統開箱即用、不需要修改特殊設定來避免損壞、系統不會被隨機更新弄壞、最重要的是，它與我需要的所有軟體都相容。

## CachyOS

在這次研究之前我沒聽說過它。使用過後我可以想像它為什麼受歡迎。它包含預先配置良好的 UI，幾乎不需要額外的 Extension 。此外，它包含經過最佳化的 Linux Kernel ，似乎能針對現代硬體加速。最初，它是我第一個選則的測試對象，也是我認真考慮使用的系統。然而，Android Studio 中的模擬器在一週內的幾次更新後就無法再啟動，看起來像 AMD Driver 相關的問題，但無法簡單修復。

## Fedora / EndeavourOS / PopOS / ArchLinux

這些發行版上的 Android Studio 模擬器遇到了與 CachyOS 相同的問題（即使是在全新安裝的情況下）。

## MX Linux

沒有基於 GNOME 的選項。

## ZorinOS / Ubuntu / Ubuntu Cinnamon

在幾次更新後，會不斷收到隨機跳出的錯誤訊息。此外，基於 Ubuntu 的發行版使用不同的來源安裝軟體，包括 Ubuntu APT、PPA 和 Snap。這使管理套件來源變得混亂，因為每個來源似乎由不同的組織維護，且更新狀態並不一致。否則，它們的使用者體驗仍足以成為我的備案（如果我找不到其他合適的發行版）。

## Linux Mint

使用自己的套件包來源，但不支援 GNOME。

## Bazzite 與類似的不可變（Immutable）發行版

Android Studio 中的模擬器運行失敗。此外，在 Container / [DistroBox](https://distrobox.it/) 中運行軟體雖然看起來很簡潔，但是常需要額外設定，特別是某些桌面應用程式在 DistroBox 中運行效果不佳。

# 總結

整個研究花了我大約 2 個月的時間來驗證軟體與桌面環境之間的相容性和穩定性。雖然最後選擇了 Debian，但我仍不認為它是完美的。這種情況似乎適用於大多數 Linux 桌面發行版，因為它們的開發是由社群驅動的。有時，使系統穩定或易於使用並非社群的主要目標，更不用說那些有商業支援的系統（例如 Microsoft、Ubuntu、Apple 和 SUSE）。如今使用者能做到的似乎就是找到一個對日常使用而言**可以接受**但**不是完美**的選擇。期望一個一勞永逸的解決方案在現在看來並不實際。

順帶一提，如果你對另外 3 個月發生了什麼感興趣，可以看看 [lbhelper](/zh-tw/posts/2026_apr_12_lbhelper/) 。

# 附錄

如果你對 Dell Pro 14 與 Surface Laptop 3 的基本硬體規格感興趣：

| | Dell Pro 14                                                                                              | Surface Laptop 3 |
| :--- |:---------------------------------------------------------------------------------------------------------| :--- |
| **CPU** | AMD Ryzen 5 220, 6 Kernel  12 執行緒, 3.2-4.9 GHz                                                           | i7-1065G7, 4 Kernel  8 執行緒, 1.3-3.9 GHz |
| **記憶體** | 64 GB, Crucial DDR5, 5600 MHz, 32 GB x 2                                                                 | 32 GB |
| **儲存空間** | 1 TB, WD Black, SN770M                                                                                   | 1 TB |
| **保固** | 3 年 [Pro Support](https://www.dell.com/support/contents/en-ie/article/warranty/prosupport-suite-for-pcs) | 1 年非人為損壞 |
| **價格** | 約 $1200 (感謝 Black Friday 與 Amazon)                                                                       | 約 $3000 |

現在我的另一個想法是避免購買超高規格的筆電。如今 CPU 升級速度非常快，大約花了 5 年時間，非頂級的 CPU 就能以近 1/3 的價格（包含記憶體與 CPU）擊敗當年的頂級型號。以前，我期望一台設備足夠強大以維持 5 年以上；然而，我現在對硬體的期望是便宜且速度快到足以應付日常工作，這樣我可以隨時更換而不會造成沉重的財務負擔。

順便說一句，如果你正在考慮購買新筆記型電腦，Dell 是一個不錯的選擇，因為與其他品牌相比，它幾乎在所有國家都提供售後/到府維修服務。例如，Dell 是唯一在愛爾蘭提供到府維修服務的品牌。其他廠牌多要求使用者自行把電腦寄到他們的歐洲總部進行維修。　