Official installation guide: https://omarchy.org/manual/getting-started/ 

**Your machine (checked from Windows, 2026-09-30)**

| | |
|---|---|
| Model | MSI GF65 Thin 10SER (board MS-16W1), BIOS E16W1IMS.50B (2020) |
| CPU / RAM | Intel Core i5-10300H, 32 GB |
| GPU | Intel UHD (drives the built-in screen) + NVIDIA RTX 2060 (Turing) |
| Screen | 15.6" AUO 1920×1080 @ 120 Hz |
| Drive | LITEON CL1-8D512 512 GB NVMe — the only internal drive, plain NVMe (no Intel RST/RAID to switch off) |
| Wi-Fi / BT / LAN | Intel AX201, Intel Bluetooth, Realtek RTL8168 — all supported by the stock kernel |
| OS | Windows 11 Home (not 10), OEM license embedded in firmware |
| Current state | UEFI boot, **Secure Boot ON**, BitLocker/device encryption OFF |

**Before you touch the laptop**

1. Back up everything from Windows. The full-disk install erases the drive. C: has about 457 GB in use, so you need an external disk or cloud space of that size. Things that are easy to forget:
   - Your VPN setups. Fortinet and OpenVPN Connect are installed, so export the `.ovpn` profiles, certificates and FortiClient gateway settings. On Linux you'll use `openvpn`/NetworkManager and `openfortivpn`.
   - Browser profiles, SSH keys (`%USERPROFILE%\.ssh`), and anything under `C:\Users\groes\sw`.
   - Bluetooth devices (WH-1000XM3, Momentum TW 3, JBL Flip 5, Argon Stream3) have to be paired again. Nothing to back up, just expect it.
2. Your hardware details are in the table above. Your **Windows 11** license is an OEM key stored in the firmware (`OEM_DM` channel), so a clean Windows 11 install reactivates on its own if you ever go back. The disk's MSI recovery partitions (~20 GB) will be wiped too. A factory restore will no longer be possible, so if you care about that, make a recovery USB first. See step 2a.

2a. **(Optional) Make a Windows recovery USB.** This gives you a way back to Windows 11 without needing another PC. You need a **separate** USB stick of at least 32 GB (Microsoft says 16 GB, but a full Windows 11 copy often doesn't fit). Everything on the stick is erased, so don't use the Verbatim.
   1. Plug the spare stick in and unplug any other USB drives, so you can't pick the wrong one.
   2. Press Start, type **Create a recovery drive**, and open it. Approve the admin prompt when it appears.
   3. Leave **"Back up system files to the recovery drive"** ticked. Without it, the stick can only repair Windows, not reinstall it.
   4. Click **Next**. Scanning can take several minutes. Then pick the spare stick and click **Next** → **Create**. Expect 30–90 minutes. Keep the laptop plugged in and don't let it sleep.
   5. When it finishes, it may offer to *"Delete the recovery partition from your PC"*. Skip that; the Omarchy install wipes the disk anyway.
   6. Label the stick and test that it's listed in the `F11` boot menu. You don't need to actually boot from it.

   To use it later, boot it via `F11`, choose **Troubleshoot → Recover from a drive**, and it reinstalls Windows 11. Your firmware license activates Windows automatically. Note what you *don't* get back:
   - It's a clean Windows, not MSI's factory image. MSI's apps and Nahimic won't be there; drivers come through Windows Update or msi.com (search "GF65 Thin 10SER").
   - The factory image in the 20 GB partition is likely the original Windows 10 anyway, and MSI's *Burn Recovery* tool for copying it to USB isn't installed on this laptop. Saving that image isn't worth the effort.

   **Simpler alternative:** skip the recovery drive and, if you ever want Windows back, download Microsoft's **Windows 11 Media Creation Tool** onto any PC and make an install stick (8 GB+). Thanks to the firmware license, the result is the same: a clean, activated Windows 11.

**Make the USB stick**

3. Go to omarchy.org and click the ISO button (Omarchy 4 is just under 6 GB). Flash it to an 8 GB+ stick with balenaEtcher (Rufus works too; pick DD/image mode, not ISO mode).
   - ⚠️ The only USB stick currently plugged in is your **Verbatim STORE N GO (248 GB)**, which has ~47 GB of data on it. Flashing erases the whole stick. Copy that data off first, and **don't use the same stick as your backup destination**. Ideally use a separate small stick for the ISO.

**BIOS**

4. Reboot and hammer `Del` to enter BIOS (`F2` on some models). Your laptop has the simple American Megatrends laptop BIOS, not the desktop MSI Click BIOS, so there is **no `F7` advanced mode**. Everything you need is on the *Security* and *Boot* tabs. (MSI's hidden advanced menu is `Right Ctrl + Right Shift + Left Alt + F2`, but you shouldn't need it.)
5. Disable **Secure Boot** (Windows reports it's currently **on**). Also disable **Fast Boot** and **TPM/PTT** if the options are there, and make sure boot mode is UEFI (you already boot UEFI; this 2020 BIOS may not even offer CSM/Legacy). Omarchy's manual says Secure Boot and/or TPM must be off or the install won't proceed. On some MSI laptops the Secure Boot toggle is greyed out until you set a supervisor password. If that happens, set one, flip the toggle, then clear the password again so you don't forget it.
6. Save and exit (`F10`), then press `F11` at boot for the one-time boot menu and pick the USB (choose the UEFI entry).

**Install**

7. Answer the config questions — keyboard layout, username, password, hostname, timezone — and confirm. Pick the Danish layout here. The password you set is also the LUKS disk-encryption password, and you'll type it with the built-in keyboard at every boot. That works fine; only Bluetooth keyboards can't enter it.
8. Select your internal drive (the 512 GB LITEON; **not** the USB stick). Everything on it gets wiped. The install typically takes 2–10 minutes.
9. Reboot, pull the USB. You'll get a LUKS passphrase prompt, then it auto-logs straight into Hyprland. Connect to Wi-Fi (the AX201 works out of the box) and run `omarchy-update`.

**Things specific to your machine**

*Hybrid graphics (the big one).* The GF65 Thin is an Intel iGPU + NVIDIA dGPU laptop **with no MUX switch**, so there is no "discrete-only" BIOS setting to fall back on. The built-in screen is always driven by the Intel GPU. On most GF65 units the HDMI port is wired to the NVIDIA GPU, so external monitors only work once the NVIDIA driver is loaded. The good news is that the RTX 2060 is Turing, which is supported by the open NVIDIA kernel modules, and Omarchy's installer detects and installs them for you. Hyprland's own docs warn that NVIDIA lacks important multi-GPU features on hybrid laptops, which can leave the setup broken or slow. If things feel choppy or the battery drains fast:
- Make Hyprland render on the Intel GPU by putting it first in `AQ_DRM_DEVICES` (use the `/dev/dri/by-path/…` symlinks, since `card0`/`card1` numbering can swap between boots).
- Install `nvidia-prime` and launch games/GPU apps with `prime-run <app>`.
- Check that the dGPU actually powers down when idle (`cat /sys/bus/pci/devices/0000:01:00.0/power/runtime_status` should say `suspended`). If it never does, battery life will be poor.

*Screen scaling.* Your panel is 1080p, and Omarchy's defaults are tuned for high-DPI screens, so XWayland apps can look double-sized. In `~/.config/hypr/monitors.lua` set `omarchy_gdk_scale = 1` and `omarchy_monitor_scale = 1`. While you're there, confirm it's running at 120 Hz (`hyprctl monitors`).

*MSI extras.* Dragon Center / MSI Center, Cooler Boost, fan curves, battery charge limit and Nahimic audio are all gone. Your keyboard is the GF65's single-colour red backlight, not SteelSeries per-key RGB, so `msi-perkeyrgb` doesn't apply. The `msi-ec` kernel module from the AUR can give you Cooler Boost, fan modes and a battery charge threshold, but only if your EC firmware is on its supported list. Check the README against your EC version before relying on it.

*Battery.* Your battery reports a full-charge capacity of ~39.9 Wh, against a design capacity of roughly 51 Wh for this model, so it's at about 78% health. Combined with the hybrid-GPU caveats, don't expect Windows-level battery life.

**If you'd rather not commit**

Omarchy 4's installer also does a free-space install alongside Windows. To use it, shrink the Windows partition in Disk Management and make sure BitLocker is off first (it currently is). On your machine that's tight: C: has only ~33 GB free, so you'd have to clear space before you could shrink it enough (aim for 100 GB+ for Omarchy). Also note there's a known 4.0.2 bug where the free-space install doesn't register a UEFI boot entry and the laptop boots straight into Windows (omacom/omarchy issue #10598). If that happens, pick Omarchy from the `F11` boot menu and add the entry afterwards.
