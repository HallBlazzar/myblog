+++
title = "DNS Resolution Issue Caused By NetBird "
date = 2026-05-23
draft = false
categories = ["Debian", "Linux", "Netbird"]
+++

If you use NetBird as VPN solution and occasionally encounter DNS resolution issue, this topic could help.

# Background

I've been using [NetBird](https://netbird.io/) to access my homelab for a while. There are few reasons I chose it:

- Sovereignty - I can deploy and gain control over all components.
- Run Out-Of-Box - NetBird provides good web-based GUI to manage clients and certs. I can control everything through the well-designed web interface without having to interact with config files and CLIs
- Custom DNS - To provide HTTPs/TLS to my homelab services, I want to bind public domains map to service IPs. However, DNS providers like CloudFlare restrict [DNS rebinding](https://en.wikipedia.org/wiki/DNS_rebinding) to public domains and private IPs. Therefore, an alternative approach is binding private IPs to private DNS records, and mapping them to public DNS records via CNAME. This approach requires VPN solutions to support custom private DNS. Though some people might compelely rely on private DNS and private IP, that means they need to manage certificates by themselves as generally modern SSL issuers like [Let's Encrypt](https://letsencrypt.org/) require DNS validation, where public domains are necessary (especially for free solutions).

Other VPN solutions also support some features above like [ZeroTier](https://www.zerotier.com/) and [TailScale](https://tailscale.com/), but their control plane are basically non-open-sourced and cannot be self-hosted.

### Deployment

I deploy NetBird and services as below:

<div style="text-align: center;">
    <img src="/posts/2026_may_23_netbird_dns/images/overview.drawio.svg">
</div>

- Server is deployed on a virtual machine run on cloud based on the [Self-Hosting Quickstart](https://docs.netbird.io/selfhosted/selfhosted-quickstart) guidance. This makes the server public accessible to my devices (laptops and smartphones).
- Services and DNS server run as containers in my local server. Communicating with NetBird service via [sidecar containers](https://docs.netbird.io/get-started/install/docker#docker-run-command).

# Problem

Occasionally, my laptop lost DNS resolution ability (Debian Trixie based). The situation was, browser and CLI tools like curl/dig couldn't access public services, e.g., Google, but pinging works.

# Investigation / Resolution

On my laptop, when the problem occurred, DNS server was pointing to the private DNS service's IP in the VPN. This was the default behavior on my laptop as I made NetBird client run on system boot. After checking, I perceive that VPN service was running into issue and couldn't handle DNS queries. Simply stopped NetBird client made DNS resolution on my laptop worked again. However, to fixed the issue, I still needed to restart it to made DNS server work again.

One interesting part was, the situation applied to Linux but Android and Windows. It looks clients on these platform leveraged stragegy to override DNS resolution. Not sure these behavior will be changed in the future, so it seems starting clients when required instead of making it always on is better approach.