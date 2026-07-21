# 🎬 Jellyfin GPU Acceleration

> Jellyfin deployment using an NVIDIA RTX 3060 passed through to a Proxmox LXC for hardware decoding and encoding.

---

# 📖 Overview

Jellyfin runs in a dedicated Linux container on Proxmox VE.

The NVIDIA RTX 3060 is passed from the Proxmox host into the Jellyfin LXC to provide hardware-accelerated media transcoding.

Hardware acceleration reduces CPU usage and improves playback performance when a client cannot directly play the original media format.

---

# 🏗️ Architecture

```text
Media Client
     │
     ▼
  Jellyfin
     │
     ▼
Jellyfin FFmpeg
     │
     ▼
NVIDIA RTX 3060
     │
     ├── NVDEC hardware decoding
     └── NVENC hardware encoding
```

The NVIDIA kernel driver runs on the Proxmox host.

The Jellyfin LXC uses the host GPU devices and compatible NVIDIA userspace libraries.

---

# 🖥️ Hardware

| Component        | Model                   |
| ---------------- | ----------------------- |
| GPU              | NVIDIA GeForce RTX 3060 |
| VRAM             | 12 GB                   |
| GPU Architecture | Ampere                  |
| Hardware Decoder | NVDEC                   |
| Hardware Encoder | NVENC                   |

The RTX 3060 supports:

* H.264 decoding and encoding
* HEVC decoding and encoding
* HEVC 10-bit decoding
* VP8 decoding
* VP9 decoding
* VP9 10-bit decoding
* AV1 decoding

The RTX 3060 does not support AV1 hardware encoding.

---

# 🔍 Checking the GPU on Proxmox

Check PCI devices:

```bash
lspci | grep -i nvidia
```

Example result:

```text
NVIDIA Corporation GA106 [GeForce RTX 3060]
```

Check the NVIDIA driver:

```bash
nvidia-smi
```

The output should display:

* GPU model
* Driver version
* VRAM usage
* GPU temperature
* Running GPU processes

---

# 📦 NVIDIA Driver

The NVIDIA driver must be installed on the Proxmox host.

Check the loaded modules:

```bash
lsmod | grep nvidia
```

Expected modules include:

```text
nvidia
nvidia_modeset
nvidia_drm
nvidia_uvm
```

Check the driver version:

```bash
cat /proc/driver/nvidia/version
```

---

# 🔌 NVIDIA Device Files

Check available device nodes on the Proxmox host:

```bash
ls -l /dev/nvidia*
```

Expected devices include:

```text
/dev/nvidia0
/dev/nvidiactl
/dev/nvidia-modeset
/dev/nvidia-uvm
/dev/nvidia-uvm-tools
```

These device files allow applications to communicate with the GPU.

---

# ⚠️ Missing NVIDIA UVM Device

A common problem is that the `nvidia_uvm` kernel module is not loaded.

Check:

```bash
lsmod | grep nvidia_uvm
```

If it is missing:

```bash
modprobe nvidia_uvm
```

Check the devices again:

```bash
ls -l /dev/nvidia*
```

---

# 🛠️ Creating NVIDIA UVM Devices

The module may be loaded while `/dev/nvidia-uvm` is still missing.

Run:

```bash
nvidia-modprobe -u -c=0
```

Check again:

```bash
ls -l /dev/nvidia*
```

Expected additional devices:

```text
/dev/nvidia-uvm
/dev/nvidia-uvm-tools
```

---

# 🔄 Loading NVIDIA UVM at Boot

Add the module on the Proxmox host:

```bash
echo nvidia_uvm >> /etc/modules
```

Verify:

```bash
cat /etc/modules
```

The module must be configured on the Proxmox host, not inside the Jellyfin LXC.

LXC containers share the Proxmox kernel and cannot load kernel modules independently.

---

# 🔢 Device Major Numbers

Check the NVIDIA device major and minor numbers:

```bash
ls -ln /dev/nvidia*
```

Example:

```text
195, 0   /dev/nvidia0
195, 255 /dev/nvidiactl
195, 254 /dev/nvidia-modeset
509, 0   /dev/nvidia-uvm
509, 1   /dev/nvidia-uvm-tools
```

The major number for UVM devices may change after a driver update.

Always verify the current values before editing the LXC configuration.

---

# 🔗 Passing the GPU into the LXC

Stop the Jellyfin LXC:

```bash
pct stop CTID
```

Edit its configuration:

```bash
nano /etc/pve/lxc/CTID.conf
```

Example device permissions:

```ini
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 509:* rwm
```

Example device mounts:

```ini
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
```

Replace the UVM major number if it differs on the host.

Start the container:

```bash
pct start CTID
```

---

# 🔍 Checking Devices Inside the LXC

Enter the Jellyfin container:

```bash
pct enter CTID
```

Check the device nodes:

```bash
ls -l /dev/nvidia*
```

Expected:

```text
/dev/nvidia0
/dev/nvidiactl
/dev/nvidia-modeset
/dev/nvidia-uvm
```

If a device appears as a normal empty file instead of a character device, the mount or device node is incorrect.

A correct device begins with:

```text
c
```

Example:

```text
crw-rw-rw-
```

---

# 📚 NVIDIA Userspace Libraries

The Proxmox host provides the NVIDIA kernel driver.

The Jellyfin LXC requires compatible NVIDIA userspace libraries.

Required packages may include:

```text
libnvidia-compute
libnvidia-decode
libnvidia-encode
libnvidia-extra
nvidia-modprobe
```

Install compatible versions:

```bash
apt update
apt install -y \
  libnvidia-compute \
  libnvidia-decode \
  libnvidia-encode \
  libnvidia-extra \
  nvidia-modprobe
```

Package names vary depending on the Linux distribution and NVIDIA repository.

---

# ⚠️ Driver and Library Version Mismatch

A common error occurs when:

```text
Proxmox NVIDIA driver: version 610
LXC NVIDIA libraries: version 595
```

Possible symptoms:

```text
Failed to initialize NVML
Driver/library version mismatch
```

Check inside the LXC:

```bash
nvidia-smi
```

Check installed libraries:

```bash
dpkg -l | grep nvidia
```

Check available versions:

```bash
apt-cache policy libnvidia-compute
```

The userspace libraries must be compatible with the host driver.

---

# 🧹 Removing Incorrect NVIDIA Packages

Example:

```bash
apt remove -y \
  nvidia-utils-595 \
  libnvidia-compute-595 \
  nvidia-kernel-common-595
```

Then:

```bash
apt autoremove -y
```

Install the matching libraries afterward.

Do not install an NVIDIA kernel driver inside the LXC.

The kernel driver belongs on the Proxmox host.

---

# ✅ Testing NVIDIA Inside the LXC

Run:

```bash
nvidia-smi
```

Expected information:

```text
NVIDIA GeForce RTX 3060
Driver Version
CUDA Version
GPU Memory
```

If `nvidia-smi` works inside the LXC, the GPU devices and NVIDIA libraries are available.

---

# 🎞️ Jellyfin FFmpeg

Jellyfin uses its bundled FFmpeg binary.

Typical path:

```text
/usr/lib/jellyfin-ffmpeg/ffmpeg
```

Check supported hardware acceleration:

```bash
/usr/lib/jellyfin-ffmpeg/ffmpeg -hwaccels
```

Expected output should include:

```text
cuda
```

Check NVIDIA encoders:

```bash
/usr/lib/jellyfin-ffmpeg/ffmpeg -encoders | grep nvenc
```

Expected encoders:

```text
h264_nvenc
hevc_nvenc
```

Check NVIDIA decoders:

```bash
/usr/lib/jellyfin-ffmpeg/ffmpeg -decoders | grep -E 'cuvid|nvdec'
```

---

# ⚙️ Jellyfin Hardware Acceleration Settings

Open:

```text
Jellyfin Dashboard
→ Playback
→ Transcoding
```

Set:

```text
Hardware acceleration:
NVIDIA NVENC
```

Enable hardware decoding for:

* H.264
* HEVC
* MPEG-2
* MPEG-4
* VC-1
* VP8
* VP9
* AV1
* HEVC 10-bit
* VP9 10-bit

HEVC RExt options can remain disabled unless the media library contains those formats.

---

# 🎥 Hardware Encoding

Enable:

```text
Enable hardware encoding
```

Enable:

```text
Allow encoding in HEVC format
```

Leave AV1 encoding disabled.

The RTX 3060 can decode AV1 but cannot encode AV1.

H.264 hardware encoding remains the most compatible option for clients.

---

# 🌈 HDR Tone Mapping

Tone mapping converts HDR content to SDR for displays that do not support HDR.

Enable:

```text
Enable tone mapping
```

Recommended algorithm:

```text
BT.2390
```

Suggested starting settings:

```text
Tone mapping algorithm: BT.2390
Tone mapping mode: Auto
Output color range: Auto
Peak: Default
```

Tone mapping requires compatible GPU processing libraries.

---

# 📝 Subtitle Handling

Subtitle formats can affect transcoding.

Text-based subtitles such as SRT may be supported directly by many clients.

ASS and PGS subtitles may require burning into the video.

Enable:

```text
Allow subtitle extraction on the fly
```

This can reduce unnecessary video transcoding when subtitles can be extracted and sent separately.

Anime content frequently uses ASS subtitles, which may require transcoding depending on the client.

---

# ⚡ Enhanced NVDEC

Jellyfin provides an Enhanced NVDEC option.

Enable it when hardware decoding is stable:

```text
Enable enhanced NVDEC decoder
```

If decoding errors occur, disable it and test the standard CUVID implementation.

---

# 📁 Transcode Directory

Jellyfin temporarily stores transcoded media segments.

The directory should be on fast storage.

Example:

```text
/var/cache/jellyfin/transcodes
```

A dedicated directory can be configured:

```text
/transcodes
```

Possible storage options:

* NVMe SSD
* LXC local disk
* Temporary RAM storage for limited workloads

Do not place active transcode files on slow HDD storage when avoidable.

---

# 🔍 Testing Hardware Transcoding

Play a file that requires transcoding.

Open:

```text
Jellyfin Dashboard
→ Active Devices
```

The playback status should show:

```text
Transcoding
```

Hardware transcoding may be shown as:

```text
Transcoding (hw)
```

---

# 📊 Monitoring GPU Usage

While a transcode is running:

```bash
nvidia-smi
```

Expected process:

```text
jellyfin-ffmpeg
```

The output should show:

* GPU memory usage
* Decoder activity
* Encoder activity
* Jellyfin FFmpeg process

For continuous monitoring:

```bash
watch -n 1 nvidia-smi
```

---

# 🧪 Forcing a Transcode

A transcode can be triggered by:

* Reducing playback quality
* Using an unsupported codec
* Enabling incompatible subtitles
* Playing HDR content on an SDR client
* Using a client that does not support the source audio format

Example:

```text
Source: 4K HEVC
Playback setting: 1080p
```

This should trigger video transcoding.

---

# 🧾 Checking Jellyfin Logs

Check the Jellyfin service:

```bash
systemctl status jellyfin --no-pager
```

View recent logs:

```bash
journalctl -u jellyfin -n 100 --no-pager
```

Follow logs:

```bash
journalctl -u jellyfin -f
```

Jellyfin FFmpeg logs are normally available through:

```text
Dashboard
→ Logs
```

Look for files beginning with:

```text
FFmpeg.Transcode
```

---

# ⚠️ FFmpeg Exit Code 187

An error similar to:

```text
FFmpeg exited with code 187
```

may be caused by:

* Missing `/dev/nvidia-uvm`
* Incorrect LXC device passthrough
* NVIDIA library mismatch
* Missing NVENC libraries
* Incorrect Jellyfin acceleration settings
* Permission problems

Check:

```bash
ls -l /dev/nvidia*
nvidia-smi
/usr/lib/jellyfin-ffmpeg/ffmpeg -hwaccels
```

---

# ⚠️ GPU Devices Missing After Reboot

If `/dev/nvidia-uvm` disappears after reboot:

```bash
modprobe nvidia_uvm
nvidia-modprobe -u -c=0
```

Confirm that `nvidia_uvm` is present in:

```text
/etc/modules
```

Check that the Proxmox LXC configuration still contains the device mounts.

---

# 🔐 Device Permissions

Jellyfin must have permission to use the GPU devices.

Check:

```bash
ls -l /dev/nvidia*
```

Check the Jellyfin user:

```bash
id jellyfin
```

The Jellyfin user may need access to the `video` and `render` groups.

Example:

```bash
usermod -aG video,render jellyfin
```

Restart Jellyfin:

```bash
systemctl restart jellyfin
```

Group requirements vary depending on the device ownership and installation method.

---

# 🔄 Restarting the Jellyfin LXC

Proxmox may not support a `pct restart` command.

Use:

```bash
pct reboot CTID
```

or:

```bash
pct stop CTID
pct start CTID
```

Check status:

```bash
pct status CTID
```

---

# 📦 Jellyfin Service Management

Restart Jellyfin:

```bash
systemctl restart jellyfin
```

Enable it at boot:

```bash
systemctl enable jellyfin
```

Check listening ports:

```bash
ss -tulpn | grep 8096
```

Default Jellyfin HTTP port:

```text
8096
```

---

# 🌐 Access Through Tailscale

Find the LXC Tailscale address:

```bash
tailscale ip -4
```

Access Jellyfin:

```text
http://TAILSCALE_IP:8096
```

With MagicDNS:

```text
http://jellyfin:8096
```

Do not expose the Jellyfin administration interface publicly without appropriate authentication and reverse-proxy security.

---

# 💾 Media Storage

Jellyfin accesses media through a Proxmox bind mount.

Example:

```ini
mp0: /pool/media,mp=/data/media
```

Example libraries:

```text
Movies:
/data/media/movies

TV Shows:
/data/media/shows

Anime:
/data/media/anime

Music:
/data/media/music
```

Jellyfin normally requires read access to media.

Write access may be required for:

* NFO metadata
* Artwork
* Subtitle downloads
* File deletion through Jellyfin

Grant only the permissions required.

---

# 💽 Backup Requirements

Back up:

```text
Jellyfin configuration
Jellyfin database
User accounts
Library configuration
Plugins
Metadata
Custom artwork
```

Typical configuration location:

```text
/var/lib/jellyfin
```

The exact path depends on the installation method.

Media files should have their own backup strategy.

---

# 🔄 Updating NVIDIA Drivers

Before updating the NVIDIA driver:

1. Record the current driver version.
2. Back up the Jellyfin LXC configuration.
3. Check the current device major numbers.
4. Stop active transcodes.
5. Update the Proxmox host driver.
6. Reboot the host if required.
7. Recreate UVM device nodes if necessary.
8. Update compatible LXC userspace libraries.
9. Test `nvidia-smi`.
10. Test Jellyfin transcoding.

Driver updates may change the UVM major number.

---

# 🛠️ Useful Commands

```bash
# Check GPU
nvidia-smi

# Check NVIDIA modules
lsmod | grep nvidia

# Load UVM
modprobe nvidia_uvm

# Create UVM devices
nvidia-modprobe -u -c=0

# Check GPU devices
ls -l /dev/nvidia*

# Check device numbers
ls -ln /dev/nvidia*

# Check LXC configuration
pct config CTID

# Enter LXC
pct enter CTID

# Check Jellyfin service
systemctl status jellyfin --no-pager

# Restart Jellyfin
systemctl restart jellyfin

# Check FFmpeg hardware acceleration
/usr/lib/jellyfin-ffmpeg/ffmpeg -hwaccels

# Check NVIDIA encoders
/usr/lib/jellyfin-ffmpeg/ffmpeg -encoders | grep nvenc

# Monitor GPU
watch -n 1 nvidia-smi

# Check Jellyfin logs
journalctl -u jellyfin -n 100 --no-pager
```

---

# ⚠️ Important Notes

* Install the NVIDIA kernel driver only on the Proxmox host.
* Install compatible userspace libraries inside the LXC.
* Verify device major numbers after driver updates.
* Load `nvidia_uvm` on the Proxmox host.
* Do not add kernel modules inside the LXC.
* Keep AV1 encoding disabled on the RTX 3060.
* Confirm GPU usage with `nvidia-smi`.
* Use fast storage for transcoding.
* Back up Jellyfin configuration before upgrades.
* Test hardware acceleration after every NVIDIA driver update.

---

# ✅ Current Status

| Task                        | Status          |
| --------------------------- | --------------- |
| NVIDIA driver on Proxmox    | ✅ Complete      |
| GPU detected                | ✅ Complete      |
| NVIDIA UVM module           | ✅ Complete      |
| UVM device nodes            | ✅ Complete      |
| GPU passed into LXC         | ✅ Complete      |
| NVIDIA libraries inside LXC | ✅ Complete      |
| `nvidia-smi` inside LXC     | ✅ Working       |
| Jellyfin FFmpeg CUDA        | ✅ Working       |
| NVENC encoding              | ✅ Working       |
| NVDEC decoding              | ✅ Working       |
| HEVC encoding               | ✅ Enabled       |
| AV1 decoding                | ✅ Supported     |
| AV1 encoding                | ❌ Not Supported |
| HDR tone mapping            | ✅ Configured    |
| Automated GPU monitoring    | ⏳ Planned       |

---

# 🚀 Future Improvements

* Add GPU metrics to Prometheus
* Create Grafana GPU dashboards
* Add transcode alerts
* Monitor GPU temperature
* Document tested client compatibility
* Optimize subtitle handling
* Add transcode-directory monitoring
* Test multiple simultaneous streams
* Automate driver compatibility checks
* Document recovery after driver upgrades
