+++
title = "lbhelper：基於 Python 的 Debian Live Build Wrapper"
date = 2026-04-12
draft = false
categories = ["Debian", "Linux", "Python", "Projects"]
+++

{{<notice tip>}}
TL;DR：如果你正嘗試建構自己的 Debian Images （針對桌面環境），我開發了一個基於 Python3 的函式庫 [lbhelper](https://hallblazzar.github.io/lbhelper/source/index.html)，可以協助你簡化這部分的開發工作。
- 官方文件 - https://hallblazzar.github.io/lbhelper/source/index.html
- GitHub - https://github.com/HallBlazzar/lbhelper
- 基於這個函式庫建構的 Debian Images ，也是我目前運行在我的筆電上運行的系統 - https://github.com/HallBlazzar/myos
- PyPi - https://pypi.org/project/lbhelper/
{{</notice>}}

## Image Customization 的解決方案

如果你正試圖為幫你的桌機或筆電建構自訂的 Debian Images ，你可能會發現不容易找到合適的解決方案。通常一般使用者傾向於在首次開機後運行 Shell Scripts 或 [Ansible playbooks](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_intro.html) 來直接設定系懂。這種方式簡單直觀。但由於這些 Script 很少而且需要手動執行，容易隨著 Package 或 Config 過期而失效。雖然直接使用 [Penguin's Egg](https://penguins-eggs.net/) 打包系統能夠直接獲得可以工作的 Image ，但由於遺漏了建構與變更 Package 或 Config 的過程，容易導致最終的 Image 往往難以重現（non-reproducible）。有些人可能會考慮 [FAI](https://fai-project.org/FAIme/)，但除了安裝套件之外，它提供的自訂功能有限。[Packer](https://developer.hashicorp.com/packer) 能讓結果便於重現，但由於依賴 VM 進行建構，導致執行的過程需要更長的時間以及更多系統資源。

除了上述通用型工具（可用於多數 Linux Distribution ）外，[Debian Live Build](https://live-team.pages.debian.net/live-manual/html/live-manual/index.en.html) 則是專為 Debian 開發並由其社群維護的工具。除了官方的 Debian Images 外，也被用於基於 Debian 的發行版，例如 [Kali Linux](https://www.kali.org/docs/development/live-build-a-custom-kali-iso/)。Live Build 支援基於檔案與 Script 的設定方式，可以建立如 Ansible 或 Packer 一樣建立結構化的設定檔，並且在 chroot 環境中運行（關於透過純 chroot 建構 Image，參見[這篇文章](https://dev.to/vaiolabs_io/how-to-create-custom-debian-based-iso-4g37)）。此外，Live Build 支援建立 Live Image，可以在不需要直接安裝到目標機器或虛擬機的情況下，就能夠運作（甚至可以自訂 Live Images 的開機選單），使驗證最終的系統和發行你的 Images 更容易。

## Live Build 運作原理

要透過 Live Build 建立自訂 Images ，我們需要準備一個目錄來存放配置。其中，每個子目錄代表不同的配置類型，如下所示：

```text
├── auto                    # live-build 自動化腳本。定義 live-build 的 config/build/cleanup 選項。
└── config
    ├── archives                # 套件來源 / 系統函式庫
    ├── hooks                   # 在建構階段運行的額外腳本
    │   ├── live                # 僅在 Live 系統運行的 Hook Scripts
    │   └── normal              # 在 Live 和 Installed System 中都會運行的 Hook Scripts
    ├── includes.binary         # 要包含在 ISO/CD-ROM 檔案系統中的檔案
    ├── includes.chroot         # 要包含在 Live 系統檔案系統中的檔案
    ├── package-lists           # 要安裝的套件列表
    │   ├── *.list.chroot       # 在已安裝和 Live System 中都要安裝的套件
    │   └── *.list.chroot_live  # 僅在 Live System 中安裝的套件
    ├── packages.chroot         # 要安裝在系統中的獨立 .deb 檔案
    ├── apt/preferences         # 建構時的 Aptitude Preference（影響建構過程）
    ├── etc/apt/preferences     # 安裝完成後的系統 Aptitude Preference 設定。
    └── bootloaders             # Live System Bootloader
```

#### Live System 與 Installed System

由 Live Build 建構的Images 是一個 **Live System**，可以直接從 Live CD 或 Live USB 執行。相比之下，從 Live System 安裝到硬碟中的系統稱為 **Installed System**。Live Build 允許使用者分別自訂要在 Live System 或 Installed System 中運行的套件與腳本，這配置和發行 Images 時提供了更大的靈活性。例如，使用者可能希望在 Live 系統中包含 [calamares](https://calamares.io/) 安裝器，但並不希望它出現在 Installed System 中。

#### 建構過程

Live Build 遵循以下階段順序來建構 Images ：
1. **[Bootstrap]** 建立一個 Debian chroot 目錄。所有變更都將在此發生，不會影響主機本身環境。
2. **[Chroot]** Chroot 至該目錄。
3. **[Packages]** 安裝 Package（/packages-lists）。
4. **[Hooks]** 執行 Hook Scripts（/hooks）。
5. **[Binary]** 加入 Static Files （/includes.*）。
6. **[Imaging]** 製作 Live Images ISO。

## lbhelper 帶來的改進

然而，Live Build 仍有一個小缺陷，即設定檔管理。對於 Packer 和 Ansible，設定檔可以根據不同的套件或用途實現完全結構化。例如，使用者可以將一個套件的安裝指令和設定檔放在同一個 Playbook 中，讓使用者能夠將想要應用到目標系統的變更模組化。設計良好的 Playbook 可以確保不同 Playbook 之間的影響降到最低。相反地，Live Build 根據檔案在實際系統中的路徑來存放檔案。雖然這種方式比較直覺（檔案會直接出現在系統相同路徑中），但一旦套件數量增加，管理與追蹤來自不同來源（例如第三方套件庫）的設定檔和套件就會變得非常困難。

例如，要設定一個預裝擴充功能與預先配置的 Firefox，使用者需要：
- 下載 FireFox Extensions 檔案
- 添加 Extension
- 修改特定路徑下的設定檔（有些使用 Hook Scripts，有些使用 Static Files）

每一部分都可能在 Live Build 設定檔目錄下產生 1-2 個額外的檔案或腳本。對於現代開發者來說，使用 2-3 個 IDE 以及超過 20 個 CLI 和桌面工具是很常見的，更不用說企業環境（例如統一的瀏覽器設定檔、LDAP/Kerberos、SELinux 等）。對於開發者或 Images 維護者來說，在類似檔案系統的目錄結構下追蹤設定檔並不理想。

因此，我開發了一個基於 Python 3 的套件 [lbhelper](https://hallblazzar.github.io/lbhelper/source/index.html) 來簡化設定檔管理工作。Live Build 中的每一種設定檔類型在這個套件中都被轉換為 Python Class ( `Target` )。

#### 目標 (Targets)

例如，要安裝 GNOME 桌面，使用者原本需要在 `config/package-lists/gnome.list.chroot` 下建立一個設定檔：

```text
task-gnome-desktop
task-english
```

在 lbhelper 中則可以被定義為一個 Python Object：

```python3
gnome_desktop_packages = UpstreamPackages(
    packages=[
        "task-gnome-desktop",
        "task-english",
    ],
    package_set_code="desktop",
)
```

透過這種變更，使用者可以將 GNOME 相關的套件、設定檔與檔案分類在同一個 Python Module 中。

一個更複雜的例子是 Firefox。要添加 Extension 並更改 UI Layout，使用者需要：
- 將 Firefox 加入安裝清單
- 添加下載 Extension 的 Script
- 在 `/usr/lib/firefox-esr/distribution/policies.json` 定義要安裝的 Extension
- 在 `/etc/firefox-esr/firefox-esr.js` 定義瀏覽器的 UI Layout

使用 lbhelper，上述變更可以被定義為：

```python3
from lbhelper import StaticFile, HookScript, download_file, render_template_to_file, render_template_to_string

from importlib.resources import files
from pathlib import Path

# 參考 https://support.mozilla.org/en-US/kb/customizing-firefox-using-policiesjson
firefox_esr_config_path = Path("/usr/lib/firefox-esr/distribution/policies.json")
extension_dir = Path("/opt/firefox-esr/extensions")
policy_path = Path("/usr/share/firefox-esr/distribution/policies.json")

workona_path = extension_dir / "workona.xpi"
undo_tab_path = extension_dir / "undo.xpi"

undo_tab_extension = StaticFile(
    undo_tab_path,
    get_source_file=lambda : download_file(undo_tab_url),
)

firefox_esr_policy_content = render_template_to_string(
    template_path=Path(str(files(__package__) / "policies.json.j2")),
    extensions=[workona_path, undo_tab_path]
)

firefox_esr_extension_installation_hook = HookScript(
    get_script_file=lambda : render_template_to_file(
        template_path=Path(str(files(__package__) / "install-extension.sh.j2")),
        extension_dir_path=extension_dir,
        policy_content=firefox_esr_policy_content,
        policy_dir_path=policy_path.parent,
        policy_path=policy_path,
    ),
    hook_name="install-firefox-esr-extensions",
)

firefox_esr_autoconfig_path = Path("/etc/firefox-esr/firefox-esr.js")

firefox_esr_autoconfig_file = StaticFile(
    firefox_esr_autoconfig_path,
    get_source_file=lambda : Path(str(files(__package__) / "firefox-esr.js"))
)

targets = [
    firefox_esr_extension_installation_hook,
    firefox_esr_autoconfig_file,
]
```

Firefox 設定檔的 Template 被定義為 `*.j2`，這也讓設定檔管理變得更加靈活。

#### 輔助函數 (Helper functions)

lbhelper 還提供了內建的輔助函數來簡化設定檔建立工作，例如：
- `download_file` - 從網路上下載任意檔案。這對於在建構過程中下載最新的第三方套件或檔案非常有用，能夠確保系統中的所有內容始終保持最新。
- `render_template_to_file`, `render_template_to_string` - 從指定的 `Jinja2` Template 和 Variable 建立字串或檔案。對於生成 Hook Script 或設定檔非常有用。

使用者只需要呼叫 `build_image`，就能將所有 `Targets` 轉換為實際的 Live Build 配置並開始建構過程：

```python3
from lbhelper import build_image
from targets import targets
from pathlib import Path

build_image(targets=targets, iso_build_dir=Path("build"))
```

#### 延伸閱讀

要深入了解 lbhelper，可以查看：
- 官方文件 - [https://hallblazzar.github.io/lbhelper/source/index.html](https://hallblazzar.github.io/lbhelper/source/index.html)
- GitHub - [https://github.com/HallBlazzar/lbhelper](https://github.com/HallBlazzar/lbhelper)
- **(強力推薦)** 基於這個 Library 的 Debian Live Images 。這也是我在筆電上安裝系統 - [https://github.com/HallBlazzar/myos](https://github.com/HallBlazzar/myos)
- PyPi 頁面 - [https://pypi.org/project/lbhelper/](https://pypi.org/project/lbhelper/)

## 未來展望

目前我正在繼續加入更多 Helper Functions 和 `Targets`，例如安裝 AppImage 和客製化 GRUB。如果你對這個專案有任何想法，請隨時在 GitHub 頁面上提出 Issue；如果你覺得它對你有幫助，也請不吝給予 Star。也歡迎任何能讓它變得更好的 Pull Requests。