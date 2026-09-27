# h3 — Modified build of DPIV00001 (DPI v2)

This file explains every modification made to the original `DPIV00001.elf`, and how to use each feature.

---

## 1. Upload any file type (not just PKG)

**Before:** the interface only accepted `.pkg` files.
**Now:** you can select or drag-and-drop **any file**, any extension (zip, txt, mkv, rar...).

- If the file is a **real PKG package** (detected from its content, or by a `.pkg` extension) → it uploads **and installs automatically**.
- Any other file → it uploads **without installing**, and shows a "File saved on the PS5 (not installed)" notice.

## 2. Name and extension stay exactly as they are

**Before:** any uploaded file had its name automatically changed to end in `.pkg` (e.g. `notes.txt` became `notes.txt.pkg`).
**Now:** the name stays **exactly** as it is. `notes.txt` stays `notes.txt`, `date.zip` stays `date.zip`.

## 3. No size limit

There used to be UI text mentioning a 50GB limit (it was only text — there was never an actual limit in the code). It's been removed, and the interface now shows "No size limit".

## 4. ⭐ Chunked upload (splitting)

This is the core feature that makes uploads faster and more reliable, especially for large files:

- The **first 1MB** of the file is sent on its own first (the server needs this to start the upload session and verify the file's checksum).
- **The rest of the file is split into 16 roughly equal parts.**
- **8 parts upload at the same time** (in parallel), and as soon as one finishes, the next one starts immediately.
- **Each part's size is capped between:**
  - **1MB to 256MB** when uploading directly from your device.
  - **1MB to 64MB** when uploading via a URL (since each part is first downloaded into the browser's memory).
  - In practice: a very small file → fewer than 16 parts. A very large file (e.g. over 4GB) → automatically more than 16 parts (each part still capped at 256MB/64MB).
- **If a part fails** (a temporary network drop), **only that part is resent**, without stopping the rest.
- **Reassembly is fully automatic**: each part is written to its correct position within the same file on the PS5 — you don't need to do anything manually once the upload completes.

**Benefit:** better use of your network speed (instead of one slow sequential upload), and better resilience against temporary connection drops.

## 5. Upload via URL

Below the drag-and-drop area there's a "Paste package link (URL)" field:

- Paste a direct link to a file, click "Add", and it uploads the same way as any other file (with the chunking described above).
- **Key requirement:** the server hosting the file must allow reading from an external browser page (CORS). Most public hosting sites (archive.org, GitHub Releases, Google Drive...) **do not allow this by default**, and you'll see a "Could not read the link" notice.
- **Included workaround:** `cors_server.py` — a simple Python script you run on any device (computer/laptop) on the same network. It hosts the file locally with correct CORS settings, and you use its local link instead of the external one.
  ```
  python3 cors_server.py
  ```
  It will print a link like `http://192.168.x.x:8000/game.pkg` — paste that into the field.

## 6. Automatic folder creation

**Before:** if `/user/data/h3` (or its parent) didn't exist, the program would fail to start.
**Now:** on every startup, the program automatically creates, with no action needed from you:
- `/user/data/h3` (then `pkgs` inside it — where uploaded files are stored)
- `/data/h3` and `/data/h3/plugins` (where the config file lives)
- and if the `DPIV00001.ini` config file doesn't exist, it writes a default one itself.

## 7. Name and logo

- The name changed from **OnionHEN** to **h3** in the page title and logo.
- The logo is now a custom image instead of the original onion logo.

## 8. Ports

Latest configuration:
- **API port:** `8098`
- **WebUI port:** `8099`

After installing, open the interface at: `http://<PS5's IP address>:8099`

> **Important note:** the ports were changed several times during development due to conflicts with older instances already running on the same device. Make sure the file actually installed is the latest build (exactly 296,560 bytes), and fully power-cycle the console after every replacement.

## 9. File paths on the PS5

| Purpose | Path |
|---|---|
| Uploaded files | `/user/data/h3/pkgs/` |
| Config file | `/data/h3/plugins/DPIV00001.ini` |
| Log file | `/data/h3/DPIV00001.log` |
| Server log | `/data/h3/DPIV00001-server.log` |
| The program file itself | `/data/OnionHEN/plugins/DPIV00001.elf` (this path is unchanged) |

## 10. Default config file contents

```
enabled=true
api_port=8098
webui_port=8099
```

---

## General disclaimer

All modifications were made by patching the compiled binary directly (without the original source code), and most were tested either in an off-device simulation or on a real PS5 during this work. Even so, it's always recommended to:
- Keep a copy of the original file before replacing it.
- Test with a small file first before large or important ones.
- Fully power-cycle the console (not Rest Mode) after every file replacement.
