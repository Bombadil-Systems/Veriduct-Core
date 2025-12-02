# Veriduct

**Format Destruction for Security Testing**

Veriduct systematically tests whether security controls detect based on behavior rather than just signatures. It destroys file format signatures while maintaining perfect reconstruction capability.

---

## What It Does

Veriduct transforms files through format destruction:

- **Destroys file headers** - PDF, DOCX, XLSX, EXE become unrecognizable
- **Breaks into salted chunks** - 4KB blocks with file-specific hashing, stored without sequence
- **Generates reconstruction keymap** - Separate file enables perfect reassembly
- **Tests behavioral detection** - Proves whether controls detect content or just patterns

**Use case:** Validate whether DLP/EDR systems detect exfiltration based on actual content analysis or just file type recognition.

---

## Validation Results

### Production Malware Testing

**Live APT-level threats from Malware Bazaar:**
- **Cobalt Strike (1.1MB):** 58/72 → 0/62 → 58/72 ✓
- **Emotet (5.4MB):** 34/73 → 0/62 → 34/73 ✓
- **ValleyRAT (314KB):** 51/72 → 0/62 → 51/72 ✓

**Total: 143 detection events → 0 after format destruction → 143 after reconstruction**  
**Verification: SHA256 hash match confirms bit-for-bit identical reconstruction**

### Behavioral Analysis Testing

**ANY.RUN Interactive Malware Sandbox:**
- Original EICAR: Suspicious activity detected, flagged as malicious
- Processed EICAR: No threats detected, identified as SQLite database
- Hash verification: Original and reassembled files are byte-for-byte identical

**This proves format destruction defeats BOTH static signature detection AND behavioral analysis.**

### Baseline EICAR Test

- **Original:** 65/68 detection on VirusTotal
- **After processing:** 0/68 detection on VirusTotal
- **Format identification:** SQLite database (no original format signature)

### Document Formats

**PDF, DOCX, XLSX:** Format completely unrecognizable after processing. VirusTotal identifies all processed documents as "SQLite database" instead of original format.

---

## Installation

```bash
  git clone https://github.com/Bombadil-Systems/Veriduct-Core.git
cd veriduct
pip install -r requirements.txt
```

**Requirements:**
- Python 3.7+
- SQLite3 (included with Python)
- zstandard library

---

## Quick Start

### Basic Usage

**1. Process a file:**
```bash
python veriduct.py annihilate test.pdf output/
```

**2. Reassemble the file:**
```bash
python veriduct.py reassemble output/veriduct_key.zst output/
```

**3. Verify integrity:**
```bash
sha256sum test.pdf output/test.pdf  # Hashes should match
```

### Process a Directory

```bash
python veriduct.py annihilate documents/ output/
python veriduct.py reassemble output/veriduct_key.zst restored/
```

---

## How It Works

### Annihilation Process

**Free Version (This Repository):**

1. **Header Randomization** - First 256 bytes replaced with cryptographic randomness
   - Destroys magic bytes and format signatures
   - Original header stored in keymap for reconstruction

2. **Salted Chunking** - File split into 4KB chunks
   - Each chunk hashed with file-specific salt (prevents hash collisions)
   - Chunks stored in SQLite database without sequence information
   - No file boundaries or ordering preserved

3. **Keymap Generation** - Separate compressed file contains reconstruction data
   - Ordered list of chunk hashes
   - Original file metadata
   - Header restoration information

**Output Files:**
- `output/veriduct_chunks.db` - SQLite database with format-destroyed chunks
- `output/veriduct_key.zst` - Compressed keymap (required for reassembly)

**Result:** Chunk database appears as benign SQLite storage with no format signatures, headers, or sequence information. Without the keymap, reconstruction is computationally infeasible.

---

## Testing Methodology

### Step 1: Baseline Test
```bash
# Upload original file to VirusTotal
# Record: Detection rate, format identification
```

### Step 2: Process with Veriduct
```bash
python veriduct.py annihilate sensitive_document.pdf output/
```

### Step 3: Post-Processing Test
```bash
# Upload output/veriduct_chunks.db to VirusTotal
# Expected: 0/68 or very low detection rate
# Expected: Format identified as SQLite, not original type
```

### Step 4: Verify Reconstruction
```bash
python veriduct.py reassemble output/veriduct_key.zst restored/
sha256sum sensitive_document.pdf restored/sensitive_document.pdf
# Hashes must match exactly - proves controlled transformation
```

### Step 5: Document Results
- Original detection rate vs. processed detection rate
- Format identification before/after
- Hash verification (proof of perfect reconstruction)

---

## Advanced Options

### Header Wipe Size

Change how many bytes to randomize:

```bash
python veriduct.py annihilate test.pdf output/ --wipe-bytes 512
```

Default is 256 bytes (sufficient for most file formats).

### HMAC Integrity Protection

Add tamper detection:

```bash
python veriduct.py annihilate test.pdf output/ --add-hmac
```

HMAC is verified during reassembly. Provides integrity verification, not confidentiality.

### Disguised Keymaps

Make the keymap look like common file types:

```bash
# CSV format (looks like data export)
python veriduct.py annihilate test.pdf output/ --disguise csv

# LOG format (looks like application logs)
python veriduct.py annihilate test.pdf output/ --disguise log

# CONF format (looks like config file)
python veriduct.py annihilate test.pdf output/ --disguise conf
```

**Reassemble with disguised keymap:**
```bash
python veriduct.py reassemble output/veriduct_key.csv output/ --disguise csv
```

### Verbose Output

See detailed per-chunk processing:

```bash
python veriduct.py annihilate test.pdf output/ --verbose
```

---

## Use Cases

### Penetration Testing
Validate client DLP behavioral detection capabilities. Test whether controls detect exfiltration based on content analysis or just pattern matching. Systematic gap analysis for professional deliverables.

### Red Team Operations
Test EDR trust models and process origin validation. Systematic approach without manual obfuscation per engagement. Repeatable methodology across assessments.

### Security Validation
Prove ROI on security control investments. Test defenses under realistic conditions. Gap analysis: signature detection versus behavioral detection.

---

## What's Included (Free Version)

This repository contains the **basic format destruction implementation:**

✅ Header randomization (destroys format signatures)  
✅ Salted chunking (removes sequence information)  
✅ SHA-256 chunk hashing  
✅ SQLite storage (benign appearance)  
✅ HMAC integrity verification  
✅ Keymap disguise formats (CSV, LOG, CONF)  
✅ Perfect reconstruction with hash verification  
✅ Batch processing (files and directories)  
✅ Python 3.7+ compatible  

**Results with free version:**
- EICAR: 65/68 → 0/68 detection
- Documents: Format completely unrecognizable
- Malware: Significant detection rate reduction

---

## Commercial Version

**In Development - Early Access Q1 2026**

The commercial version includes advanced features for comprehensive security testing:

- **Semantic Shatter Mapping (SSM)**
- **XOR Entanglement**
- **Substrate Poisoning**
- **Variable Chunking with Jitter**
- **Professional Support**

**Results with commercial version:**
- Live malware: 143 detection events → 0 (100% evasion rate)
- ANY.RUN behavioral sandbox: Identified as benign
- Production-grade robustness against advanced analysis

**Contact:** chris@bombadil.systems  
**Website:** [veriduct.com](https://veriduct.com)

---

## What's Next

### Current Status
- ✅ Technique validated (VirusTotal, ANY.RUN, live malware)
- ✅ Free version available (this repository)
- ✅ Commercial version in development
- ✅ Presenting at DEF CON DC862 (December 5, 2025)

### Next Steps
- 📋 Vendor notification (responsible disclosure to EDR vendors)
- 💬 Community feedback (tool? service? something else?)
- 🔬 Early access program (limited availability, Q1 2026)
- 🤝 Seeking testing partners with EDR lab access

**Interested in early access or testing partnerships?** Contact chris@bombadil.systems

---

## Command Reference

### Annihilate

```bash
python veriduct.py annihilate <input_path> <output_dir> [options]
```

**Arguments:**
- `input_path` - File or directory to process
- `output_dir` - Where to write chunks.db and keymap

**Options:**
- `--wipe-bytes N` - Randomize first N bytes (default: 256)
- `--add-hmac` - Add integrity verification
- `--disguise {csv,log,conf}` - Output keymap in disguised format
- `--force-internal` - Allow output dir inside input dir
- `--verbose` - Show detailed per-chunk logging

**Examples:**
```bash
# Basic file processing
python veriduct.py annihilate document.pdf output/

# Directory with integrity protection
python veriduct.py annihilate documents/ output/ --add-hmac

# With disguised keymap
python veriduct.py annihilate secret.xlsx output/ --disguise log --add-hmac
```

### Reassemble

```bash
python veriduct.py reassemble <keymap_path> <output_dir> [options]
```

**Arguments:**
- `keymap_path` - Path to veriduct_key.zst or disguised keymap
- `output_dir` - Where to write reassembled files

**Options:**
- `--disguise {csv,log,conf}` - Specify keymap disguise format
- `--ignore-integrity` - Continue even if integrity checks fail
- `--verbose` - Show detailed per-chunk logging

**Examples:**
```bash
# Standard keymap
python veriduct.py reassemble output/veriduct_key.zst restored/

# Disguised keymap
python veriduct.py reassemble output/veriduct_key.log restored/ --disguise log
```

---

## Important Notes

### Data Integrity

- **Both files required:** Chunk database AND keymap needed for reassembly
- **Keymap security:** Protect the keymap - compromise enables full reconstruction
- **Lost keymap = permanent data loss:** No recovery possible without the keymap
- **HMAC recommended:** Use `--add-hmac` for tamper detection

### What This Is NOT

Veriduct is designed for **security testing**, not operational data protection:

- ❌ Does NOT provide encryption (chunks are stored unencrypted)
- ❌ Does NOT meet compliance requirements for data-at-rest encryption
- ❌ NOT designed for high-throughput production systems
- ❌ NOT a backup or archival solution

**This is a testing tool to validate security controls, not a data protection mechanism.**

### Forensic Implications

Without the keymap, analysis of the chunk database reveals:
- SQLite database structure (may indicate Veriduct usage)
- Chunk creation timestamps
- No file format signatures or headers
- No file boundaries or sequence information
- No content metadata

---

## Responsible Use

**⚠️ This tool is for authorized security testing only.**

Users must:
- ✅ Obtain explicit written authorization before testing systems
- ✅ Use only on data they are authorized to access and process
- ✅ Comply with all applicable laws and regulations
- ✅ Follow responsible disclosure for identified vulnerabilities
- ✅ Maintain appropriate data handling and retention practices

**Unauthorized use may violate computer fraud, data protection, or other laws.**

---

## Troubleshooting

### "Database file not found next to key file"
The chunk database must be in the same directory as the keymap. If you moved the keymap, move `veriduct_chunks.db` with it.

### "Missing chunk" errors during reassembly
The chunk database has been corrupted or modified. Reassembly cannot complete. Use `--add-hmac` to detect this earlier.

### "Hash mismatch" or "HMAC mismatch"
File integrity verification failed. Chunks or keymap were modified/tampered. Do not trust the reassembled output.

### "Unable to process file type" on VirusTotal
**This is the intended result.** VirusTotal cannot identify the file format because signatures have been destroyed. This proves Veriduct is working correctly.

---

## License

MIT License - See LICENSE file for details.

**Commercial use restrictions:**
- Penetration testing services (paid client engagements)
- Red team consulting (paid services)
- Integration into commercial security products

Contact chris@bombadil.systems for commercial licensing.

---

## Contact

**Chris Aziz**  
**Bombadil Systems LLC**

- Email: chris@bombadil.systems
- Website: [veriduct.com](https://veriduct.com)
- GitHub: [@reapermunky](https://github.com/reapermunky)

For commercial licensing, early access, testing partnerships, or technical support, contact via email.

---

## Presented At

- **DEF CON DC862** (December 5, 2025) - "Format Destruction for Security Testing"

---

**Veriduct: Test your security controls against format-destroyed threats.**
