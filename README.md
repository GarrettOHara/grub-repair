# grub-repair
A guide to repair grub on a LUKS encrypted drive (Ubuntu)

src: https://gist.github.com/samuelcolvin/43c5ed2807e7db004b1058d0c9bfb068
```bash
sudo su
cryptsetup luksOpen /dev/nvme0n1p2 luks_root
vgchange -ay
lvscan
mkdir /media/linux
mount /dev/vgubuntu/root /media/linux/
mount -o bind /proc /media/linux/proc
mount -o bind /dev /media/linux/dev
mount -o bind /sys /media/linux/sys
chroot /media/linux /bin/bash
```

List drives
```bash
fdisk -l
```

Mount EFI
```bash
mount /dev/nvme0n1p1 /boot/efi
```

NOTE: If you want to install LUKS encryption for GRUB CLI jump [here](#enable-luks-support-for-grub)

Install GRUB on drive
```bash
grub-install /dev/nvme0n1
grub-update
```

Unmount
```bash
umount /boot/efi
exit
```

Unmount drive
```bash
umount /media/linux/boot/efi
umount -l /media/linux
```

Re-encrypt LUKS drive
```bash
vgchange-an
cryptsetup luksClose luks_root
```

# Enable LUKS Support for GRUB

src: https://www.reddit.com/r/archlinux/comments/10pq74e/my_easy_method_for_setting_up_secure_boot_with/?rdt=52103
```bash
sudo grub-install --target=x86_64-efi --efi-directory=esp --bootloader-id=GRUB --modules="tpm" --disable-shim-lock
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
**Key Options for grub-install**

Here are some common flags you can use with grub-install:

--modules="module1 module2": Preloads specific GRUB modules. For LUKS support, you might preload luks and cryptodisk like this:

```bash
grub-install /dev/nvme0n1 --modules="luks cryptodisk"
```
- `--target=`: Specifies the target platform (e.g., x86_64-efi, i386-pc, etc.).
- `--boot-directory=`: Specifies the directory where the boot files are installed.
- `--efi-directory=`: Specifies the EFI system partition (for UEFI systems).

However, even if you preload the `luks` and `cryptodisk` modules during installation, you still need to configure GRUB to use these features in the actual boot configuration files. This involves:

- Enabling LUKS support in `/etc/default/grub`: As mentioned before, you need to set `GRUB_ENABLE_CRYPTODISK=y` in this file.
- Regenerating GRUB configuration: After setting the options, you need to run `update-grub` to regenerate the `/boot/grub/grub.cfg` file with the new settings.

### Example: Full Workflow

*Here’s how you would perform a full GRUB installation with `LUKS` support:*

Install GRUB with `LUKS` and `cryptodisk` modules preloaded:

```bash
sudo grub-install /dev/nvme0n1 --modules="luks cryptodisk"
```

Edit `/etc/default/grub` to enable `cryptodisk`:

```bash
sudo vim /etc/default/grub
```

Add or modify the following lines:

```bash
GRUB_ENABLE_CRYPTODISK=y
GRUB_PRELOAD_MODULES="luks cryptodisk"
```

Regenerate the GRUB configuration:


```bash
update-grub
```

Reboot and Test: Reboot the system to ensure that GRUB prompts for the LUKS passphrase and can unlock the encrypted partition.

In summary, while you can use grub-install with flags like `--modules` to include support for `LUKS` and `cryptodisk`, you still need to configure the GRUB environment through configuration files (e.g., `/etc/default/grub`) and regenerate the GRUB configuration to ensure that the `cryptodisk` features are correctly set up during boot.

### Command summary

Add these lines to `/etc/default/grub`:
```
GRUB_ENABLE_CRYPTODISK=y
GRUB_PRELOAD_MODULES="luks cryptodisk"
```

```bash
sudo grub-install /dev/nvme0n1 --modules="luks cryptodisk"
sudo vim /etc/default/grub
update-grub
```
