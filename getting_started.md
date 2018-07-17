# Setting up your ThreeFold Farm from Mac OSX

Steps:
- [Create an ItsYou.online organization](#iyo-org)
- [Register your farm](#register)
- [Download the EFI (UEFI) bootable image](#download)
- [Format your USB drive](#format)
- [Copy EFI image to USB flash drive](#copy)
- [Boot your node](#boot)

Also see the [Threefold farmers documentation](https://github.com/zero-os/home/blob/master/docs/farmers/README.md#configure-your-nodes).

<a id='iyo-org'></a>

## Create an ItsYou.online organization

Your ThreeFold Farm needs to be associated to an ItsYou.online organization. 

In order to create such an ItsYou.online organization you need an ItsYou.online account. Creating one is simple, head to http://itsyou.online/ and follow the sign-up instructions.

Once logged in to your ItsYou.online account, also creating an ItsYou.online organization is simple.

Click **Organization** from the **Profile** menu in the top right corner:
![](https://raw.githubusercontent.com/zero-os/home/master/docs/farmers/images/iyo-organizations.png)

Now click **+ CREATE NEW**:
![](https://raw.githubusercontent.com/zero-os/home/master/docs/farmers/images/iyo-create-new-org.png)

And finally specify the name of the new ItsYou.online organization and click **CREATE**:
![](https://raw.githubusercontent.com/zero-os/home/master/docs/farmers/images/iyo-create-new-org2.png)

> For the name of the ItsYou.online organization only lower case characters, numbers or spaces are allowed. 

> The name of ItsYou.online organization needs to be globally unique, and therefor also referenced to as the **Global ID** of the ItsYou.online organization.


<a id='register'></a>

## Register your farm

Go to the ThreeFold Grid Capacity web site: https://capacity.threefoldtoken.com

![](images/capacity.png)

In the top right corner click **Register Farm**:

![](images/capacity2.png)

Here you specify:
- **Farm Name**: this can be anything, the name will be displayed next to your node in the listing of the capacity
- **Organization ID**: the global ID of the ItsYou.online organization you created in the previous step
- **TFT Wallet Address**: optionally you can also associate a ThreeFold Token wallet address to your farm
- **Farm Location**: location on the map of your farm

The result of your registration will be a JSON Web Token (JWT), which is your **farmer ID**. Make sure to copy this token and keep it, because you will need it later in the process of setting up your nodes.


<a id='download'></a>

## Download the EFI (UEFI) bootable image

On https://bootstrap.gig.tech click the **Assistant** button:

![](images/bootstrap.png)

In **step 1** of the Zero-OS **Boot Generator Assistant** page select Zero-OS branch `v1.4.1`:

![](images/bootstrap1.png)

Next, in **step 2**, specify the ID of ZeroTier management address, the default one to use is `c7c8172af1f387a6`:

![](images/bootstrap2.png)

In **step 3**, pass your `farmer_id={$farmer_id}` as a kernel parameter:

![](images/bootstrap3.png)

Optionally, in order to boot your ThreeFold node in development node, you can also add the `%20development`: `farmer_id={$farmer_id}%20development`

In **step 4**, next to **EFI** you can see the EFI (UEFI) bootable image download URL for your ThreeFold node:

![](images/bootstrap4.png)

You might need to scroll to the right in order to see the **EFI Kernel** download button:

![](images/bootstrap5.png)

Or just copy the download URL into your browser:

https://bootstrap.gig.tech/uefi/v1.4.1/c7c8172af1f387a6/farmer_id=<id>

![](images/download.png)


<a id='format'></a>

## Format your USB drive

Start by connecting your USB drive to your Mac OS X computer.

Search for **Disk Utility** in Launchpad and open it:

![](images/disk_utility.png)

Select your USB drive and click **Erase**.

In the dialog that pops up  optionally enter a new name for the USB drive, and then select **MS-DOS(FAT)** from the Format dropdown menu:

![](images/disk_utility2.png)

Click **Erase** which will format your USB disk and report about the result:

![](images/disk_utility3.png)


The above can also be achieve from the command line:
```bash
diskutil list
diskutil eraseDisk FAT32 "ZOS" /dev/diskX
```

> Caution, in the above replace `/dev/diskX` with the device file of your USB disk, which you can find by checking the `diskutil list` first, make sure to not shoot yourself in the feet here.

<a id='copy'></a>

### Format your USB drive

Open **Finder** and select your formated USB drive from **Devices**:

![](images/finder.png)

Create a root folder **EFI**:

![](images/finder2.png)

![](images/finder3.png)

In the new root folder create a sub directory **BOOT**:

![](images/finder4.png)

![](images/finder5.png)

Locate your downloaded EFI image, here under the downloads:

![](images/finder6.png)

Rename `ipxe-v1.4.1.efi` to `BOOTX64.EFI`:

![](images/finder7.png)

![](images/finder8.png)

Copy this file to the previously created folder `/EFI/BOOT` on your USB device:

![](images/finder9.png)

![](images/finder10.png)

![](images/finder11.png)

And finally, eject your USB drive:

![](images/finder12.png)

Or all the above from the command line:
```bash
mkdir -p /Volumes/ZOS/EFI/BOOT/
wget -O /Volumes/ZOS/EFI/BOOT/BOOTX64.EFI https://bootstrap.gig.tech/uefi/v1.4.1/c7c8172af1f387a6/farmer_id=eyJhbGciOiJFUzM4NCIsInR5cCI6IkpXVCJ9.eyJhenAiOiJ0aHJlZWZvbGQuZmFybWVycyIsImV4cCI6MTUyODI4MzY3MSwiaXNzIjoiaXRzeW91b25saW5lIiwicmVmcmVzaF90b2tlbiI6IlBJcGQ4QmlfOXAyd1drYlB0dHQ4SEZpSEJXSk4iLCJzY29wZSI6WyJ1c2VyOm1lbWJlcm9mOnl2ZXNmYXJtIl0sInVzZXJuYW1lIjoieXZlcyJ9.8siq1Tk_b6ZzM675K4Aq3SYwS5J8Lk_5W5XSIbOrUgikJteTbmNzClOPNV1gTJVOFhfE4c-f1AEX2M4GM-Gs69cqpi1_YgXq_RPJvz6JuCbJdR8xBkJjgOfI7FS8PnUq%20development
diskutil umount /Volumes/ZOS
```

<a id='boot'></a>

## Boot your node

Boot your node with the USB drive and check the status of your farm on https://capacity.threefoldtoken.com:

![](images/farm.png)

Under **Resource Units** you can click the **details** button which will bring up the details of the selected node:

![](images/farm_details.png)