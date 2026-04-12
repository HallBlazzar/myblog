+++
title = "lbhepler: A Python based wrapper for Debian Live Build"
date = 2026-04-12
draft = false
categories = ["Debian", "Linux", "Python", "Projects"]
+++

{{<notice tip>}}
TL;DR: If you're trying to build your own Debian image(especially for desktop environment), I created a Python3 based library, [lbhelper](https://hallblazzar.github.io/lbhelper/source/index.html), could help you simplify the work.
- Online document - https://hallblazzar.github.io/lbhelper/source/index.html
- GitHub - https://github.com/HallBlazzar/lbhelper
- A Debian Image based on this library, which is also the OS I run on my laptop - https://github.com/HallBlazzar/myos
- PyPi page - https://pypi.org/project/lbhelper/
{{</notice>}}

## Image Customization Options

If you're trying to build your own Debian image for your desktop/laptop, you might have hard time on finding a suitable solution. In general, people tend to run shell scripts or [Ansible playbooks](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_intro.html) after first boot, which is simple and straightforward. But these scripts are rarely performed and handy, which are broken easily as packages/configs out-of-date. Though the [Penguin's Egg](https://penguins-eggs.net/) is a good approaches to package your system, result image is easily become non-reproducible as changes to packages/configs are missing. Some people might consider [FAI](https://fai-project.org/FAIme/), but it doesn't provide customizations more than install packages. [Packer](https://developer.hashicorp.com/packer) seems makes result more reliable and reproducible, but is too heavy as it requires VMs to build images.

Other than general purpose tools above, [Debian Live Build](https://live-team.pages.debian.net/live-manual/html/live-manual/index.en.html) is developed for Debian and maintained by its community. In addition to official Debian images, it's also used by Debian based distributions like [Kali Linux](https://www.kali.org/docs/development/live-build-a-custom-kali-iso/). The Live Build supports files/scripts based configuration, which you can create structural configs as Ansible and Packer, and also runs in chroot-based environment (For pure chroot approach, see the [post](https://dev.to/vaiolabs_io/how-to-create-custom-debian-based-iso-4g37)). Also, the Live Build supports creating live images. It allows users to run on target machines/VMs without installation (you can even customize live images' boot menu), which is easier to validate results and distribute images.

## How It Works

To create a customized image via Live Build, we need to prepare a directory to store configurations. Each subdirectory represents different configuration as below shows:

```text
├── auto                    # live-build auto-scripts. Defines config/build/cleanup options for live-build.
├── config
├── archives                # Package mirrors/repositories
├── hooks                   # Extra scripts to run during build stages
│   ├── live                # Hooks to run on live system
│   └── normal              # Hooks to run on both live and installed system
├── includes.binary         # Files to include on the ISO/CD Rom filesystem
├── includes.chroot         # Files to include in the live system's filesystem
├── package-lists           # Packages to install
│   ├── *.list.chroot       # Packages to install on both installed and live system
│   └── *.list.chroot_live  # Packages to install only on live system
├── packages.chroot         # Standalone .deb packages to install on both installed and live system
├── apt/preferences         # Build time aptitude preference. Takes effect while building imaage
├── etc/apt/preferences     # Aptitude preference from installed system.
└── bootloaders             # Bootloaders for live system
```

#### Live and Installed System

An image built from Live Build is a live system which can run directly on Live CD or Live USB. By contrast, a system installed from a live system is called **installed system**. Live Build allows user to customize packages and scripts run on live and installed systems, which gives them more flexibilities while defining and distributing images. For instance, users might want to include [calamares](https://calamares.io/) installer in their live system, which is not expected on an installed system.

#### Build Process

Live Build follows the sequence of stages to build an image:
1. **[Bootstrap]** Create a Debian chroot directory. All changes will happen in it and won’t cause impacts to host.
2. **[Chroot]** Chroot to the directory.
3. **[Packages]** Install packages (/packages-lists).
4. **[Hooks]** Run hooks scripts (/hooks).
5. **[Binary]** Include static files ( /includes.* ).
6. **[Imaging]** Create live image ISO.

## Improvements From lbhelper

However, the Live Build still has a small flaw, i.e., configuration management. For Packer and Ansible, configuration can be fully structured based on different packages or purposes. For example, users can put installation commands and configurations of a package in a playbook. It allows users to modulize changes they'd like to apply to the target systems. Well-designed playbooks can ensure the least side effects across different playbooks. Instead, the Live Build stores files under paths as in actual systems. The approach is straightforward as files will appear in installed/live systems, but once number of packages grow, tracking configs and packages from different sources (e.g, 3rd party repositories) become problematic.

For instance, a pre-configured Firefox browser(with configs and extensions) requires users to:
- Install firefox package
- Adding extensions
- Alter config file on specific paths (some use hooks scripts, some use static file) 

Each part could generate 1-2 extra files/scripts under the Live Build config directories. For a modern developer, it's pretty common using 2-3 IDEs and more than 20 CLI and desktop tools, no to mention for enterprise scenario (e.g., unified browser configs, LDAP/Kerberos, SELinux, etc). It's not ideal for a developer or image maintainer to track configs under a filesystem-liked directory structure.

As a result, I wrote a Python 3 based library, [lbhelper](https://hallblazzar.github.io/lbhelper/source/index.html), to simplify the config management work. Each type of config under the Live Build is turned into Python classes in the library. 

#### Targets

For instance, to install gnome desktop, users need to create a config file under the `config/package-lists/gnome.list.chroot`:

```text
task-gnome-desktop
task-english
```

In lbhelper, it can be defined as Python object:

```python3
gnome_desktop_packages = UpstreamPackages(
    packages=[
        "task-gnome-desktop",
        "task-english",
    ],
    package_set_code="desktop",
)
```

With this change, users can categorize GNOME related packages/configs/files in a Python module. 

A more complex example is FireFox. To add extensions and change browser layout, users need to:
- Add FireFox as package to install
- Add extension download script
- Define extension to install in `/usr/lib/firefox-esr/distribution/policies.json`
- Define browser layout in `/etc/firefox-esr/firefox-esr.js`

With lbhelper, changes above can be written as:

```python3
from lbhelper import StaticFile, HookScript, download_file, render_template_to_file, render_template_to_string

from importlib.resources import files
from pathlib import Path

# Refer to https://support.mozilla.org/en-US/kb/customizing-firefox-using-policiesjson
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

Templates of FireFox configs are defined as `*.j2`, which also makes configuration managements more flexible. 

#### helper functions

The lbhelper also provides built-in helper functions to simplify configuration creation work like:
- `download_file` - Download arbitrary file from the internet. Useful to download latest 3rd-party packages or files during build process to ensure everything in system always up-to-date.
- `render_template_to_file`, `render_template_to_string` - Create strings or files from given `Jinja2`(https://jinja.palletsprojects.com/en/stable/intro/#installation) template and variables. Useful for generating hook scripts or files.

To turn all targets into actual Live Build configs and start building process, users can simply call the `build_image` and provide necessary parameters:

```python3
from lbhelper import build_image
from targets import targets
from pathlib import Path

build_image(targets=targets, iso_build_dir=Path("build"))
```

#### Further Readings

To gain more insights about the lbhelper, you can check:
- Online document - https://hallblazzar.github.io/lbhelper/source/index.html
- GitHub - https://github.com/HallBlazzar/lbhelper
- **(Strongly recommended)** A Debian Live Image based on this library. This is also the OS I run on my laptop - https://github.com/HallBlazzar/myos
- PyPi page - https://pypi.org/project/lbhelper/

## Future Of This Project

Currently, I'm still working on adding more helper functions and targets to making the library cover more general scenarios. For example, installing AppImages and customizing GRUB. If you have any thoughts about this project, please feel free to create issues on its GitHub page; if you find it useful to you, please also don't hasitate to give it a star. Any PRs to make it better are welcomed.
