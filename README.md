# The Ghost's Workstation - CTF Challenge

## Overview

**Difficulty:** Hard  
**Theme:** Digital Forensics & Whistleblower Investigation  
**Domains:** Digital Forensics, Steganography, OSINT  

This CTF challenge simulates a real-world digital forensics investigation where participants must recover deleted evidence from a whistleblower's workstation at Nexora Dynamics, a multinational defense contractor accused of illegal arms deals.

## Challenge Summary

Participants receive a single 10 MB FAT16 disk image seized from a missing systems administrator's workstation. They must recover a deleted ZIP file using forensic tools, extract a JPG image, analyze its metadata to find a Base64-encoded password, decode it, and then use steganography to extract the final flag hidden inside the image. The challenge chains five techniques across multiple domains.

### Scenario

Nexora Dynamics, a multinational defense contractor, has been rocked by allegations of illegal arms deals. A senior systems administrator disappeared after copying classified documents to an external drive. The company's internal security team seized his workstation but found nothing — the files had been deleted. A disk image named `disk_image_IE3132.dd` was made before the machine was wiped.

**Your mission:** As a freelance forensic investigator hired by Nexora's legal team, recover the deleted evidence. Your only lead is the disk image taken from the workstation. The evidence was deleted, but the ghost left traces. Recover the deleted files and extract the final flag.

### Starting Point

- **File:** `disk_image_IE3132.dd`
- **Source:** Disk image seized from the whistleblower's workstation

## Learning Objectives

1. Apply disk forensics techniques to recover deleted files from a FAT16 image
2. Use file carving tools (`icat`, `fls`) to extract deleted data
3. Analyze image metadata with `exiftool` to uncover hidden clues
4. Decode Base64-encoded strings
5. Extract hidden data using steganography (`steghide`)
6. Chain multiple forensic techniques in a logical investigation

## Required Tools

| Tool | Purpose |
|------|---------|
| `sleuthkit` | Suite of forensic tools for analyzing disk images and filesystems |
| `fls` | Lists files and directories on a disk image, including deleted ones |
| `icat` | Extracts a file from a disk image by its inode number |
| `exiftool` | Reads and writes metadata (EXIF) in image and media files |
| `base64` | Encodes or decodes data using Base64 encoding |
| `steghide` | Hides or extracts data within image and audio files |
| `unzip` | Extracts files from ZIP archives |

### Installation

```bash
sudo apt install -y sleuthkit exiftool steghide unzip
```

## Solution Path

### Phase 1: Reconnaissance

| Step | Command | Purpose | Finding |
|------|---------|---------|---------|
| 1 | `file disk_image_IE3132.dd` | Identify file type | FAT16 disk image |
| 2 | `strings disk_image_IE3132.dd \| head -30` | View raw text | Decoy filenames, no flag |
| 3 | `fls -r disk_image_IE3132.dd` | List all files | Decoys + `backup_2024.zip` |
| 4 | `fls -r -d disk_image_IE3132.dd` | List deleted files only | `backup_2024.zip` (inode 17) |
| 5 | `strings disk_image_IE3132.dd \| grep -i "PK" \| head -5` | Check for ZIP signature | PK found → ZIP data exists |

### Phase 2: Recovery

| Step | Command | Purpose | Finding |
|------|---------|---------|---------|
| 1 | `INODE=$(fls -r -d disk_image_IE3132.dd \| grep "backup" \| awk '{print $3}' \| tr -d ':')` | Extract inode number | INODE=17 |
| 2 | `echo "Inode: $INODE"` | Confirm inode | Inode: 17 |
| 3 | `icat disk_image_IE3132.dd $INODE > recovered.zip` | Recover deleted file by inode | `recovered.zip` created |
| 4 | `file recovered.zip` | Verify recovered file type | Zip archive data |
| 5 | `unzip -o recovered.zip` | Extract archive | `company_photo.jpg` |

### Phase 3: Metadata Analysis

| Step | Command | Purpose | Finding |
|------|---------|---------|---------|
| 1 | `exiftool company_photo.jpg` | Read image metadata | ImageDescription: `djRuZ3U0cmQyMDI0` |
| 2 | `echo "djRuZ3U0cmQyMDI0" \| base64 -d` | Decode Base64 string | Password: `v4ngu4rd2024` |

### Phase 4: Steganography

| Step | Command | Purpose | Finding |
|------|---------|---------|---------|
| 1 | `steghide info company_photo.jpg` | Check for hidden data | `final_flag.txt` embedded |
| 2 | `steghide extract -sf company_photo.jpg -p v4ngu4rd2024 -xf flag.txt -f` | Extract hidden file with password | `flag.txt` extracted |

### Phase 5: Submission

| Step | Command | Purpose | Finding |
|------|---------|---------|---------|
| 1 | `cat flag.txt` | Read the flag | `IE3132{gh0st_1n_th3_w0rkst4t10n}` |

## Flag

```
IE3132{gh0st_1n_th3_w0rkst4t10n}
```

## Hints

<details>
<summary><b>Hint 1</b></summary>
"The evidence was deleted, but disk forensics can recover it. Use `fls -r -d disk_image_IE3132.dd` to list deleted files. Look for a ZIP file and note its inode number."
</details>

<details>
<summary><b>Hint 2</b></summary>
"Use the inode number from the previous step to recover the deleted file. Try: `INODE=$(fls -r -d disk_image_IE3132.dd | grep "backup" | awk '{print $3}' | tr -d ':')` then `icat disk_image_IE3132.dd $INODE > recovered.zip`"
</details>

<details>
<summary><b>Hint 3</b></summary>
"The recovered file is a ZIP archive. Extract it with `unzip`. Inside, you'll find an image file."
</details>

<details>
<summary><b>Hint 4</b></summary>
"The image's metadata contains a Base64-encoded string. Use `exiftool` to read it, then decode it with `base64`. The result is a password you'll need later."
</details>

<details>
<summary><b>Hint 5</b></summary>
"Not all secrets are visible. Use `steghide` to check for hidden data in the image. The password from the previous step will unlock it. Extract the hidden file to get the flag."
</details>

## Hidden Artifacts

| Artifact | Location | Value |
|----------|----------|-------|
| Password hint | Image EXIF `ImageDescription` | `djRuZ3U0cmQyMDI0` (Base64) |
| Hint | Image EXIF `XPComment` | "Hint: Image Description" |
| Embedded file | Image steganography | `final_flag.txt` |
| Steghide password | Decoded from Base64 | `v4ngu4rd2024` |
| Flag | `final_flag.txt` | `IE3132{gh0st_1n_th3_w0rkst4t10n}` |

## Stage Specification

| Field | Information |
|-------|-------------|
| Stage ID | CTF-DF |
| Title | The Ghost's Workstation |
| Domain | Digital Forensics, Steganography, OSINT |
| Difficulty | Hard |
| Difficulty Justification | Multi-step chain: disk recovery → archive extraction → metadata analysis → Base64 decoding → steganography. Requires knowledge of 5+ tools and non-obvious pivots. |
| Scenario | Investigator seized a workstation disk image named `disk_image_IE3132.dd` from the whistleblower. The evidence was deleted. Recover it and extract the final flag. |
| Learning Objective | Apply disk forensics, metadata analysis, encoding, and steganography in a chained investigation. |
| Environment | Disk image file (`disk_image_IE3132.dd`) |
| Player Task | Recover deleted ZIP → extract image → read metadata → decode password → extract hidden flag with steghide |
| Tools Required | sleuthkit (fls, icat), exiftool, base64, steghide, unzip |
| Flag Format | `IE3132{gh0st_1n_th3_w0rkst4t10n}` |

## Author Notes

This challenge is designed to simulate a realistic digital forensics investigation scenario. Participants are expected to:

- Apply disk forensics techniques to recover deleted data
- Chain multiple forensic techniques in a logical investigation
- Think critically about each clue and pivot between different tools
- Use both technical tools and analytical thinking

The trail is intentionally multi-layered and requires patience and attention to detail to complete successfully.

---

*Happy investigating, and remember: the evidence was deleted, but the ghost left traces.*
