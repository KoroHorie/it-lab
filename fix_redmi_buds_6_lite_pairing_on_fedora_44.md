# Fix Redmi Buds 6 Lite Pairing on Fedora 44

## Problem

When trying to pair **Redmi Buds 6 Lite** on Fedora 44:

- GUI shows a random 5-digit code
- Pairing fails
- `bluetoothctl` shows errors like:

```
Authentication Failed
br-connection-key-missing
```

## Cause

- Fedora stores a **corrupted or incomplete Bluetooth pairing key**
- The earbuds and system no longer agree on authentication
- Fedora may also detect the **LE (Low Energy)** device instead of the correct audio device

---

## Solution

### 1. Pair with the device using bluetoothctl

```bash
bluetoothctl
```

Then:

```text
power on
agent NoInputNoOutput
default-agent
scan on
```

Wait for device:

MAC Address is <mac-address>

Then run:

```text
pair <mac-address>
trust <mac-address>
connect <mac-address>
exit
```

---

## Optional Fix (If Pairing Still Fails)
```bash
sudo btmgmt io-cap 0x03
```

Then retry pairing.

---

## Notes

- Ensure earbuds are in **pairing mode** (LED blinking fast)
- Always pair the **non-LE device** for audio
- Once paired, Fedora should auto-connect in future

---

## Expected Result

- Successful pairing
- No more authentication errors
- Earbuds appear in:

```
Settings → Sound → Output
```
