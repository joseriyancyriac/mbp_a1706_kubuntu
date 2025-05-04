# mbp_a1706_kubuntu
My personal fallback protocal for re-installing Kubuntu 22.04
# Kubuntu 22.04 on MBP A1706

## System Specs

```bash
# dump basic HW summary to the screen
inxi -Faz | less
```

> [!NOTE]
> Install once with
> `sudo apt install inxi`
> ( `inxi` prints CPU, GPU, disks, battery, temps, etc. in an easy‑read block )

---

```bash
System:
  Host: Eclipse Kernel: 6.8.0-59-generic x86_64 bits: 64 compiler: N/A
    Desktop: KDE Plasma 5.24.7 tk: Qt 5.15.3 wm: kwin_x11 dm: SDDM
    Distro: Ubuntu 22.04.5 LTS (Jammy Jellyfish)
Machine:
  Type: Laptop System: Apple product: MacBookPro14,2 v: 1.0
    serial: <superuser required> Chassis: type: 9 v: Mac-CAD6701F7CEA0921
    serial: <superuser required>
  Mobo: Apple model: Mac-CAD6701F7CEA0921 v: MacBookPro14,2
    serial: <superuser required> UEFI: Apple v: 529.140.2.0.0 date: 06/23/2024
Battery:
  ID-1: BAT0 charge: 40.1 Wh (80.2%) condition: 50.0/49.2 Wh (101.7%)
    volts: 11.8 min: 11.4 model: SMP bq20z451 serial: N/A status: Discharging
CPU:
  Info: dual core model: Intel Core i5-7267U bits: 64 type: MT MCP
    arch: Amber/Kaby Lake note: check rev: 9 cache: L1: 128 KiB L2: 512 KiB
    L3: 4 MiB
  Speed (MHz): avg: 1175 high: 3500 min/max: 400/3500 cores: 1: 400 2: 400
    3: 400 4: 3500 bogomips: 24799
  Flags: avx avx2 ht lm nx pae sse sse2 sse3 sse4_1 sse4_2 ssse3 vmx
Graphics:
  Device-1: Intel Iris Plus Graphics 650 vendor: Apple driver: i915 v: kernel
    ports: active: eDP-1 empty: DP-1, DP-2, HDMI-A-1, HDMI-A-2 bus-ID: 00:02.0
    chip-ID: 8086:5927
  Device-2: Apple iBridge type: USB
    driver: apple-ibridge-hid,usbhid,uvcvideo bus-ID: 1-3:2 chip-ID: 05ac:8600
  Display: x11 server: X.Org v: 1.21.1.4 compositor: kwin_x11 driver: X:
    loaded: modesetting unloaded: fbdev,vesa gpu: i915 display-ID: :0
    screens: 1
  Screen-1: 0 s-res: 1920x1200 s-dpi: 96
  Monitor-1: eDP-1 model: Apple Color LCD res: 1920x1200 dpi: 171
    diag: 337mm #(13.3')
  OpenGL:
    renderer: Mesa Intel Iris Plus Graphics 650 (Kaby Lake GT3e) (KBL GT3)
    v: 4.6 Mesa 23.2.1-1ubuntu3.1~22.04.3 direct render: Yes
Audio:
  Device-1: Intel Sunrise Point-LP HD Audio driver: snd_hda_intel v: kernel
    bus-ID: 00:1f.3 chip-ID: 8086:9d71
  Sound Server-1: ALSA v: k6.8.0-59-generic running: yes
  Sound Server-2: PulseAudio v: 15.99.1 running: yes
  Sound Server-3: PipeWire v: 0.3.48 running: yes
Network:
  Device-1: Broadcom BCM43602 802.11ac Wireless LAN SoC vendor: Apple
    driver: brcmfmac v: kernel pcie: speed: 2.5 GT/s lanes: 1 bus-ID: 02:00.0
    chip-ID: 14e4:43ba
  IF: wlp2s0 state: down mac: 00:90:4c:0d:f4:3e
  Device-2: TP-Link UE300 10/100/1000 LAN (ethernet mode) [Realtek RTL8153]
    type: USB driver: r8152 bus-ID: 6-2:2 chip-ID: 2357:0601
  IF: enx6032b1450d3f state: up speed: 100 Mbps duplex: full
    mac: 60:32:b1:45:0d:3f

```

## Prep

```bash
sudo apt update && sudo apt full-upgrade
# pull in the HWE stack – newer kernels help a lot
sudo apt install linux-generic-hwe-22.04 linux-firmware gedit
```
---

## Wi-Fi – Broadcom BCM43602

1. Install the open driver (already loaded) plus the missing NVRAM file:

```bash
sudo apt install --reinstall linux-firmware  # makes sure *.bin + *.clm_blob are there
cd /tmp
wget -O brcmfmac43602-pcie.txt https://bugzilla.kernel.org/attachment.cgi?id=285753
```
---

2. Edit Config TXT

```bash
gedit brcmfmac43602-pcie.txt
```
> [!IMPORTANT]
>```txt
> macaddr=00:90:4c:0d:f4:3e      # your MAC adddress with ip link show wlp2s0 | awk '/ether/ {print $2}'
> ccode=DE                       # ← use your 2-letter country code
> regrev=1
> .
> .
> .
> .
> .
> boardflags3=0x00000303
>```
---

3. Install and reload teh driver

```bash
sudo mv brcmfmac43602-pcie.txt /usr/lib/firmware/brcm/
sudo reboot
```
---

## Audio - Cirrus-8409

1.  Install build prerequisites

```bash
sudo apt update
sudo apt install build-essential dkms linux-headers-$(uname -r) git
```
---

2. grab the source .deb (latest build)

```bash
cd /tmp
wget http://archive.ubuntu.com/ubuntu/pool/main/l/linux/linux-source-6.8.0_6.8.0-57.59_all.deb
sudo dpkg -i linux-source-6.8.0_6.8.0-57.59_all.deb         # installs /usr/src/linux-source-6.8.0.tar.xz
```
---

3. Give the file the exact name the script looks for (it wants .tar.bz2)

```bash
cd /usr/src
sudo ln -s linux-source-6.8.0.tar.xz linux-source-6.8.0.tar.bz2
```

---

4.  Grab & install the Cirrus-8409 DKMS driver

```bash
cd
git clone https://github.com/davidjo/snd_hda_macbookpro.git
cd snd_hda_macbookpro
sudo ./install.cirrus.driver.sh        # builds + registers with DKMS
reboot
```

---

## TouchBar
1. Become the superuser

```bash
sudo su
```
---

2. Ubuntu uses initramfs so add our new modules to the list to be loaded

```bash
cat <<EOF | tee -a /etc/initramfs-tools/modules
# drivers for keyboard+touchpad
applespi
apple_ib_tb
intel_lpss_pci
spi_pxa2xx_platform
EOF
```
---

3. Build and install drivers from the source code.

```bash
apt install dkms
cd /usr/src
git clone https://github.com/almas/macbook12-spi-driver
cd macbook12-spi-driver
git checkout touchbar-driver-hid-driver
ln -s `pwd` /usr/src/applespi-0.1
dkms install applespi/0.1 --force
```
---

4. Test the drivers by loading them and their dependencies

```bash
modprobe intel_lpss_pci spi_pxa2xx_platform applespi apple_ib_tb
# An empty output indicates success
```
---

5. Nudge the USB

```bash
echo '1-3' | sudo tee /sys/bus/usb/drivers/usb/unbind
echo '1-3' | sudo tee /sys/bus/usb/drivers/usb/bind
```

6. You need to run the commannd everytime, so create a "macbook-quirks.service" and have it start up with the computer:
```bash
cat <<EOF | tee /etc/systemd/system/macbook-quirks.service
[Unit]
Description=Re-enable MacBook 14,3 TouchBar
Before=display-manager.service

[Service]
Type=oneshot
ExecStartPre=/bin/sleep 2
ExecStart=/bin/sh -c "echo '1-3' > /sys/bus/usb/drivers/usb/unbind"
ExecStart=/bin/sh -c "echo '1-3' > /sys/bus/usb/drivers/usb/bind"
RemainAfterExit=yes
TimeoutSec=0

[Install]
WantedBy=multi-user.target
EOF
```
---

7. Enable the service

```bash
systemctl enable macbook-quirks.service
sudo reboot
```
---

## Fan Control

1. Pkg Install: `mbpfan` lets the SMC fans ramp up aggressively before the CPU hits throttling temps.

```bash
# 1. packages
sudo apt install lm-sensors git build-essential

# detect all temp sensors (answer YES for everything)
sudo sensors-detect --auto
```
---

2. build & install mbpfan

```bash
cd /tmp
git clone https://github.com/linux-on-mac/mbpfan.git
cd mbpfan
make
sudo make install           # installs binary + default config
```
---

3. enable the service

```bash
sudo cp mbpfan.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now mbpfan
```
---

4. check it’s running

```bash
systemctl status mbpfan --no-pager
watch -n1 sensors           # temps + fan RPM every second
```

5. Tuning (optional): Open `/etc/mbpfan.conf` and adjust:

```bash
low_temp  = 50
high_temp = 70
max_temp  = 87
min_fan_speed = 2000   # RPM
max_fan_speed = 6000
```
---

6. Then restart:

```bash
sudo systemctl restart mbpfan
```
---

## Refernce

[almas/ubuntu-on-mbp-a-1707.md](https://gist.github.com/almas/5f75adb61bccf604b6572f763ce63e3e)
