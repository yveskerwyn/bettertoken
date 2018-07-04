# Getting started

## Non-connected node


Based on this input you can download the UEFI boot loader executable for your Zero-OS node:
```
ipxe: https://bootstrap.gig.tech/ipxe/development/9f77fc393e7fd8b4/organization=zos-training-org%20development
```




## Connecting to the ThreeFold Grid

http://tffarmers.com/http://tffarmers.com/

Also see the [Threefold farmers documentation](https://github.com/zero-os/home/blob/master/docs/farmers/README.md#configure-your-nodes).

farmer_id:
```
eyJhbGciOiJFUzM4NCIsInR5cCI6IkpXVCJ9.eyJhenAiOiJ0aHJlZWZvbGQuZmFybWVycyIsImV4cCI6MTUyODI4MzY3MSwiaXNzIjoiaXRzeW91b25saW5lIiwicmVmcmVzaF90b2tlbiI6IlBJcGQ4QmlfOXAyd1drYlB0dHQ4SEZpSEJXSk4iLCJzY29wZSI6WyJ1c2VyOm1lbWJlcm9mOnl2ZXNmYXJtIl0sInVzZXJuYW1lIjoieXZlcyJ9.8siq1Tk_b6ZzM675K4Aq3SYwS5J8Lk_5W5XSIbOrUgikJteTbmNzClOPNV1gTJVOFhfE4c-f1AEX2M4GM-Gs69cqpi1_YgXq_RPJvz6JuCbJdR8xBkJjgOfI7FS8PnUq
```

Wallet address:
```
01928f832c3146daa91cff1a6c9057ba52657e4e403250d553b1a1b8c1ed22a240df0f7907454d
```


On https://bootstrap.gig.tech click **Generator** and select Zero-OS branch `v1.4.1` in step 1:

![](images/generator.png)

Next, in step 2, specify the ID of ZeroTier management address, the default one to use is `c7c8172af1f387a6`:

![](images/zerotier.png)

In step 3, pass `farmer_id={$farmer_id}` as a kernel parameter:

![](images/kernelparams.png)


Based on this input you can download the UEFI boot loader executable for your Zero-OS node:
```
 https://bootstrap.gig.tech/uefi/v1.4.1/c7c8172af1f387a6/farmer_id=eyJhbGciOiJFUzM4NCIsInR5cCI6IkpXVCJ9.eyJhenAiOiJ0aHJlZWZvbGQuZmFybWVycyIsImV4cCI6MTUyODI4MzY3MSwiaXNzIjoiaXRzeW91b25saW5lIiwicmVmcmVzaF90b2tlbiI6IlBJcGQ4QmlfOXAyd1drYlB0dHQ4SEZpSEJXSk4iLCJzY29wZSI6WyJ1c2VyOm1lbWJlcm9mOnl2ZXNmYXJtIl0sInVzZXJuYW1lIjoieXZlcyJ9.8siq1Tk_b6ZzM675K4Aq3SYwS5J8Lk_5W5XSIbOrUgikJteTbmNzClOPNV1gTJVOFhfE4c-f1AEX2M4GM-Gs69cqpi1_YgXq_RPJvz6JuCbJdR8xBkJjgOfI7FS8PnUq
```


## On Mac

First list all your disk devices:
```bash
diskutil list
```

Format the USB device:
```bash
diskutil eraseDisk FAT32 "ZOS" /dev/disk2
```

As a result the formated disk will get mounted (into `/Volumes/ZOS`), check all mount points:
```bash
mount
```

Copy the EFI file - connected to the TF Grid:
```bash
mkdir -p /Volumes/ZOS/EFI/BOOT/
wget -O /Volumes/ZOS/EFI/BOOT/BOOTX64.EFI https://bootstrap.gig.tech/uefi/v1.4.1/c7c8172af1f387a6/farmer_id=eyJhbGciOiJFUzM4NCIsInR5cCI6IkpXVCJ9.eyJhenAiOiJ0aHJlZWZvbGQuZmFybWVycyIsImV4cCI6MTUyODI4MzY3MSwiaXNzIjoiaXRzeW91b25saW5lIiwicmVmcmVzaF90b2tlbiI6IlBJcGQ4QmlfOXAyd1drYlB0dHQ4SEZpSEJXSk4iLCJzY29wZSI6WyJ1c2VyOm1lbWJlcm9mOnl2ZXNmYXJtIl0sInVzZXJuYW1lIjoieXZlcyJ9.8siq1Tk_b6ZzM675K4Aq3SYwS5J8Lk_5W5XSIbOrUgikJteTbmNzClOPNV1gTJVOFhfE4c-f1AEX2M4GM-Gs69cqpi1_YgXq_RPJvz6JuCbJdR8xBkJjgOfI7FS8PnUq
```

Or if not connected:
```bash
mkdir -p /Volumes/ZOS/EFI/BOOT/
wget -O /Volumes/ZOS/EFI/BOOT/BOOTX64.EFI https://bootstrap.gig.tech/uefi/development/9f77fc393e7fd8b4/organization=zos-training-org%20development
```

Unmount:
```bash
diskutil umount /Volumes/ZOS
```

Now remove the USB from your Mac and boot the node.

## Linux:


List all block devices:
```bash
lsblk
```

```bash
sudo parted -a optimal -s /dev/disk2 mklabel msdos
sudo parted -a optimal -s /dev/disk2 mkpart primary fat32 2048s 100%
sudo parted -a optimal -s /dev/disk2 set 1 boot # not strictly necessary
sudo mkfs.vfat /dev/sdb1

mkdir /tmp/zosboot
mount /tmp/zosboot /dev/disk2

mkdir -p /tmp/zosboot/EFI/BOOT/
mv ipxescript /tmp/zosboot/EFI/BOOT/BOOTX64.EFI
umount /tmp/zosboot
rm -rf /tmp/zosboot
```


## ThreeFold Grid Capacity

https://capacity.threefoldtoken.com/?cru=0&mru=0&hru=0&sru=0&country=&farmer=yvesfarm




