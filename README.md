# Solus 4.7 Endurance Hardening Guide

This guide provides a comprehensive approach to hardening Solus 4.7 Endurance, focusing on enhancing security and privacy. The recommendations are based on the principles outlined in the https://madaidans-insecurities.github.io/, adapted for Solus.

---

## Table of Contents
1. [Introduction](#introduction)
2. [Kernel Hardening](#kernel-hardening)
3. [Sysctl Settings](#sysctl-settings)
4. [Boot Parameters](#boot-parameters)
5. [Mandatory Access Control (MAC)](#mandatory-access-control-mac)
6. [Sandboxing](#sandboxing)
7. [File Permissions](#file-permissions)
8. [Core Dumps and Swap](#core-dumps-and-swap)
9. [Physical Security](#physical-security)
10. [Best Practices](#best-practices)
11. [Further Reading](#further-reading)

---

## Introduction
Solus 4.7 Endurance is a rolling-release distribution that uses the ```eopkg``` package manager. While it is secure by default, additional steps can be taken to enhance its security.

### Key Recommendations:
- Use a non-root user for daily activities.
- Regularly update your system.
- Disable unnecessary services.
- Use encryption for sensitive data.

---

## Kernel Hardening
The kernel is a critical component of system security. Use a hardened kernel if possible.

### Steps:
1. **Install a Hardened Kernel:**
   - Check if a hardened kernel package is available in the Solus repositories.
   - If available, install it using:
     ```bash
     sudo eopkg install linux-hardened
     ```
   - If unavailable, consider compiling a custom kernel with hardening features.

2. **Blacklist Unnecessary Kernel Modules:**
   - Identify and blacklist unnecessary modules by creating a file in ```/etc/modprobe.d/```:
     ```bash
     sudo nano /etc/modprobe.d/blacklist.conf
     ```
   - Add lines like:
     ```
     install dccp /bin/false
     install sctp /bin/false
     ```
   - Reload the modules:
     ```bash
     sudo modprobe -r dccp sctp
     ```

3. **Disable Unneeded Kernel Features:**
   - Use ```sysctl``` to disable unneeded features (see [Sysctl Settings](#sysctl-settings)).

---

## Sysctl Settings
```sysctl``` allows you to configure kernel parameters at runtime or persistently.

### Recommended Settings:
1. **Kernel Self-Protection:**
   ```bash
   sudo sysctl -w kernel.kptr_restrict=2
   sudo sysctl -w kernel.dmesg_restrict=1
   sudo sysctl -w kernel.printk=3 3 3 3
   ```

2. **Network Hardening:**
   ```bash
   sudo sysctl -w net.ipv4.tcp_syncookies=1
   sudo sysctl -w net.ipv4.conf.all.rp_filter=1
   sudo sysctl -w net.ipv4.icmp_echo_ignore_all=1
   ```

3. **User Space Hardening:**
   ```bash
   sudo sysctl -w kernel.yama.ptrace_scope=2
   sudo sysctl -w vm.mmap_rnd_bits=32
   sudo sysctl -w vm.mmap_rnd_compat_bits=16
   ```

### Persistent Settings:
- Add the above settings to ```/etc/sysctl.conf``` or create a drop-in file in ```/etc/sysctl.d/```.

---

## Boot Parameters
Boot parameters can enhance security by enabling kernel-level hardening features.

### Steps:
1. **Edit Bootloader Configuration:**
   - For ```systemd-boot```, edit the loader entry:
     ```bash
     sudo nano /boot/loader/entries/solus.conf
     ```
   - Add parameters like:
     ```
     init_on_alloc=1 init_on_free=1 slab_nomerge
     ```
   - Regenerate the configuration if necessary.

2. **CPU Mitigations:**
   - Enable all applicable CPU mitigations:
     ```
     spectre_v2=on spec_store_bypass_disable=on tsx=off
     ```

---

## Mandatory Access Control (MAC)
Use AppArmor for fine-grained access control.

### Steps:
1. **Enable AppArmor:**
   - Ensure AppArmor is enabled in the kernel.
   - Install ```apparmor-utils```:
     ```bash
     sudo eopkg install apparmor-utils
     ```

2. **Create Profiles:**
   - Use ```aa-genprof``` to generate profiles for applications:
     ```bash
     sudo aa-genprof /path/to/program
     ```

3. **Enforce Policies:**
   - Set policies to "enforce" mode after testing:
     ```bash
     sudo aa-enforce /etc/apparmor.d/profile
     ```

---

## Sandboxing
Isolate applications to reduce the attack surface.

### Tools:
1. **Bubblewrap:**
   - Install ```bubblewrap```:
     ```bash
     sudo eopkg install bubblewrap
     ```
   - Example usage:
     ```bash
     bwrap --dev-bind / --proc-self --die-after-child "$program"
     ```

2. **Firejail:**
   - Install ```firejail``` (if available) for user-friendly sandboxing:
     ```bash
     sudo eopkg install firejail
     ```
   - Example usage:
     ```bash
     firejail --seccomp "$program"
     ```

---

## File Permissions
Restrict file permissions to minimize exposure.

### Steps:
1. **Restrict Home Directories:**
   ```bash
   chmod 700 ~
   ```

2. **Restrict Setuid/Setgid Binaries:**
   - Find setuid/setgid binaries:
     ```bash
     sudo find / -type f \( -perm -4000 -o -perm -2000 \)
     ```
   - Remove unnecessary setuid/setgid bits:
     ```bash
     sudo chmod u-s /path/to/program
     ```

3. **Set umask:**
   - Edit ```/etc/profile```:
     ```bash
     sudo nano /etc/profile
     ```
   - Add:
     ```
     umask 0077
     ```

---

## Core Dumps and Swap
Prevent sensitive data from being written to disk.

### Steps:
1. **Disable Core Dumps:**
   - Set ```kernel.core_pattern=|/bin/false``` in ```/etc/sysctl.conf```.

2. **Configure Swap:**
   - Reduce swapping:
     ```bash
     sudo sysctl -w vm.swappiness=1
     ```

---

## Physical Security
Protect against physical attacks.

### Steps:
1. **Full Disk Encryption:**
   - Use ```dm-crypt``` or ```LUKS``` to encrypt your drive.

2. **BIOS/UEFI Hardening:**
   - Enable Secure Boot.
   - Set a strong BIOS/UEFI password.
   - Disable unnecessary boot devices.

3. **USB Security:**
   - Use ```usbguard``` to control USB devices:
     ```bash
     sudo eopkg install usbguard
     ```
   - Configure rules to whitelist trusted devices.

---

## Best Practices
1. **Regular Updates:**
   - Update daily:
     ```bash
     sudo eopkg update && sudo eopkg upgrade
     ```

2. **Monitoring:**
   - Use ```auditd``` or ```sysdig``` to monitor system activity.

3. **User Education:**
   - Avoid using ```sudo``` unnecessarily.
   - Use strong, unique passwords.

---

## Further Reading
- [Solus Documentation](https://getsol.us/documentation/)
- [AppArmor Documentation](https://gitlab.com/apparmor/apparmor/-/wikis/Documentation)
- [Kernel Self Protection Project](https://kernsec.org/wiki/index.php/Kernel_Self_Protection_Project)


## hardened_malloc

https://discuss.grapheneos.org/d/11196-hardened-malloc-functionality

https://discuss.getsol.us/d/11432-solus-os-basic-security-guide-for-new-users

https://discuss.getsol.us/d/4805-solus-security-test/9

https://discuss.getsol.us/d/1038-question-security-of-solus-os

https://discuss.getsol.us/d/146-linux-survival-guide

https://www.siberoloji.com/a-beginners-guide-to-solus-linux-distribution/

https://archive.is/VlvDU

https://archive.ph/QhYIZ

https://ajaykumawat.com/solus-os-in-2024-revolutionizing-the-linux-experience/

https://www.beyondtrust.com/blog/entry/harden-unix-linux-systems-close-security-gaps

https://www.comparitech.com/net-admin/server-hardening-guide/

https://thelinuxcode.com/linux_security_hardening_checklist/

https://madaidans-insecurities.github.io/guides/linux-hardening.html

https://ubuntu.com/blog/what-is-system-hardening-definition-and-best-practices

https://tuxcare.com/blog/linux-hardening/

https://atlantsecurity.com/blog/how-to-properly-harden-your-operating-systems/

https://perception-point.io/guides/os-isolation/os-hardening-10-best-practices/

https://averagelinuxuser.com/after-installing-solus/

https://www.technewsworld.com/story/Solus-Brightens-Computing-Across-the-Linux-User-Spectrum-86305.html

https://securityboulevard.com/2024/07/the-ultimate-guide-to-linux-hardening-focus-on-kernel-security/

https://itsfoss.community/t/my-journey-with-solus-so-far/10888

https://medium.com/@d19cyber/23-things-to-do-after-installing-solus-4-0-fortitude-862562659779

https://github.com/CISOfy/lynis

https://www.kicksecure.com/wiki/Hardened_Malloc#:~:text=On%20Computer,Installation%20Debian&text=A%29%20Hardened,the%20host.

https://forums.gentoo.org/viewtopic-p-8773400.html

https://gitcode.com/gh_mirrors/ha/hardened_malloc

https://forums.whonix.org/t/hardened-malloc-hardened-memory-allocator/7474/82?page=5

https://discuss.privacyguides.net/t/secureblue-atomic-fedora-hardening/16086 and [https://github.com/secureblue/secureblue](https://github.com/secureblue/secureblue)

https://secureblue.dev/

https://bugs-devel.debian.org/cgi-bin/bugreport.cgi?bug=945457

https://blog.csdn.net/gitblog_00710/article/details/142018340

https://blog.csdn.net/gitblog_00008/article/details/143941327

https://blog.csdn.net/gitblog_00140/article/details/142021890

https://github.com/aldorithms/hardened_malloc_explained

https://obscurix.github.io/security/hardened-malloc.html

https://dustri.org/b/using-hardened_malloc-in-alpine-linux.html

https://discuss.grapheneos.org/t/hardened-malloc

https://wiki.alpinelinux.org/wiki/Hardened_malloc

https://crates.io/crates/hardened_malloc-rs

https://dan-kir.github.io/2022/05/22/Experimenting-with-Hardened_malloc.html

https://copr.fedorainfracloud.org/coprs/secureblue/hardened_malloc/

https://lib.rs/crates/hardened_malloc-rs

https://forums.whonix.org/t/hardened-malloc-hardened-memory-allocator/7474

https://aur.archlinux.org/packages/hardened_malloc

https://blog.gitcode.com/0f269a86e8073777fb2b7ce15c74095e.html

https://wiki.archlinux.org/title/Security

https://aur.archlinux.org/packages/hardened_malloc

https://dustri.org/b/musings-on-cve-2023-6246-on-hardened_malloc.html

https://copr.fedorainfracloud.org/coprs/lhajn/hardened_malloc

https://forums.freebsd.org/threads/hardened-malloc-for-freebsd.87000/

https://privsec.dev/posts/linux/desktop-linux-hardening/

https://discussion.fedoraproject.org/t/should-we-implement-hardened-memory-allocator/41270

https://www.cnblogs.com/hahaha111122222/p/14250054.html





---

This guide provides a foundation for hardening Solus 4.7 Endurance. Always test changes in a non-production environment first.



