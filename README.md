# Veriduct

**Format Destruction Tool for EDR/DLP Testing**

Veriduct destroys file structure and format signatures to test whether your security controls detect threats based on behavior or just known patterns.

---

## What It Does

Veriduct transforms files into format-agnostic chunks:

- **Destroys file headers** - PDF, DOCX, XLSX become unrecognizable
- **Breaks into salted chunks** - 4KB blocks with file-specific hashing  
- **Generates reconstruction keymap** - Separate file needed for reassembly
- **Tests real security** - Proves your EDR/DLP detects behavior, not just signatures

**Use case:** Test if client DLP blocks file exfiltration or just blocks known file types.

---

## Proof

**EICAR test file:**
- Original: 65/68 detection on VirusTotal
- After Veriduct (with SSM): 0/62 detection on VirusTotal

**Document formats:**
- PDF, DOCX, XLSX: Format completely unrecognizable after processing
- VirusTotal identifies processed files as "SQLite database" instead of original format

---

## Installation

```bash
git clone https://github.com/Bombadil-Systems/Veriduct-Core.git
cd veriduct-core
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

**Annihilation Process:**

1. **Header Randomization** - First 256 bytes (configurable) replaced with random data
2. **Salted Chunking** - File split into 4KB chunks, each hashed with file-specific salt
3. **Database Storage** - Chunks stored in SQLite database without sequence information
4. **Keymap Generation** - Separate compressed file contains reconstruction data

**Output Files:**
- `output/veriduct_chunks.db` - SQLite database with format-destroyed chunks
- `output/veriduct_key.zst` - Compressed keymap (required for reassembly)

**Result:** Chunk database contains no format signatures, headers, or sequence information. Without the keymap, reconstruction is computationally infeasible.

---

## Advanced Options

### Header Wipe Size

Change how many bytes to randomize at the file start:

```bash
python veriduct.py annihilate test.pdf output/ --wipe-bytes 512
```

Default is 256 bytes.

### HMAC Integrity Protection

Add tamper detection to verify chunks haven't been modified:

```bash
python veriduct.py annihilate test.pdf output/ --add-hmac
```

The HMAC is verified during reassembly. Note: This provides integrity verification, not confidentiality.

### Disguised Keymaps

Make the keymap look like a common file type:

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
python veriduct.py reassemble output/veriduct_key.log output/ --disguise log
python veriduct.py reassemble output/veriduct_key.conf output/ --disguise conf
```

### Verbose Output

See detailed per-chunk processing:

```bash
python veriduct.py annihilate test.pdf output/ --verbose
```

---

## Testing Methodology

**Step 1: Baseline Test**
```bash
# Upload original file to VirusTotal
# Record: Detection rate, format identification
```

**Step 2: Process with Veriduct**
```bash
python veriduct.py annihilate sensitive_document.pdf output/
```

**Step 3: Post-Processing Test**
```bash
# Upload output/veriduct_chunks.db to VirusTotal
# Record: Detection rate (should be 0/68 or very low)
# Record: Format identification (should show SQLite, not PDF)
```

**Step 4: Verify Reconstruction**
```bash
python veriduct.py reassemble output/veriduct_key.zst restored/
sha256sum sensitive_document.pdf restored/sensitive_document.pdf
# Hashes must match exactly
```

**Step 5: Document Results**
- Original detection rate vs processed detection rate
- Format identification before/after
- Hash verification results

---

## Use Cases

### Penetration Testing
Test client DLP systems with transformed documents. Prove that security controls rely on signatures rather than behavioral analysis.

### Red Team Exercises
Validate EDR behavioral detection capabilities. Determine if endpoint security can detect unusual file operations beyond signature matching.

### Security Validation
Demonstrate to stakeholders that security controls detect actual threats, not just known file patterns. Build evidence for security control improvements.

---

## What You Get (This Version)

✅ Basic format destruction (header randomization + salted chunking)  
✅ HMAC integrity verification  
✅ Keymap disguise formats (CSV, LOG, CONF)  
✅ Works on any file type  
✅ Batch processing (files and directories)  
✅ Python 3.7+ compatible  

❌ Advanced features not included (see commercial version)

---

## Commercial Version

**$2,500/year per organization**

The commercial version includes additional transformations for comprehensive security testing:

- **Semantic Shatter Mapping (SSM)** - Byte-level permutation within chunks
- **Variable Chunking** - Non-uniform chunk sizes with jitter
- **XOR Entanglement** - Reversible dependencies between chunk groups
- **Substrate Poisoning** - Decoy chunk injection
- **Priority Support** - Direct email support
- **Full Source Code** - All advanced features included

**Contact:** chris@veriduct.com  
**Website:** [veriduct.com](https://veriduct.com)

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

# Custom header wipe size
python veriduct.py annihilate file.bin output/ --wipe-bytes 512
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
- `--ignore-integrity` - Continue even if integrity checks fail (not recommended)
- `--verbose` - Show detailed per-chunk logging

**Examples:**
```bash
# Standard keymap
python veriduct.py reassemble output/veriduct_key.zst restored/

# Disguised keymap
python veriduct.py reassemble output/veriduct_key.log restored/ --disguise log

# Force reassembly despite errors (may produce corrupted files)
python veriduct.py reassemble output/veriduct_key.zst restored/ --ignore-integrity
```

---

## Important Notes

### Data Integrity

- **Both files required:** Chunk database AND keymap needed for reassembly
- **Keymap security:** Protect the keymap separately - compromise enables full reconstruction
- **Lost keymap = permanent data loss:** No recovery possible without the keymap
- **HMAC recommended:** Use `--add-hmac` for tamper detection

### What This Is NOT

Veriduct is designed for **security testing**, not operational data protection:

- ❌ Does NOT provide encryption (chunks are stored unencrypted)
- ❌ Does NOT meet compliance requirements for data-at-rest encryption
- ❌ NOT designed for high-throughput production systems
- ❌ NOT a backup or archival solution

### Forensic Implications

Without the keymap, analysis of the chunk database reveals:
- SQLite database structure (may indicate Veriduct usage)
- Chunk creation timestamps
- No file format signatures or headers
- No file boundaries or sequence information
- No content metadata

File carving tools may identify format boundaries or extract meaningful data from free version.

---

## License

**Commercial license required for:**
- Penetration testing services (paid client engagements)
- Red team consulting (paid services)
- Integration into commercial security products
- Any revenue-generating use or business operations
- Government or defense contract deliverables

Contact chris@veriduct.com for commercial licensing.


---

## Responsible Use

**This tool is for authorized security testing only.**

Users must:
- ✅ Obtain explicit written authorization before testing systems
- ✅ Use only on data they are authorized to access and process
- ✅ Comply with all applicable laws and regulations
- ✅ Follow responsible disclosure for identified vulnerabilities
- ✅ Maintain appropriate data handling and retention practices

⚠️ **Unauthorized use may violate computer fraud, data protection, or other laws.**

---

## Troubleshooting

### "Database file not found next to key file"

The chunk database must be in the same directory as the keymap. If you moved the keymap, move `veriduct_chunks.db` with it.

### "Missing chunk" errors during reassembly

The chunk database has been corrupted or modified. Reassembly cannot complete. This is why `--add-hmac` is recommended.

### "Hash mismatch" or "HMAC mismatch"

File integrity verification failed. Either:
1. Chunks were modified/corrupted
2. Keymap was modified/tampered
3. Wrong keymap used for this chunk database

Do not trust the reassembled output.

### "Unable to process file type" on VirusTotal

**This is the intended result.** VirusTotal cannot identify the file format because format signatures have been destroyed. This proves Veriduct is working correctly.

---

## Contributing

Contributions welcome under Apache 2.0 license.

**Areas of interest:**
- Additional keymap disguise formats
- Performance optimization for large-scale datasets
- Integration examples with security testing frameworks
- Bug fixes and documentation improvements

Submit issues and pull requests on GitHub.

---

## Contact

**Chris Aziz**  
**Bombadil Systems LLC**

- Email: chris@veriduct.com
- Website: [veriduct.com](https://veriduct.com)
- GitHub: [@reapermunky](https://github.com/reapermunky)

For commercial licensing, consulting, or technical support, contact via email.
