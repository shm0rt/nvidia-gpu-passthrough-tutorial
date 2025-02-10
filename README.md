# GPU Passthrough Tutorial with Nvidia GPU

## 1. Enable IOMMU in BIOS

Before anything, enable IOMMU (Intel VT-d or AMD-Vi) in your BIOS. This setting is usually found under:

- Advanced > CPU Configuration
- Advanced > Chipset Configuration

For Intel CPUs, make sure VT-d is enabled.

## 2. Install the Required Packages

`sudo apt install libvirt-daemon-system libvirt-clients qemu-kvm qemu-utils virt-manager ovmf`

## 3. Get the PCI IDs of the 1080 Ti

Run: `lspci -nn | grep -i nvidia`

You should see output like this:

```bash
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GP102 [GeForce GTX 1080 Ti] [10de:1b06] (rev a1)
01:00.1 Audio device [0403]: NVIDIA Corporation GP102 HDMI Audio [10de:10ef] (rev a1)
```

- 10de:1b06 → GPU ID
- 10de:10ef → HDMI Audio ID

## 4. Enable IOMMU on the Kernel

Modify your **Grub configuration:**

1. Open the **Grub** config

`sudo nano /etc/default/grub`

2. Find the line:

`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`

and modify it to:

`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=on kvm.ignore_msrs=1 vfio-pci.ids=10de:1b06,10de:10ef`

- Replace intel_iommu=on with amd_iommu=on if you have an AMD CPU.
- Replace 10de:1b06,10de:10ef with your GPU and audio device PCI IDs (explained in step 3).

3. Update Grub:

`sudo update-grub`

4. Reboot:

`sudo reboot`

## 5. Bind the 1080 Ti to VFIO

Create a new file:

`sudo nano /etc/modprobe.d/vfio.conf`

Add:

`options vfio-pci ids=10de:1b06,10de:10ef`

Save and exit.

Run:

`sudo update-initramfs -u`

## 5. Blacklist NVIDIA Drivers for the 1080 Ti

Create:

`sudo nano /etc/modprobe.d/blacklist-nvidia.conf`

Add:

```bash
blacklist nouveau
blacklist nvidia
blacklist nvidia-drm
blacklist nvidia-modeset
blacklist nvidia-uvm
```

Save and exit.

## 6. Pass Through the 1080 Ti in Virt-Manager

## 7. Reboot and Verify

`lspci -nnk | grep -A3 -i nvidia`

Your GTX 970 should be used normally, and GTX 1080 Ti should be bound to vfio-pci.
