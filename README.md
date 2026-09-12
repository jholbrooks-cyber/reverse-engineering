# reverse-engineering

GitLab CI component providing binary reverse-engineering and signature scanning templates.
Each template is an independent job that scans a compiled binary (ELF, PE, shellcode)
for indicators of interest — crypto constants, hardcoded secrets, network IOCs, anti-debug
techniques, packer signatures, shellcode stubs, and more.

## Templates

| File | Scan Type | Tool |
|---|---|---|
| `re-strings.yml` | String extraction + pattern matching | `strings` + Python regex |
| `re-crypto.yml` | Crypto constant / algorithm detection | Python + known byte signatures |
| `re-yara.yml` | YARA signature scanning | `yara-python` |
| `re-entropy.yml` | Per-section entropy analysis | Python (pefile / pyelftools) |
| `re-imports.yml` | Import/export dangerous function analysis | pefile / pyelftools |
| `re-packer.yml` | Packer / protector detection | `binwalk` + DIE heuristics |
| `re-antidebug.yml` | Anti-analysis technique detection | strings + pefile |
| `re-shellcode.yml` | Shellcode stub / stager pattern detection | Python regex |
| `re-network.yml` | Network indicator extraction (IPs, URLs, C2) | Python regex |
| `re-symbols.yml` | Symbol / debug info analysis | pyelftools / pefile |

## Usage

Include one or more templates in your `.gitlab-ci.yml`:

```yaml
stages:
  - reverse-engineer

include:
  - project: homelab/components/reverse-engineering
    ref: main
    file: re-strings.yml
  - project: homelab/components/reverse-engineering
    ref: main
    file: re-crypto.yml
  - project: homelab/components/reverse-engineering
    ref: main
    file: re-entropy.yml

strings-scan:
  extends: .re-strings
  variables:
    RE_TARGET: "build/suspicious.elf"

crypto-scan:
  extends: .re-crypto
  variables:
    RE_TARGET: "build/suspicious.elf"

entropy-scan:
  extends: .re-entropy
  variables:
    RE_TARGET: "build/suspicious.elf"
```

## Common Variables

| Variable | Default | Description |
|---|---|---|
| `RE_TARGET` | `build/target` | Path to binary to analyse |
| `RE_MIN_STRING_LEN` | `6` | Minimum string length for extraction |
| `NEXUS_REPORT_UPLOAD` | `true` | Upload findings to Nexus |
| `NEXUS_BIN_REPORTS_REPO` | `bin-reports` | Nexus raw repo for reports |

## Output Format

All templates produce DefectDojo Generic Findings JSON (`*-findings.json`)
and a human-readable Markdown report (`*-report.md`).
