```
#!/data/data/com.termux/files/usr/bin/bash
# Patch NH-Terminal's launcher to a stable Magisk+chroot shim
# Run from Termux or adb shell after a cold start if needed.

TARGET="/data/data/com.offsec.nhterm/files/usr/bin/kali"

cat > "$TARGET" <<'EOF'
#!/system/bin/sh
if [ -x /debug_ramdisk/magisk ]; then
  MAGISK=/debug_ramdisk/magisk
elif [ -x /sbin/magisk ]; then
  MAGISK=/sbin/magisk
else
  MAGISK="$(magisk --path 2>/dev/null)/magisk"
fi
[ -x "$MAGISK" ] || { echo "[nh] magisk CLI not found"; exit 127; }
KALIFS="/data/local/nhsystem/kalifs"
[ -d "$KALIFS" ] || { echo "[nh] missing $KALIFS"; exit 1; }
exec "$MAGISK" su --mount-master -c "/system/bin/chroot $KALIFS /bin/bash -l"
EOF

chmod 755 "$TARGET"
# normalize CRLF if any
sed -i 's/\r$//' "$TARGET" 2>/dev/null || true
echo "[*] Patched $TARG
ET"

git clone https://github.com/Dylanmurzello/kali-nhterm-magisk-fix
  cd kali-nhterm-magisk-fix
  chmod +x scripts/fix-nh.sh
  su
  bash
adb shell scripts/fix-nh.sh
bash


## What this fixes
- NH-Terminal shows `line 35: -c: command not found (code 127)` on launch.
- Editing random 'kali' scripts doesn't help because NH-Terminal uses its own launcher.
- Forwarding to Termux's boot-kali fails (`inaccessible or not found`) due to app sandboxing.
- `su` inside NH-Terminal is missing — modern Magisk exposes `magisk` CLI instead.

## How it works
We replace the app's launcher at:`/data/data/com.offsec.nhterm/files/usr/bin/kali`with a tiny shim that runs:

MAGISK su --mount-master -c "/system/bin/chroot /data/local/nhsystem/kalifs /bin/bash -l"

Where `MAGISK` is auto-detected (`/debug_ramdisk/magisk`, `/sbin/magisk`, or `$(magisk --path)/magisk`).

bash scripts/fix-nh.sh
cp scripts/kali /data/data/com.offsec.nhterm/files/usr/bin/kali
chmod 755 /data/data/com.offsec.nhterm/files/usr/bin/kali
sed -i 's/\r$//' /data/data/com.offsec.nhterm/files/usr/bin/kali


Open **Kali** in NH-Terminal → you should be at `root@kali`.

magisk --path            # e.g. /debug_ramdisk
$(magisk --path)/magisk su --mount-master -c id
```
