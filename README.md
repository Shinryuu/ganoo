# Ganoo (The Gentoo Installation Manual)

# Variables
EFI options
```
EFI_OPTS="rw,noatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,utf8,errors=remount-ro"
```

BTRFS options
```
BTRFS_OPTS="rw,noatime,compress=zstd,space_cache=v2,commit=120"
```

Stage tarball to use
```
STAGE_3="https://distfiles.gentoo.org/releases/amd64/autobuilds/20240623T164908Z/stage3-amd64-systemd-20240623T164908Z.tar.xz"
```

# Wipe the disk
```wipefs -a /dev/nvme0n1```

# Partitioning
```
parted -a opt -s /dev/nvme0n1 mklabel gpt
parted -a opt -s /dev/nvme0n1 mkpart primary 1MiB 513MiB
parted -a opt -s /dev/nvme0n1 mkpart primary 513MiB 100%
parted -a opt -s /dev/nvme0n1 set 1 esp on
```

# Encrypt partition(s)
```cryptsetup luksFormat -s 512 /dev/nvme0n1p2```

# BTRFS subvolumes
```
mkdir -p /mnt/gentoo
mount /dev/mapper/ganoo /mnt/gentoo
cd /mnt/gentoo
btrfs subvolume create @
btrfs subvolume create @home
btrfs subvolume create @snaps
umount -R /mnt/gentoo && cd
```

# Mount volumes back
```
mount -o $BTRFS_OPTS,subvol=@ /dev/mapper/ganoo /mnt/gentoo
mkdir -p /mnt/gentoo/{boot,home,.snapshots}
mount -o $EFI_OPTS /dev/nvme0n1p1 /mnt/gentoo/boot
mount -o $BTRFS_OPTS,subvol=@home /dev/mapper/ganoo /mnt/gentoo/home
mount -o $BTRFS_OPTS,subvol=@snaphots /dev/mapper/ganoo /mnt/gentoo/.snapshots
```

# Stage 3
```
cd /mnt/gentoo
wget $STAGE3
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

# Copy personal files
```cp -vr /home/gentoo/ganoo/etc/portage/* /mnt/gentoo/etc/portage/```

# Copy DNS
```cp --dereference /etc/resolv.conf /mnt/gentoo/etc/```

# Generate fstab
```genfstab -U /mnt/gentoo >> /mnt/gentoo/etc/fstab```

# Chroot
```
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --bind /run /mnt/gentoo/run
```
mount --make-slave /mnt/gentoo/run
chroot /mnt/gentoo /bin/bash
