# Ubuntu 16.04 auto-Install for UEFI using preseed

### Preseeding methods
As mentioned in the [official installation guide](https://www.debian.org/releases/stable/i386/apb.en.html), there are several ways to feed the preseed file to the installer. And in this guide, I will use preseed via `Adding the preseed file to the installer's initrd.gz`.

Get Ubuntu image.

    cd /tmp/
    wget http://releases.ubuntu.com/16.04.2/ubuntu-16.04.2-server-amd64.iso

### Extract ISO
    xorriso -osirrox on -indev ubuntu-16.04.2-server-amd64.iso -extract / custom-iso

### Change dir/file permission to write access
    chmod 700 custom-iso/preseed custom-iso/boot/grub/grub.cfg custom-iso/isolinux/txt.cfg custom-iso/isolinux/isolinux.cfg 

### Adding the preseed.cfg to Ubuntu preseed folder
    cp preseed.cfg custom-iso/preseed/ 

### Edit boot/grub/grub.cfg
This is for UEFI booting. This specifies the timeout on the grub menu to 1 second, tells grub what network adapter to use and grabs the preseed file.

    cd custom-iso
    vi boot/grub/grub.cfg

Replace the contents with grub.cfg with the following.

    if loadfont /boot/grub/font.pf2 ; then
         set gfxmode=auto
         insmod efi_gop
         insmod efi_uga
         insmod gfxterm
         terminal_output gfxterm
    fi
    set default=0
    set timeout=1
    set menu_color_normal=white/black
    set menu_color_highlight=black/light-gray

### Edit isolinux/txt.cfg

    vi isolinux/txt.cfg

Replace the contents with txt.cfg with the following.

    default install
        label install
        menu label ^Auto Install Ubuntu Server
        kernel /install/vmlinuz
        append  file=/cdrom/preseed/preseed.cfg vga=788 initrd=/install/initrd.gz quiet debian-installer/language=en_US:en localechooser/supported-locales=en_US.UTF-8 debian-installer/locale=en_US.UTF-8 debian-installer/country=US keyboard-configuration/layoutcode=us --

### Edit isolinux/isolinux.cfg

    vi isolinux/isolinux.cfg

Replace the contents with isolinux.cfg with the following.

    path 
    include menu.cfg
    prompt 0
    timeout 1

## Obtain isohdpfx.bin for hybrid ISO

    cd ../
    sudo dd if=ubuntu-16.04.2-server-amd64.iso bs=512 count=1 of=custom-iso/isolinux/isohdpfx.bin

## Create new ISO

    cd custom-iso/
    sudo xorriso -as mkisofs -isohybrid-mbr isolinux/isohdpfx.bin -c isolinux/boot.cat -b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table -eltorito-alt-boot -e boot/grub/efi.img -no-emul-boot -isohybrid-gpt-basdat -o ../custom-ubuntu.iso .

## Confirm partitions
As we have created a hybrid ISO, meaning it can be used in both UEFI and Legacy modes, it's good to make sure EFI is showing in the partition table. You can check this by using the following command:

    cd ..
    fdisk -l custom-ubuntu.iso

If the partitions have been created correctly, you should see something similar to the following:

    Disk custom-ubuntu.iso: 841 MiB, 881852416 bytes, 1722368 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x694f323a

    Device             Boot Start     End Sectors  Size Id Type
    custom-ubuntu.iso1 *        0 1722367 1722368  841M  0 Empty
    custom-ubuntu.iso2       4036    8899    4864  2.4M ef EFI (FAT-12/16/32)

## User passwords
You can either encrypt your password, or pass it over plain text. If you want to use plain text, then comment the line `d-i passwd/user-password-crypted` and uncomment the two lines above containing `passwd/user-password` and `passwd/user-password-again`. __Don't forget to update your password__. If you would like to hash the password, enter the following line. This will prompt for a password and return a hash for you to use. You must then add your hash to the preseed.cfg file. 
    
    mkpasswd -m sha-512

