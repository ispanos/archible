# Task notes

Create demo gif
https://github.com/andreyv/sbupdate
https://gist.github.com/urwx/738e40217c4c7fd4ac896c7a4b71ba9e
https://github.com/desbma/pacman-hooks

[reflector](https://wiki.archlinux.org/index.php/Reflector#Pacman_hook)
[hashicorp vaultproject](https://www.vaultproject.io/)
[Secure Linux](https://christitus.com/secure-linux/)

## Systemd boot

```yml
- name: Add loader Configuration
  copy:
    content: |
      title   Arch Linux
      linux   /vmlinuz-linux
      initrd  /${cpu}-ucode.img
      initrd  /initramfs-linux.img
      options root=UUID=${root_id} $kernel_parms
    dest: /mnt/boot/loader/entries/ArchLinux.conf
```

### Alternative options for unencrypted boot partition

```ini
options {{ extra_kernel_parameters }} rd.luks.name={{ root_luks_uuid.stdout }}=root rd.luks.options=timeout=90s,cipher=aes-xts-plain64:sha256,size=512 root=/dev/mapper/root
```

</br>

```yml
- name: Setup netctl
  block:
    - name: Create netctl profile for wired connection
      copy:
        content: |
          Description='Wired with DHCP'
          Interface={{ wired_interface }}
          Connection=ethernet
          IP=dhcp
          IP6=dhcp-noaddr
          # IPv6 traffic is tunneled over IPv4, which eats 20 bytes of the MTU.
          ExecUpPost='/usr/bin/ip link set {{ wired_interface }} mtu 1480'
        dest: /mnt/etc/netctl/wired
    - name: Enable wired netctl profile
      command: arch-chroot /mnt netctl enable wired
  tags:
    - netctl
```
