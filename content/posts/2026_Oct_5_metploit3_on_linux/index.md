+++
title = "Problem with Metasploitable3 on Linux+AMD"
date = 2026-10-05
draft = false
categories = ["Linux", "Security"]
+++

TL;DR, If you're trying to run Metasploitable3 on Linux/KVM and AMD CPU, my best advise is: DON'T. Switch to Wintel environments—it will save a lot of headaches.

# Background and Workarounds

Many security 101 articles recommend [Metasploitable3](https://github.com/rapid7/metasploitable3) as target machine, but this project is not actively maintained. In a Wintel-based environment, you can still simply boot it through [Vagrant](https://portal.cloud.hashicorp.com/vagrant/discover?query=rapid7%2Fmetasploitable) (even though the link to the Vagrant Registry in its repository is broken). By contrary, in a KVM/QEMU based environment, users must rebuild images by themselves as KVM/QEMU is not officially supported—and building them is practically impossible.

There are two distributions of this project: Windows Server 2008 and Ubuntu 14.04. 

## Windows Server 2008

Building Windows Server 2008 goes smoothly, but when attempting to boot it, you might encounter the error:

```text
hv-time unsupported HyperV Enlightenment feature: time
```

According to this [blog](https://blog.wikichoon.com/2014/07/enabling-hyper-v-enlightenments-with-kvm.html) [article](https://termbasehub.com/hyper-v-enlightenments/), `Hyper-V Enlightments` are required by Windows Server 2008+ VMs to enable Hyper-V paravirtualization features on Linux hosts. However, [this is not supported on AMD CPUs](https://platform9.com/kb/pcd/compute/unable-to-deploy-windows-vm-using-iso-with-amd-based-cpu). Manually disabling them can break the boot process entirely. Without Intel CPUs, running this image becomes exceedingly difficult. 

## Ubuntu 14.04

Building Ubuntu 14.04 directly for KVM/QEMU is practically impossible.

First of all, build command inside the build script is broken, and must be patched manually based on this PR 
- https://github.com/rapid7/metasploitable3/pull/627/changes/9d8a669fbdef64158b6684ec3755b31e1462e611. 

Second, the build process requires [Packer Plugin Chef](https://github.com/hashicorp/packer-plugin-chef) to install dependencies, but this plugin is unmaintained and no longer available from Packer's official plugin registry. Although [manual installation](https://github.com/hashicorp/packer-plugin-chef#manual-installation) still works, [Chef itself is not open-source anymore](https://www.google.com/url?sa=t&source=web&rct=j&opi=89978449&url=https://www.reddit.com/r/devops/comments/b8l22o/chef_is_going_to_stop_open_source_releases/&ved=2ahUKEwi7x6WRv6GXAxXeQUEAHXhMCZEQFnoECCAQAQ&usg=AOvVaw3wG5a-2wOdDuGQ1eunb8Cj). The plugin needs to install Chef binary first to run cookbooks, but installation will always fail unless users modify the source code and supply a valid license.

The Good news is, the [Ubuntu 14.04 Box](https://portal.cloud.hashicorp.com/vagrant/discover/rapid7/metasploitable3-ub1404) supports VirtualBox and can be converted to libvirt format using [Vagrant Mutate](https://github.com/sciurus/vagrant-mutate). Commands are:

```shell
vagrant box add rapid7/metasploitable3-ub1404
vagrant mutate apid7/metasploitable3-ub1404 libvirt
```

# Other Fixes

To boot target machines and make them reachable from attacker machines, users also need to reconfigure target machines' network settings. In my setup, my attacker machines run on KVM's default network, so the following Vagrantfile works:

```text
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

If the `management_network_*` options throw errors, make sure the `vagrant-libvirt` plugin is installed:
```shell
vagrant plugin install vagrant-libvirt
```

Because the `management_network_autostart=false` prevents the default network from starting automatically in KVM/QEMU, you may need to start and re-enable it manually after reboot host machine in future:
```shell
sudo virsh net-autostart default
sudo virsh net-start default
```
