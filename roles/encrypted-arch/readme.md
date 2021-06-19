# Archible - Arch Installer using [Anible]

A pet project to configure ArchLinux and quickly deploy new systems,
in virtual machines, cloud instances or local hardware. I have given
more emphasis to the latter one, thus, for now the playbook to install
the base system, ecrypts the system drive with the intention to use
secure boot.

The `install.yml` playbook installs a small number of packages and minimal
configurations meant to be fast, so it can be used for testing.
With the addition of a small vitual drive to keep pacman's cache directory,
I've managed to reduces the installation to less than 150s.

All tasks or blocks are tagged aproprieatly to help debug changes and new tasks.
Tags with `always` tag are "dependencies" to other tasks and are always executed.
The `never` and `always` tags are special tags. Please read the ansible docs to
learn more about ansible if you've never used it before.

## install.yml

**WARNING: This playbook wipes the entire drive (`install_drive` variable).**

To use this:

- Create a keyfile on your local host containing the password for
your LUKS root volume (`echo -n "your_password" > keyfile`).

- Generate a hash for the password to be used on your personal
account using `mkpasswd --method=sha-512`. Then set the `user_password`
varible as the output of that command.

- `vars` in the install playbook are the ones I use in testing. It's also
a way to make sure I do not nuke the wrong drive if I forget to change the
`install_drive` variable. `/dev/vda` is a virtual device when using a vm.
`root_partition` and `efi_partition` must be changed to `p1` and `p2` when
using an nvme drive.

The partition scheme is really simple, hense the rest of the set-up is quite
complicated. The first partition is the EFI, mounted at `/efi`. The rest is
the root partition, which includes boot and is ecrypted with LUKS.

`/boot` is encrypted along with the rest of the disk, using luks2.
A Unified Kernel Image is created in the EFI directory so that
systemd-boot can be used as a bootloader, instead of Grub.
Both start-up time and the installation take less time this way.

Btrfs is the fileystem of choice. I've created some extra subvolumes for
specific `/var` directories that either don't need to be backed-up, or
shouldn't be rolled-back along with the rest of the root (`@`) subvolume.

To run the playbook against localhost, without an inventory file use:

```sh
ansible-playbook install.yml -c local -i "localhost,".
```

To connect to the vm without adding an entry to the Knownhosts, run:

```sh
ssh root@192.168.122.150 -i ~/.ssh/git -o "StrictHostKeyChecking no" -o "UserKnownHostsFile /dev/null"
```

To quickly remount all the partitions and use arch-chroot after restart
run `ansible-playbook install.yml --tags post_restart_mount`.

### Future plans

- I am considering adding suppport for non-UEFI Bios.

</br>
</br>
</br>
## Sources

[dm-crypt main page](https://wiki.archlinux.org/index.php/Dm-crypt)

[Microcode @ArchWiki](https://wiki.archlinux.org/index.php/Microcode#Early_loading)

[systemd-boot efistub?](https://wiki.archlinux.org/index.php/EFI_system_partition#Using_systemd)

[Encrypted boot - Grub - Grub settings](https://wiki.archlinux.org/index.php/GRUB#Encrypted_/boot)

[Encrypted boot - dmcrypt - Grub settings](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#Encrypted_boot_partition_(GRUB))

[Unified Kernel Image - Grub entry](https://wiki.archlinux.org/index.php/GRUB#Chainloading_a_unified_kernel_image)

[Unified Kernel Image - microcode](https://wiki.archlinux.org/index.php/microcode#Unified_kernel_images)

[Secure boot @ArchWiki](https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface/Secure_Boot)

[btrfs @ArchWiki](https://wiki.archlinux.org/index.php/btrfs)

[Ansible - custom modules](https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html)
