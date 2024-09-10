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
