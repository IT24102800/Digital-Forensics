# README.md — The Ghost's Workstation

```markdown
# The Ghost's Workstation — CTF Challenge

## Overview

**Difficulty:** Hard  
**Theme:** Digital Forensics & Whistleblower Investigation  
**Domains:** Digital Forensics, Steganography, OSINT  

This CTF challenge simulates a real-world forensic investigation where participants must recover deleted evidence from a seized workstation disk image. The trail leads through disk forensics, metadata analysis, Base64 decoding, and steganography to uncover the final flag.

## Challenge Summary

Participants receive a single 10 MB FAT16 disk image seized from a whistleblower's workstation. They must recover a deleted ZIP file, extract a JPG image, analyze its metadata to find a Base64-encoded password, decode it, and then use steganography to extract the final flag hidden inside the image.

### Scenario

Nexora Dynamics, a multinational defense contractor, has been rocked by allegations of illegal arms deals. A senior systems administrator disappeared after copying classified documents to an external drive. The company's internal security team seized his workstation but found nothing. The files had been deleted. A disk image was made before the machine was wiped.

**Your mission:** You are a freelance forensic investigator hired by Nexora's legal team to recover the deleted evidence. Your only lead is the disk image taken from the workstation. The evidence was deleted, but the ghost left traces.

### Starting Point

- **File:** `disk_image_IE3132.dd`
- **Type:** FAT16 disk image (10 MB)

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
| `sleuthkit` | Forensic suite for disk images (`fls`, `icat`) |
| `exiftool` | Read image metadata |
| `base64` | Decode Base64 strings |
| `steghide` | Extract hidden data from images |
| `unzip` | Extract ZIP archives |

## Prerequisites

```bash
sudo apt install -y sleuthkit exiftool steghide unzip
```

## Solution Path

### Phase 1: Reconnaissance

```bash
# Identify file type
file disk_image_IE3132.dd
# Output: FAT16 disk image

# View raw text
strings disk_image_IE3132.dd | head -30
# Output: Decoy filenames, no flag

# List all files (including deleted)
fls -r disk_image_IE3132.dd
# Output: Decoys + backup_2024.zip

# List deleted files only
fls -r -d disk_image_IE3132.dd
# Output: backup_2024.zip (inode 17)

# Check for ZIP signature
strings disk_image_IE3132.dd | grep -i "PK" | head -5
# Output: PK found → ZIP data exists
```

### Phase 2: Recovery

```bash
# Extract inode number of deleted ZIP
INODE=$(fls -r -d disk_image_IE3132.dd | grep "backup" | awk '{print $3}' | tr -d ':')
echo "Inode: $INODE"
# Output: Inode: 17

# Recover deleted file by inode
icat disk_image_IE3132.dd $INODE > recovered.zip
# Output: recovered.zip created

# Verify recovered file type
file recovered.zip
# Output: Zip archive data

# Extract archive
unzip -o recovered.zip
# Output: company_photo.jpg
```

### Phase 3: Metadata Analysis

```bash
# Read image metadata
exiftool company_photo.jpg
# Output: ImageDescription: djRuZ3U0cmQyMDI0

# Decode Base64 string
echo "djRuZ3U0cmQyMDI0" | base64 -d
# Output: v4ngu4rd2024
```

### Phase 4: Steganography

```bash
# Check for hidden data
steghide info company_photo.jpg
# Enter password: v4ngu4rd2024
# Output: final_flag.txt embedded

# Extract hidden file with password
steghide extract -sf company_photo.jpg -p v4ngu4rd2024 -xf flag.txt -f
# Output: flag.txt extracted
```

### Phase 5: Submission

```bash
# Read the flag
cat flag.txt
# Output: IE3132{gh0st_1n_th3_w0rkst4t10n}
```

## Flag

```
IE3132{gh0st_1n_th3_w0rkst4t10n}
```

## Hints

<details>
<summary><b>Hint 1 (Free)</b></summary>
The evidence was deleted, but disk forensics can recover it.
Use `fls -r -d disk_image_IE3132.dd` to list deleted files.
Look for a ZIP file and note its inode number.
</details>

<details>
<summary><b>Hint 2</b></summary>
Use the inode number from the previous step to recover the deleted
file. Try: `icat disk_image_IE3132.dd <inode> > recovered.zip`
</details>

<details>
<summary><b>Hint 3</b></summary>
The recovered file is a ZIP archive. Extract it with `unzip`.
Inside, you'll find an image file.
</details>

<details>
<summary><b>Hint 4</b></summary>
The image's metadata contains a Base64-encoded string.
Use `exiftool` to read it, then decode it with `base64 -d`.
The result is a password you'll need later.
</details>

<details>
<summary><b>Hint 5</b></summary>
Not all secrets are visible. Use `steghide` to check for hidden
data in the image. The password from the previous step will
unlock it. Extract the hidden file to get the flag.
</details>

## Hidden Artifacts

| Artifact | Location | Value |
|----------|----------|-------|
| Password hint | Image EXIF ImageDescription | `djRuZ3U0cmQyMDI0` (Base64) |
| Hint | Image EXIF XPComment | "Hint: decode me" |
| Embedded file | Image steganography | `final_flag.txt` |
| Steghide password | Decoded from Base64 | `v4ngu4rd2024` |
| Flag | `final_flag.txt` | `IE3132{gh0st_1n_th3_w0rkst4t10n}` |

## Red Herrings

| Red Herring | Why It's a Trap |
|-------------|-----------------|
| JWT Token | Decoy cryptographic token |
| admin/T3chP@ss2024! | Fake credentials |
| 192.168.1.105 | Internal IP address (not accessible) |

## Stage Specification

| Field | Information |
|-------|-------------|
| Stage ID | CTF-DF |
| Title | The Ghost's Workstation |
| Domain | Digital Forensics, Steganography, OSINT |
| Difficulty | Hard |
| Environment | Disk image file (`disk_image_IE3132.dd`, 10 MB, FAT16) |
| Player Task | Recover deleted ZIP → extract image → read metadata → decode password → extract hidden flag with steghide |
| Tools Required | `sleuthkit` (fls, icat), `exiftool`, `base64`, `steghide`, `unzip` |
| Flag Format | `IE3132{...}` |

## Author Notes

This challenge is designed to simulate a realistic forensic investigation. Participants are expected to:

- Document their findings professionally
- Think critically about each clue
- Chain multiple forensic techniques
- Use both technical tools and analytical thinking

The trail is intentionally non-obvious and requires knowledge of disk forensics, metadata analysis, and steganography to complete successfully.

---

*Happy investigating. The evidence was deleted — but the ghost left traces.*
```

---

## GitHub Repository Settings

| Field | Value |
|-------|-------|
| **Repository name** | `The-Ghosts-Workstation-CTF` |
| **Description** | `Hard digital forensics CTF — recover deleted ZIP from a FAT16 disk image, decode metadata, extract flag with steganography.` |
| **Topics** | `ctf`, `forensics`, `steganography`, `cybersecurity`, `disk-image`, `sleuthkit`, `steghide`, `exiftool` |
| **License** | Educational use only |

---

## One-Line Description (For GitHub "About")

```
Hard digital forensics CTF — recover deleted evidence from a FAT16 disk image, decode Base64 password, extract hidden flag with steghide.
```

---

**Copy this README.md into your repository. It matches your Assignment 2 document exactly.**
