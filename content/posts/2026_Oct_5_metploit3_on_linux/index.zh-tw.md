+++
title = "在 Linux + AMD 環境下執行 Metasploitable3 可能會遇到的問題"
date = 2026-10-05
draft = false
categories = ["Linux", "Security"]
+++

**TL;DR：** 如果你正嘗試在 Linux/KVM + AMD 平台上運行 Metasploitable3，我最好的建議是：別這麼做，直接換到 Wintel 環境。幫你自己省下大把時間與力氣。

# 背景

網路上許多資安入門文章都推薦使用 [Metasploitable3](https://github.com/rapid7/metasploitable3) 作為靶機，但這個專案目前已不再積極維護。在 Wintel 環境下，你依然可以透過 [Vagrant](https://portal.cloud.hashicorp.com/vagrant/discover?query=rapid7%2Fmetasploitable) 輕鬆啟動它（連 Repo 中指向 Vagrant Registry 的連結都已經失效了 ...）。相反地，在 KVM/QEMU 環境中，由於沒有官方支援，使用者必須自行重新建置映像檔——但實際上這幾乎是不可能的任務。

該專案提供了兩種系統版本：Windows Server 2008 與 Ubuntu 14.04。

## Windows Server 2008

建置 Windows Server 2008 的過程相當順利，但在嘗試開機時，你很可能會遇到以下錯誤：

```text
hv-time unsupported HyperV Enlightenment feature: time

```

根據這篇[部落格文章](https://blog.wikichoon.com/2014/07/enabling-hyper-v-enlightenments-with-kvm.html)與[技術解析](https://termbasehub.com/hyper-v-enlightenments/)，`Hyper-V Enlightenments` 是 Windows Server 2008+ 虛擬機器在 Linux 主機上啟用 Hyper-V Paravirtualization所必需的特性。然而，[AMD 處理器並不支援](https://platform9.com/kb/pcd/compute/unable-to-deploy-windows-vm-using-iso-with-amd-based-cpu)。若手動將其停用，又會直接導致開機失敗。如果沒有 Intel CPU，要成功運行這個映像檔會變得極其困難。

## Ubuntu 14.04

直接為 KVM/QEMU 建置 Ubuntu 14.04 幾乎是不可能的。

首先，建置腳本中的指令本身就有問題，必須依據此 PR 手動進行修補：[https://github.com/rapid7/metasploitable3/pull/627/changes/9d8a669fbdef64158b6684ec3755b31e1462e611](https://github.com/rapid7/metasploitable3/pull/627/changes/9d8a669fbdef64158b6684ec3755b31e1462e611)。

其次，建置 Ubuntu 14.04 需要透過 [packer-plugin-chef](https://github.com/hashicorp/packer-plugin-chef) 來安裝套件，但此外掛已停止維護，且無法再從 Packer 的官方外掛來源取得。雖然[手動安裝](https://github.com/hashicorp/packer-plugin-chef#manual-installation)依然可行，但 [Chef 本身已不再開源](https://www.reddit.com/r/devops/comments/b8l22o/chef_is_going_to_stop_open_source_releases/)。該外掛在執行 cookbook 時需要下載安裝 Chef 執行檔，但除非使用者自行修改原始碼並注入授權，否則安裝程序一定會失敗。

好消息是，官方提供的 [Ubuntu 14.04 VirtualBox Box](https://portal.cloud.hashicorp.com/vagrant/discover/rapid7/metasploitable3-ub1404) 可以透過 [vagrant-mutate](https://github.com/sciurus/vagrant-mutate) 轉換為 libvirt 格式：

```shell
vagrant box add rapid7/metasploitable3-ub1404
vagrant mutate rapid7/metasploitable3-ub1404 libvirt
```

# 其他修復項目

為了順利開機並讓攻擊機能夠連線到靶機，你還需要重新設定靶機的網路。在我的配置中，攻擊機運行於 KVM 的 Default Network 上，因此可以使用以下 `Vagrantfile` 設定：

```ruby
Vagrant.configure("2") do |config|
  config.vm.synced_folder '.', '/vagrant', disabled: true
  config.vm.define "ub1404" do |ub1404|
    ub1404.vm.box = "rapid7/metasploitable3-ub1404"
    ub1404.vm.hostname = "metasploitable3-ub1404"
    config.ssh.username = 'vagrant'
    config.ssh.password = 'vagrant'

    ub1404.vm.provider "libvirt" do |v|
      v.memory = 2048

      v.management_network_autostart = false
      v.management_network_name = "default"
      v.management_network_keep = true
    end
  end
end

```

如果 `management_network_*` 相關設定報錯，可能是缺少了 `vagrant-libvirt` 外掛，需要先執行安裝：

```shell
vagrant plugin install vagrant-libvirt

```

此外，由於 `management_network_autostart = false` 會停用 KVM/QEMU Default Network 的自動啟動，未來若需要手動啟動它，要執行：

```shell
sudo virsh net-autostart default
sudo virsh net-start default

```