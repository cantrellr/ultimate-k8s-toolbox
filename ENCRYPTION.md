# Encryption Implementation Plan

**Status**: Planning Phase  
**Target**: AES-256 encryption with certificate-based key management  
**Created**: November 25, 2025

---

## 🎯 Objective

Implement hybrid encryption for offline bundles:
- **AES-256-CBC** for bulk data encryption (tarball)
- **RSA/X.509 certificates** for key encryption (hybrid cryptography)
- Support for **multiple recipient certificates**
- Automated encryption/decryption in Makefile
- Verification and integrity checks

---

## 🔐 Encryption Architecture

### Hybrid Encryption Model

```
┌─────────────────────────────────────────────────────────────┐
│                    ENCRYPTION PROCESS                       │
└─────────────────────────────────────────────────────────────┘

1. Generate random AES-256 key (symmetric)
   └─> 256-bit random key for data encryption

2. Encrypt tarball with AES-256-CBC
   └─> offline-bundle.tar.gz → offline-bundle.tar.gz.enc

3. Encrypt AES key with RSA public key from certificate
   └─> AES key → encrypted_key.bin (per recipient)

4. Create metadata file with:
   - Encryption algorithm (AES-256-CBC)
   - IV (Initialization Vector)
   - Certificate fingerprints
   - File checksums
   - Timestamp

5. Package encrypted bundle:
   offline-bundle-encrypted/
   ├── bundle.tar.gz.enc       (encrypted tarball)
   ├── encryption-metadata.json (encryption details)
   ├── encrypted_keys/
   │   ├── recipient1.key.enc  (encrypted AES key for cert 1)
   │   ├── recipient2.key.enc  (encrypted AES key for cert 2)
   │   └── ...
   └── signatures/
       ├── bundle.sig          (digital signature)
       └── metadata.sig        (metadata signature)
```

### Security Properties

- **Confidentiality**: AES-256-CBC encryption (NIST approved)
- **Authentication**: RSA-SHA256 digital signatures
- **Integrity**: SHA256 checksums throughout
- **Access Control**: Certificate-based authorization
- **Non-repudiation**: Signed metadata with timestamps

---

## 📁 Directory Structure Changes

### New Directories

```
/ultimate-k8s-toolbox/
├── certs/                          (NEW)
│   ├── README.md                   (Certificate management guide)
│   ├── encryption/                 (Encryption certificates)
│   │   ├── recipient1.crt         (Public certificates for encryption)
│   │   ├── recipient2.crt
│   │   └── .gitignore             (Ignore private keys)
│   └── examples/                   (Example certificates for testing)
│       ├── generate-certs.sh      (Helper script)
│       └── test-cert.crt          (Self-signed test cert)
│
├── scripts/
│   ├── encrypt-bundle.sh          (NEW - Encryption script)
│   ├── decrypt-bundle.sh          (NEW - Decryption script)
│   ├── verify-bundle.sh           (NEW - Verification script)
│   └── generate-encryption-cert.sh (NEW - Certificate generation)
│
└── dist/
    ├── offline-bundle/            (Existing unencrypted)
    └── encrypted-bundle/          (NEW - Encrypted output)
        ├── bundle.tar.gz.enc
        ├── encryption-metadata.json
        ├── encrypted_keys/
        └── signatures/
```

---

## 🔧 Makefile Changes

### New Variables

```makefile
ENCRYPTION_ENABLED := false
ENCRYPTION_CERTS_DIR := certs/encryption
ENCRYPTED_BUNDLE_DIR := dist/encrypted-bundle
ENCRYPTED_ARCHIVE := dist/$(CHART_NAME)-encrypted-$(BUNDLE_VERSION).tar.gz
```

### New Targets

#### 1. `generate-encryption-cert`
**Purpose**: Create self-signed certificate for testing

**Actions**:
- Generate RSA 4096-bit private key
- Create X.509 certificate
- Save to `certs/encryption/`
- Set proper permissions (600 for key, 644 for cert)

**Usage**:
```bash
make generate-encryption-cert
make generate-encryption-cert NAME=admin DAYS=3650
```

#### 2. `encrypt-bundle`
**Purpose**: Encrypt the offline bundle

**Dependencies**: `offline-bundle`

**Actions**:
- Check for encryption certificates
- Generate random AES-256 key and IV
- Encrypt tarball with AES-256-CBC
- Encrypt AES key with each recipient certificate
- Generate metadata JSON
- Create digital signatures
- Package encrypted bundle

**Usage**:
```bash
make encrypt-bundle
make encrypt-bundle CERTS="cert1.crt,cert2.crt"
```

#### 3. `decrypt-bundle`
**Purpose**: Decrypt an encrypted bundle

**Actions**:
- Verify signatures
- Decrypt AES key using private key
- Decrypt tarball using AES key
- Verify checksums
- Extract to `dist/offline-bundle/`

**Usage**:
```bash
make decrypt-bundle KEY=certs/encryption/private.key
```

#### 4. `verify-encrypted-bundle`
**Purpose**: Verify encrypted bundle integrity

**Actions**:
- Verify digital signatures
- Check metadata integrity
- Validate certificate fingerprints
- Display bundle information

**Usage**:
```bash
make verify-encrypted-bundle
```

#### 5. `offline-bundle-encrypted`
**Purpose**: Create encrypted offline bundle in one command (meta target)

**Actions**:
- Execute `make offline-bundle`
- Execute `make encrypt-bundle`

**Usage**:
```bash
make offline-bundle-encrypted
```

### Updated Targets

- **`help`**: Add encryption-related commands and documentation
- **`info`**: Add encryption status and certificate information
- **`clean`**: Remove encrypted bundles and temporary encryption files

---

## 📜 Scripts to Create

### 1. `scripts/generate-encryption-cert.sh`

**Purpose**: Generate RSA key pair and X.509 certificate

**Features**:
- Interactive prompts for certificate details
- Support for RSA 2048, 3072, 4096 bits
- Create self-signed or CSR
- Proper permissions handling
- Validation checks

**Usage**:
```bash
./scripts/generate-encryption-cert.sh [options]
  --name <name>        Certificate name
  --bits <2048|4096>   Key size (default: 4096)
  --days <days>        Validity period (default: 3650)
  --output <dir>       Output directory
```

**Example**:
```bash
./scripts/generate-encryption-cert.sh \
  --name admin \
  --bits 4096 \
  --days 3650 \
  --output certs/encryption
```

### 2. `scripts/encrypt-bundle.sh`

**Purpose**: Encrypt offline bundle with AES-256 + certificates

**Features**:
- Multi-recipient support
- Generate random AES key and IV
- Encrypt with AES-256-CBC
- Encrypt AES key with RSA certificates
- Create metadata with timestamps
- Generate SHA256 checksums
- Create digital signatures
- Verify encryption success

**Usage**:
```bash
./scripts/encrypt-bundle.sh \
  --input dist/offline-bundle.tar.gz \
  --output dist/encrypted-bundle \
  --certs certs/encryption/*.crt \
  --sign-key certs/signing/private.key
```

**Parameters**:
- `--input`: Input tarball path
- `--output`: Output directory
- `--certs`: One or more public certificates
- `--sign-key`: Private key for signing (optional)
- `--algorithm`: Encryption algorithm (default: aes-256-cbc)
- `--no-verify`: Skip encryption verification

### 3. `scripts/decrypt-bundle.sh`

**Purpose**: Decrypt encrypted bundle

**Features**:
- Verify signatures before decryption
- Support multiple encrypted keys
- Try keys until success
- Verify checksums after decryption
- Auto-detect encryption metadata
- Resume interrupted decryption

**Usage**:
```bash
./scripts/decrypt-bundle.sh \
  --input dist/encrypted-bundle \
  --output dist/decrypted-bundle.tar.gz \
  --key certs/encryption/private.key \
  --verify-sig certs/signing/public.crt
```

**Parameters**:
- `--input`: Encrypted bundle directory
- `--output`: Output tarball path
- `--key`: Private key for decryption
- `--passphrase`: Key passphrase (or via env: `DECRYPT_PASSPHRASE`)
- `--no-verify`: Skip signature verification
- `--force`: Overwrite existing output

### 4. `scripts/verify-bundle.sh`

**Purpose**: Verify encrypted bundle integrity

**Features**:
- Verify all signatures
- Check metadata integrity
- Validate certificate fingerprints
- Display encryption details
- Check file sizes and checksums

**Usage**:
```bash
./scripts/verify-bundle.sh \
  --bundle dist/encrypted-bundle \
  --cert certs/signing/public.crt
```

---

## 📄 Metadata Format

### encryption-metadata.json

```json
{
  "version": "1.0",
  "created_at": "2025-11-25T10:30:00Z",
  "bundle": {
    "name": "ultimate-k8s-toolbox-offline-v1.0.0.tar.gz",
    "original_size": 1234567890,
    "encrypted_size": 1234567900,
    "sha256": "abc123...",
    "encrypted_sha256": "def456..."
  },
  "encryption": {
    "algorithm": "AES-256-CBC",
    "key_size": 256,
    "iv": "base64_encoded_iv",
    "padding": "PKCS7"
  },
  "key_encryption": {
    "algorithm": "RSA-OAEP",
    "hash_algorithm": "SHA-256",
    "recipients": [
      {
        "id": "recipient1",
        "certificate_fingerprint": "SHA256:abc123...",
        "certificate_subject": "CN=Recipient 1,O=Company",
        "encrypted_key_file": "encrypted_keys/recipient1.key.enc",
        "encrypted_key_sha256": "xyz789..."
      },
      {
        "id": "recipient2",
        "certificate_fingerprint": "SHA256:def456...",
        "certificate_subject": "CN=Recipient 2,O=Company",
        "encrypted_key_file": "encrypted_keys/recipient2.key.enc",
        "encrypted_key_sha256": "uvw012..."
      }
    ]
  },
  "signatures": {
    "bundle": {
      "file": "signatures/bundle.sig",
      "algorithm": "RSA-SHA256",
      "signer_fingerprint": "SHA256:signer123..."
    },
    "metadata": {
      "file": "signatures/metadata.sig",
      "algorithm": "RSA-SHA256",
      "signer_fingerprint": "SHA256:signer123..."
    }
  },
  "tools": {
    "openssl_version": "OpenSSL 3.0.2",
    "created_by": "make encrypt-bundle"
  }
}
```

---

## 🔐 Encryption Workflow

### BUILD SIDE (Online/Source Environment)

**Step 1: Generate or obtain encryption certificates**
```bash
make generate-encryption-cert
# Or: Use existing organizational certificates
```

**Step 2: Create offline bundle**
```bash
make offline-bundle
```

**Step 3: Encrypt the bundle**
```bash
make encrypt-bundle
# Or for one command:
make offline-bundle-encrypted
```

**Step 4: Transfer encrypted bundle to air-gapped environment**
```bash
scp dist/ultimate-k8s-toolbox-encrypted-v1.0.0.tar.gz user@target:/tmp/
```

### DEPLOYMENT SIDE (Offline/Target Environment)

**Step 1: Transfer private key separately (secure channel)**
```bash
# Use physical media, secure key management system, or HSM
```

**Step 2: Extract encrypted bundle**
```bash
tar -xzf ultimate-k8s-toolbox-encrypted-v1.0.0.tar.gz
```

**Step 3: Verify bundle integrity (optional but recommended)**
```bash
./scripts/verify-bundle.sh --bundle encrypted-bundle
```

**Step 4: Decrypt the bundle**
```bash
./scripts/decrypt-bundle.sh \
  --input encrypted-bundle \
  --output offline-bundle.tar.gz \
  --key /secure/path/to/private.key
```

**Step 5: Extract and deploy**
```bash
tar -xzf offline-bundle.tar.gz
cd offline-bundle/scripts
./deploy-offline.sh
```

---

## 🔒 Security Considerations

### Key Management

✓ **Private keys NEVER included in bundle**  
✓ **Private keys stored with 600 permissions**  
✓ **Support for passphrase-protected keys**  
✓ **Optional HSM integration for key storage**  
✓ **Key rotation documentation**  
✓ **Separate key transport mechanism**

### Algorithm Choices

✓ **AES-256-CBC** for bulk encryption (NIST approved)  
✓ **RSA 4096-bit** for key encryption  
✓ **SHA-256** for hashing and signatures  
✓ **PKCS#7 padding**  
✓ **Secure random IV generation**

### Verification

✓ **Digital signatures** on all encrypted files  
✓ **SHA256 checksums** for integrity  
✓ **Certificate fingerprint validation**  
✓ **Metadata signature verification**  
✓ **Post-encryption verification**

### Best Practices

✓ Use organizational PKI when available  
✓ Separate signing and encryption certificates  
✓ Minimum 4096-bit RSA keys  
✓ Regular certificate rotation  
✓ Audit logging for encryption/decryption  
✓ Secure key distribution channels

---

## 🛠️ OpenSSL Commands Reference

### Generate Key Pair
```bash
openssl genrsa -aes256 -out private.key 4096
openssl rsa -in private.key -pubout -out public.pem
```

### Generate Self-Signed Certificate
```bash
openssl req -new -x509 -key private.key -out cert.crt -days 3650
```

### Generate Random AES Key and IV
```bash
openssl rand -hex 32 > aes.key          # 256-bit key
openssl rand -hex 16 > aes.iv           # 128-bit IV
```

### Encrypt File with AES-256-CBC
```bash
openssl enc -aes-256-cbc -salt -pbkdf2 \
  -in bundle.tar.gz \
  -out bundle.tar.gz.enc \
  -K $(cat aes.key) \
  -iv $(cat aes.iv)
```

### Encrypt AES Key with RSA Certificate
```bash
openssl rsautl -encrypt -pubin \
  -inkey public.pem \
  -in aes.key \
  -out aes.key.enc
```

### Decrypt AES Key with Private Key
```bash
openssl rsautl -decrypt \
  -inkey private.key \
  -in aes.key.enc \
  -out aes.key
```

### Decrypt File with AES Key
```bash
openssl enc -d -aes-256-cbc -pbkdf2 \
  -in bundle.tar.gz.enc \
  -out bundle.tar.gz \
  -K $(cat aes.key) \
  -iv $(cat aes.iv)
```

### Create Digital Signature
```bash
openssl dgst -sha256 -sign private.key \
  -out bundle.sig bundle.tar.gz.enc
```

### Verify Digital Signature
```bash
openssl dgst -sha256 -verify public.pem \
  -signature bundle.sig bundle.tar.gz.enc
```

### Extract Public Key from Certificate
```bash
openssl x509 -pubkey -noout -in cert.crt > pubkey.pem
```

### Get Certificate Fingerprint
```bash
openssl x509 -in cert.crt -noout -fingerprint -sha256
```

---

## 📋 Implementation Checklist

### Phase 1: Foundation (Core Infrastructure)
- [ ] Create `certs/` directory structure
- [ ] Create `scripts/` for encryption utilities
- [ ] Add OpenSSL dependency check in Makefile
- [ ] Create `.gitignore` for private keys
- [ ] Update SBOM to include OpenSSL

### Phase 2: Certificate Management
- [ ] Create `generate-encryption-cert.sh` script
- [ ] Create `certs/README.md` documentation
- [ ] Add example self-signed certificates
- [ ] Implement certificate validation functions
- [ ] Add certificate listing/inspection commands

### Phase 3: Encryption Implementation
- [ ] Create `encrypt-bundle.sh` script
- [ ] Implement AES-256 encryption
- [ ] Implement RSA key encryption
- [ ] Add multi-recipient support
- [ ] Create metadata JSON generation
- [ ] Implement digital signatures
- [ ] Add post-encryption verification

### Phase 4: Decryption Implementation
- [ ] Create `decrypt-bundle.sh` script
- [ ] Implement signature verification
- [ ] Implement key decryption
- [ ] Implement AES decryption
- [ ] Add checksum verification
- [ ] Add passphrase support
- [ ] Add error handling and recovery

### Phase 5: Makefile Integration
- [ ] Add `generate-encryption-cert` target
- [ ] Add `encrypt-bundle` target
- [ ] Add `decrypt-bundle` target
- [ ] Add `verify-encrypted-bundle` target
- [ ] Add `offline-bundle-encrypted` meta target
- [ ] Update `help` target with encryption commands
- [ ] Update `info` target with encryption status
- [ ] Update `clean` target for encrypted files

### Phase 6: Verification Tools
- [ ] Create `verify-bundle.sh` script
- [ ] Implement signature verification
- [ ] Implement metadata validation
- [ ] Implement integrity checks
- [ ] Add certificate chain verification
- [ ] Create verification report output

### Phase 7: Documentation
- [ ] Create comprehensive encryption guide (this file)
- [ ] Update `README.md` with encryption section
- [ ] Update `INDEX.md` with encryption references
- [ ] Update `OFFLINE-DEPLOYMENT.md` with encrypted workflow
- [ ] Update `MAKEFILE.md` with encryption targets
- [ ] Create encryption quick reference card
- [ ] Add troubleshooting section

### Phase 8: Testing & Validation
- [ ] Test certificate generation
- [ ] Test single-recipient encryption
- [ ] Test multi-recipient encryption
- [ ] Test decryption workflow
- [ ] Test signature verification
- [ ] Test error conditions
- [ ] Test with passphrase-protected keys
- [ ] Validate against large bundles (1GB+)
- [ ] Test cross-platform compatibility

### Phase 9: Integration & CI/CD
- [ ] Add optional encryption to CI pipeline
- [ ] Create example GitHub Actions workflow
- [ ] Add encryption to release process
- [ ] Document CI/CD integration
- [ ] Add automated testing for encryption

### Phase 10: Advanced Features (Optional)
- [ ] HSM integration support
- [ ] PKCS#11 token support
- [ ] TPM integration
- [ ] Cloud KMS integration (AWS KMS, Azure Key Vault)
- [ ] Age encryption format support
- [ ] GPG/PGP compatibility layer

---

## 💡 Usage Examples (After Implementation)

### Generate Certificates
```bash
make generate-encryption-cert
```

### Create Encrypted Bundle (all-in-one)
```bash
make offline-bundle-encrypted
```

### Create Encrypted Bundle (step-by-step)
```bash
make offline-bundle
make encrypt-bundle
```

### Encrypt for Multiple Recipients
```bash
ENCRYPTION_CERTS="certs/encryption/admin.crt,certs/encryption/ops.crt" \
  make encrypt-bundle
```

### Verify Encrypted Bundle
```bash
make verify-encrypted-bundle
```

### Decrypt Bundle
```bash
make decrypt-bundle KEY=certs/encryption/private.key
```

### Custom Encryption
```bash
./scripts/encrypt-bundle.sh \
  --input dist/offline-bundle.tar.gz \
  --output dist/my-encrypted-bundle \
  --certs certs/encryption/recipient1.crt \
  --certs certs/encryption/recipient2.crt \
  --sign-key certs/signing/private.key
```

### Custom Decryption
```bash
./scripts/decrypt-bundle.sh \
  --input dist/encrypted-bundle \
  --output dist/decrypted.tar.gz \
  --key certs/encryption/private.key \
  --passphrase-env MY_KEY_PASSWORD
```

---

## ⚠️ Important Considerations

### 1. Key Storage
- **NEVER commit private keys to git**
- Use `.gitignore` for `*.key` files
- Store keys in secure key management system
- Consider HSM for production environments

### 2. Performance
- AES encryption is fast (100+ MB/s)
- Large bundles (>1GB) may take 10-30 seconds
- Add progress indicators for user feedback
- Consider parallel encryption for very large files

### 3. Compatibility
- OpenSSL 1.1.1+ required
- Test on Ubuntu 24.04, RHEL 8+, macOS
- Document minimum OpenSSL version
- Provide fallback for older systems

### 4. Backward Compatibility
- Keep unencrypted option as default
- Encryption is opt-in via flag or target
- Maintain existing `offline-bundle` workflow
- Add clear migration path

### 5. Key Distribution
- Private keys must be distributed separately
- Use secure channels (physical media, secure transfer)
- Document key distribution procedures
- Consider split-key scenarios

### 6. Compliance
- Document encryption algorithms (FIPS 140-2)
- Add export control notice if applicable
- Consider regulatory requirements
- Maintain audit logs

---

## 📊 Estimated Implementation Effort

| Component                          | Effort    | Priority |
|------------------------------------|-----------|----------|
| Certificate generation script      | 2 hours   | High     |
| Encryption script                  | 4 hours   | High     |
| Decryption script                  | 3 hours   | High     |
| Makefile integration              | 2 hours   | High     |
| Basic documentation               | 2 hours   | High     |
| Verification script               | 2 hours   | Medium   |
| Comprehensive documentation       | 3 hours   | Medium   |
| Testing & validation              | 4 hours   | High     |
| Example certificates              | 1 hour    | Medium   |
| CI/CD integration                 | 2 hours   | Low      |
| Advanced features (HSM, etc.)     | 8+ hours  | Low      |
| **TOTAL CORE FEATURES**           | **22 hours** |       |
| **TOTAL WITH ADVANCED**           | **30+ hours** |      |

### Recommended Approach

1. **Implement core features first (Phases 1-5)**: ~12 hours
2. **Add verification & docs (Phases 6-7)**: ~7 hours
3. **Test thoroughly (Phase 8)**: ~4 hours
4. **Advanced features as needed (Phase 10)**: Optional

---

## ✅ Benefits of This Approach

### Security
✓ Industry-standard encryption (AES-256)  
✓ Strong key encryption (RSA 4096)  
✓ Digital signatures for authenticity  
✓ Multi-recipient support  
✓ No plaintext keys in bundle

### Usability
✓ Simple make commands  
✓ Automated encryption workflow  
✓ Clear error messages  
✓ Comprehensive documentation  
✓ Example certificates for testing

### Flexibility
✓ Optional - doesn't break existing workflow  
✓ Multiple recipients supported  
✓ Works with organizational PKI  
✓ Scriptable for automation  
✓ Compatible with CI/CD

### Compliance
✓ Audit trail with metadata  
✓ Certificate-based access control  
✓ FIPS-compliant algorithms  
✓ Verifiable integrity  
✓ Non-repudiation with signatures

---

## 📚 Next Steps

1. **Review this plan** and provide feedback
2. **Prioritize** which phases to implement
3. **Begin with Phase 1** (Foundation)
4. **Iterate** through phases with testing
5. **Document** as you go

### Questions to Consider

- Do you want encryption enabled by default or opt-in?
- Should we support organization-provided certificates?
- Are there specific compliance requirements?
- What's the expected bundle size range?
- Will this be used in CI/CD pipelines?

---

## 📖 References

- [NIST SP 800-38A](https://csrc.nist.gov/publications/detail/sp/800-38a/final) - Recommendation for Block Cipher Modes
- [RFC 5280](https://datatracker.ietf.org/doc/html/rfc5280) - X.509 Certificate Profile
- [OpenSSL Documentation](https://www.openssl.org/docs/)
- [FIPS 140-2](https://csrc.nist.gov/publications/detail/fips/140/2/final) - Security Requirements for Cryptographic Modules

---

**Document Status**: Planning Phase  
**Last Updated**: November 25, 2025  
**Next Review**: Before Phase 1 implementation
