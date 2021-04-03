# Lenovo Y530 GPU passthru script

Script is used on run virtual windows 10 with Nvidia GPU passthru.
Requirments:
* Archlinux
* Qemu 5.2.0 compiled with: 
<pre>
../configure --prefix=/usr --sysconfdir=/etc --target-list=x86_64-softmmu --libexecdir=/usr/lib/qemu --localstatedir=/var --bindir=/usr/bin/ --enable-gnutls --enable-gtk --enable-vnc --enable-vnc-sasl --enable-vnc-png --enable-vnc-jpeg --enable-curl --enable-kvm --enable-linux-aio --enable-cap-ng --enable-vhost-net --enable-vhost-crypto --enable-spice --enable-usb-redir --enable-lzo --enable-snappy --enable-bzip2 --enable-coroutine-pool --enable-libxml2 --enable-tcmalloc --enable-replication --enable-tools --enable-capstone --audio-drv-list=alsa,pa,oss --enable-sdl --enable-vnc --enable-opengl --enable-libusb --enable-guest-agent --enable-curses --enable-kvm --enable-virglrenderer --enable-plugins --enable-vvfat --enable-vdi --enable-qed --enable-qcow1 --enable-dmg --enable-virtfs --enable-libudev --enable-vde --enable-linux-aio --enable-linux-io-uring --enable-cap-ng --enable-vhost-net --enable-vhost-vsock --enable-vhost-scsi --enable-libiscsi --enable-libnfs --enable-libusb --enable-tpm --enable-avx2 --enable-avx512f --enable-opengl --enable-tools --enable-bochs --enable-cloop --enable-parallels --enable-crypto-afalg --enable-debug --enable-vte
</pre>

* Arch linux vfio kernel - 5.11.6 - https://aur.archlinux.org/packages/linux-vfio/
* Spice server for usb passthru
* Intel GVT-g for vm screen:
/etc/systemd/system/gvtvgpu.service
<pre>
[Unit]
Description=Create Intel GVT-g vGPU

[Service]
Type=oneshot
ExecStart=/bin/sh -c "echo '1ae40c36-b180-4af0-8fab-c27de21f597d' > /sys/devices/pci0000:00/0000:00:02.0/mdev_supported_types/i915-GVTg_V5_2/create"
ExecStop=/bin/sh -c "echo '1' > /sys/devices/pci0000:00/0000:00:02.0/1ae40c36-b180-4af0-8fab-c27de21f597d/remove"
RemainAfterExit=yes

[Install]
WantedBy=graphical.target
</pre>

* Vfio device id assignemnt:
/etc/modprobe.d/vfio.conf
<pre>
options vfio-pci ids=10de:1c8c,10de:0fb9
</pre>

* Vfio module asignment:
In /etc/mkinitcpio.conf:
<pre>
MODULES=(vfio vfio_iommu_type1 vfio_pci vfio_virqfd vfio-mdev kvmgt)
</pre>

* Grub configuration:
In /etc/default/grub:
<pre>
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet modprobe.blacklist=nouveau modprobe.blacklist=nvidia vfio-pci.ids=10de:1c8c,10de:0fb9 i915.enable_gvt=1 intel_iommu=on iommu=pt rd.driver.pre=vfio-pci video=efifb:off rd.driver.blacklist=nouveau kvm.ignore_msrs=1 vfio_iommu_type1.allow_unsafe_interrupts=1 kvm.allow_unsafe_assigned_interrupts=1 intel_iommu=igfx_off"</pre>


## Usage:

Run script with:
<pre>sudo ./windows_10_intel_nvidia.sh win10_uefi</pre>
where parameter is your qcow2 img name in same folder as this script
