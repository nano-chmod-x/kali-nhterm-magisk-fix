patch this 
# kali-nhterm-magisk-fix
# "/data/data/com.termux/files/usr"
# "/data/local/tmp/Boot-Animation-install.
zip"

**Fix NH-Terminal (NetHunter Terminal) to reliably launch the Kali chroot on modern Magisk using the Magisk CLI** — no `su`, no Termux paths, no `-c: not found`.

```

git clone https://github.com/Dylanmurzello/kali-nhterm-magisk-fix
  cd kali-nhterm-magisk-fix
  chmod +x scripts/fix-nh.sh
  su
  bash


bash
# from Termux or
adb shell scripts/fix-nh.sh
bash


## What this fixes
- NH-Terminal shows `line 35: -c: command not found (code 127)` on launch.
- Editing random 'kali' scripts doesn't help because NH-Terminal uses its own launcher.
- Forwarding to Termux's boot-kali fails (`inaccessible or not found`) due to app sandboxing.
- `su` inside NH-Terminal is missing — modern Magisk exposes `magisk` CLI instead.

## How it works
We replace the app's launcher at:`/data/data/com.offsec.nhterm/files/usr/bin/kali`with a tiny shim that runs:
```
MAGISK su --mount-master -c "/system/bin/chroot /data/local/nhsystem/kalifs /bin/bash -l"
```
Where `MAGISK` is auto-detected (`/debug_ramdisk/magisk`, `/sbin/magisk`, or `$(magisk --path)/magisk`).

## Install

### Option A — one-shot patch
```
bash scripts/fix-nh.sh
```

### Option B — manual
```
cp scripts/kali /data/data/com.offsec.nhterm/files/usr/bin/kali
chmod 755 /data/data/com.offsec.nhterm/files/usr/bin/kali
sed -i 's/\r$//' /data/data/com.offsec.nhterm/files/usr/bin/kali
```

Open **Kali** in NH-Terminal → you should be at `root@kali`.

## Verify Magisk CLI
```
magisk --path            # e.g. /debug_ramdisk
$(magisk --path)/magisk su --mount-master -c id
```

If that fails, enable **Allow external apps** in Magisk settings and ensure NH-Terminal/NetHunter are not on DenyList.

## Notes
- NH-Terminal may restore its original launcher only on a cold start. If so, run `scripts/fix-nh.sh` again.
- To hide the minimal-login banner inside Kali:
```
touch ~/.hushlogin
```

## Troubleshooting
- **code 126 "No such file"** (file exists): wrong shebang → use `#!/system/bin/sh`.
- **`su not found`**: never use plain `su`; use the Magisk CLI as shown.
- **`Bad system call (159)`**: missing `--mount-master` or broken chroot. Reinstall chroot via NetHunter app.
- **CRLF^M**: `sed -i 's/\r$//' /path/to/script`.

## Credits
- Original investigation & fixes by Dylan & thanks to Aravind Potluri (CIPH3R) for his boot-kali script
- License: MIT.
