# T2HASH CORE Analysis


This repository documents a static and dynamic analysis of the published `t2hash-deploy` binary.

## Summary

The published binary was advertised as a low-level Assembly / Layer-0 / UDP tunneling core.
However, analysis shows that the distributed ELF is primarily an obfuscated shell-script wrapper/installer.

## Main Findings

- Original file is a stripped ELF.
- It imports `execvp`.
- It launches `/bin/bash`.
- Runtime dump revealed a hidden Bash installer.
- The recovered Bash script writes Assembly source files to `/root`.
- It installs `nasm`, `build-essential`, `wget`, and `gzip`.
- It downloads `gost` from GitHub without checksum verification.
- It creates systemd services as root.
- It applies network buffer sysctl changes.
- It uses a simple XOR `0x5A` operation in the generated Assembly code.

## Important Finding

The binary itself is not a transparent Assembly core.
It is an obfuscated ELF wrapper that decodes and runs a Bash installer.

## Security Concerns

This installer should not be run blindly as root because it:

- Is obfuscated.
- Hides its real Bash payload.
- Downloads a binary without checksum verification.
- Creates root-level systemd services.
- Opens network services.
- Removes previous files and services with matching names.
- Does not provide strong authentication or cryptographic protection.

## Recovered File

Recovered script path:

    recovered/recovered_t2hash.sh

## Evidence

Evidence files:

    evidence/execvp-output.txt
    evidence/findings-short.txt

## Conclusion

This is not confirmed classic malware based on the recovered script, but it is an unsafe and misleading root installer.

Calling an obfuscated Bash installer "Layer-0 Assembly Core" is not engineering; it is marketing.
