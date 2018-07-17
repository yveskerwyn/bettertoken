# How to create an EFI booteable USB disk on Linux



List all block devices:
```bash
lsblk
```

Format:
```bash
sudo parted -a optimal -s /dev/disk2 mklabel msdos
sudo parted -a optimal -s /dev/disk2 mkpart primary fat32 2048s 100%
sudo parted -a optimal -s /dev/disk2 set 1 boot # not strictly necessary
sudo mkfs.vfat /dev/sdb1
```

Mount:
```bash
mkdir /tmp/zosboot
mount /tmp/zosboot /dev/disk2
```

Copy image:
```bash
mkdir -p /tmp/zosboot/EFI/BOOT/
mv ipxescript /tmp/zosboot/EFI/BOOT/BOOTX64.EFI
```

Unmount:
```bash
umount /tmp/zosboot
```

Cleanup:
```bash
rm -rf /tmp/zosboot
```