# What can you do with this?

You can pop your SD card out of your camera, plug it into a Raspberry Pi,
and have all your photos synced to a remote server automatically, with notifications
via webhooks (e.g. Home Assistant).

All images on the SD card are copied to a single directory on the remote server.

e.g.
`20251004_161949_DSCF0292.JPG`

# Setup

- (optional step) [Setup wifi configuration over bluetooth](https://github.com/nksan/Rpi-SetWiFi-viaBluetooth).
  - Run install script.
  - Install ios app.

- Generate ssh keys `ssh-keygen -t rsa`.
- Add `~/.ssh/id_rsa.pub` to your server.
- Install [pi-usb-automount](https://github.com/fasteddy516/pi-usb-automount).
  This will auto mount usb drives in `/media`.
- Add udev usb automount rule by creating a new file `/etc/udev/rules.d/99-usb-automount.rules`.
  This will execute a script when a usb drive is plugged in. Add the following to this newly created file.

```
ACTION=="add", SUBSYSTEM=="block", ENV{ID_FS_TYPE}!="", RUN+="/usr/bin/systemd-run --no-block /home/pi/usb-handler.sh %k"
```

- Update rules.

```sh
sudo udevadm control --reload-rules
sudo udevadm trigger
```

- Create `/home/pi/usb-handler.sh`.
- `sudo chmod +x /home/pi/usb-handler.sh`
- Fill in the following.
  - Server info/destination directory.
  - (optional) Webhook URLs.

```bash
#!/bin/bash

sleep 2


# OPTIONAL: Uncomment and fill in webhook for notifications. e.g. Home Assistant webhook
# curl <web hook url - upload started...>

find /media -type f \( \
  -iname '*.jpg' -o -iname '*.jpeg' -o -iname '*.png' -o -iname '*.heic' -o \
  -iname '*.raw' -o -iname '*.cr2' -o -iname '*.nef' -o -iname '*.arw' \
  -iname '*.mp4' -o -iname '*.mov' -o -iname '*.avi' -o -iname '*.mkv' \
\) -exec bash -c '
  count=0
  for f; do
    # Try to extract the capture date from metadata
    date=$(exiftool -DateTimeOriginal -MediaCreateDate -MediaModifyDate \
      -d "%Y%m%d_%H%M%S" -s3 "$f" 2>/dev/null | head -n1)

    # Fallback: use file modification time if metadata not found
    if [ -z "$date" ]; then
      date=$(date -r "$f" +%Y%m%d_%H%M%S)
    fi

    base=$(basename "$f")
    name="${date}_${base}"

    # Transfer file (flattened)
    if rsync -av --ignore-existing "$f" "<user>@<server>:/path/to/photos/$name" | grep -q "$base"; then
      ((count++))
    fi
  done

  # OPTIONAL: Uncomment and fill in webhook for notifications. e.g. Home Assistant webhook
  # curl -X POST -H "Content-Type: application/json" -d "{\"count\": $count}" <web hook POST url>
' bash {} +
```
