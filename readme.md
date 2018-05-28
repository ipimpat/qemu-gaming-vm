# NVIDIA GPU passthrough to Windows 10 guest with QEMU/KVM on Linux Mint 18

This is an example of how to passthrough an NVIDIA GPU to a virtual Windows 10 on Linux Mint 18.3 using QEMU/KVM with crackle free audio output and massively improved audio input delay.

- Hugepage memory backing for virtual memory tuning.
- VirtIO is used for storage and network for best performance.
- Intel HD graphics is used for host video.

Please note, that this is still work in progress, and some information is missing.

## Hardware used in this example

**CPU:** Intel Core i7 7700K
**RAM:** 32 GB
**GPU:** Gainward GTX 1070 8GB
**Motherboard:** ASUS STRIX Z270F GAMING
**Storage:** Intel 512GB SSD (storage pool)

## Install QEMU and libvirt

todo

### Configure libvirt

/etc/libvirt/qemu.conf

    user = "1000"

### Compile patched QEMU emulator with improved hda/pulseaudio

    mkdir -p ~/src
    git clone https://github.com/spheenik/qemu.git ~/src/qemu-spheenik
    mkdir ~/src/qemu-spheenik/build && cd ~/src/qemu-spheenik/build
    ../configure --prefix=/opt/qemu-spheenik --python=/usr/bin/python2 --target-list=x86_64-softmmu --audio-drv-list=pa --disable-werror
    make
    sudo make install

## Host configuration

### Define CPU isolation kernel parameters

/etc/default/grub.d/isolate_cpus.conf

    GRUB_CMDLINE_LINUX_DEFAULT="$GRUB_CMDLINE_LINUX_DEFAULT isolcpus=2,3,6,7"

### Define intel_iommu kernel parameters

/etc/default/grub.d/intel_iommu.conf

    GRUB_CMDLINE_LINUX_DEFAULT="$GRUB_CMDLINE_LINUX_DEFAULT intel_iommu=on"

### Load KVM kernel modules

/etc/modules-load.d/kvm.conf 

    kvm
    kvm_intel

### Load VFIO kernel modules

/etc/modules-load.d/vfio.conf 

    vfio
    vfio_pci
    vfio_virqfd
    vfio_iommu_type1

### Blacklist closed and opensource nvidia drivers

/etc/modprobe.d/blacklist-video.conf

    blacklist nouveau
    blacklist nvidia

### Define KVM kernel modules options

/etc/modprobe.d/kvm.conf

    options kvm_intel nested=1

### Define VFIO kernel modules options

Discover the video and audio IDs of the NVIDIA vga device using this command

    lspci -nn | grep NVIDIA

/etc/modprobe.d/vfio.conf

    options vfio-pci ids=10de:1b81,10de:10f0
    options vfio_iommu_type1 allow_unsafe_interrupts=1

### Allocate hugepages

/etc/sysctl.d/60-hugepages.conf

    # Allocate 8192 HugePageTables (16GB)
    vm.nr_hugepages = 8192

### Regenerate grub configuration

    sudo update-grub

### Regenerate initramfs image
    
    sudo update-initramfs -u


## Windows 10 libvirt configuration

### XML dump

Full libvirt domain XML [domain xml](win10.xml)
