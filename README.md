# Configurating a Raspberry Pi Zero W to auto Rsync (ssh) photos from an SD card to a remote server.

- [Setup wifi configuration over bluetooth](https://github.com/nksan/Rpi-SetWiFi-viaBluetooth).
  - Run install script.
  - Install ios app.

- Generate ssh keys `ssh-keygen -t rsa`.
- Add `~/.ssh/id_rsa.pub` to server.
- Install [pi-usb-automount](https://github.com/fasteddy516/pi-usb-automount).
  This will auto mount usb drives in `/media`.
- Add udev usb automount rule. This will execute a script when a usb drive is plugged in.

**/etc/udev/rules.d/99-usb-automount.rules**

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

**/home/pi/usb-handler.sh**

```bash
#!/bin/bash

sleep 2
curl <web hook url - upload started...>

rsync -avz --progress \
  --include='*/' \
  --include='*.jpg' \
  --include='*.JPG' \
  --include='*.jpeg' \
  --include='*.JPEG' \
  --include='*.cr2' \
  --include='*.CR2' \
  --include='*.nef' \
  --include='*.NEF' \
  --include='*.arw' \
  --include='*.ARW' \
  --exclude='*' \
  /media user@server:/path/to/dest/

curl <web hook url - upload finished...>
```
