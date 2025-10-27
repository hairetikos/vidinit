# vidinit

Play a full definition video (with sound) during Linux bootup.

## Overview

vidinit allows you to display a video with audio during the Linux boot process.  This is achieved using mpv and systemd to play the video early in the boot sequence.

this is the initial conception, i will explore a solution to have it spawn the video possibly at ore-initramfs stage, or immediately upon initramfs

## Installation

1. **Install mpv** (if not already installed):
   ```bash
   sudo apt install mpv  # Debian/Ubuntu
   sudo dnf install mpv  # Fedora
   sudo pacman -S mpv    # Arch Linux
   ```

2. **Place your video file**:
   ```bash
   sudo cp your-video.mp4 /boot/video.mp4
   ```
   
   **Note**: You can use a different location, but make sure to update the path in the systemd service file accordingly.

3. **Create the systemd service file**:
   ```bash
   cp -av vidinit.service /etc/systemd/system/vidinit.service

4. **Enable and test the service**:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable vidinit.service
   sudo reboot
   ```
   for quick video testing on the TTY console (eg:`(ctl + alt + F1`), copy the one-liner and see which --vo and --ao works best for your system:
   
   `mpv --no-config ---vo=gpu-next --gpu-context=drm --really-quiet --no-osc --no-osd-bar --no-input-default-bindings --no-terminal --loop-file=no /boot/video.mp4`
   
   also consider different video formats (.mp4 H264 video AAC audio should work well)

## Configuration Options

### Video Output (--vo) Options

The default configuration uses `---vo=gpu-next --gpu-context=drm` which works on most modern systems. If you experience issues, try these alternatives:

- `--vo=drm` - Direct Rendering Manager (modern systems)
- `--vo=fbdev2` - Framebuffer device (better for older systems)
- `--vo=fbdev` - Legacy framebuffer
- `--vo=caca` - ASCII art output (text-based, fallback option)
- `--vo=tct` - True color terminal output
- `--vo=gpu` - GPU-accelerated (may require X/Wayland)

**Example for older systems**:
```ini
ExecStart=/usr/bin/mpv --really-quiet --no-terminal --vo=fbdev2 --ao=alsa /boot/video.mp4
```

### Audio Output (--ao) Options

- `--ao=alsa` - ALSA audio (default, most compatible)
- `--ao=pulse` - PulseAudio
- `--ao=pipewire` - PipeWire

### Additional Useful Options

- `--loop` - Loop the video until boot completes
- `--fs` - Force fullscreen
- `--no-audio` - Disable audio playback
- `--volume=50` - Set volume level (0-100)
- `--length=10` - Play only first 10 seconds
- `--speed=1.5` - Adjust playback speed

the default behaviour will also cut the video short once the system has booted
to make it wait, change `Type=forking` to `Type=oneshot`

**Example with loop**:
```ini
ExecStart=/usr/bin/mpv --really-quiet --no-terminal --vo=drm --ao=alsa --loop /boot/video.mp4
```

## Troubleshooting

### Video doesn't play
- Check that mpv is installed: `which mpv`
- Verify video file exists: `ls -l /boot/video.mp4`
- Check service status: `sudo systemctl status vidinit.service`
- View logs: `sudo journalctl -u vidinit.service`

### Black screen or corruption
- Try different `--vo` options (fbdev2, fbdev, caca)
- Ensure your video format is supported (MP4 with H.264 is recommended)

### No audio
- Check audio device: `aplay -l`
- Try `--ao=pulse` or `--ao=pipewire` instead of alsa
- Verify volume isn't muted

### Video path
If you prefer a different location for your video, update the path in the service file:
```ini
ExecStart=/usr/bin/mpv --really-quiet --no-terminal --vo=drm --ao=alsa /path/to/your/video.mp4
```

## Uninstallation

```bash
sudo systemctl disable vidinit.service
sudo rm /etc/systemd/system/vidinit.service
sudo systemctl daemon-reload
sudo rm /boot/video.mp4  # if you want to remove the video
```

## License

MIT License - Feel free to use and modify as needed.

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.
