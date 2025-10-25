# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Added
- Complete restructuring of documentation with improved organization
- Hardware specifications table for APU2e4 and StarTech adapter
- Comprehensive troubleshooting sections with tables
- References section with organized links by category
- `.gitignore` file for ISO images and configuration files
- CHANGELOG.md for version tracking

### Changed
- Reformatted README.md with proper markdown structure
- Reorganized BIOS upgrade instructions with better formatting
- Enhanced serial console setup documentation with hardware and software sections
- Improved migration guide with step-by-step numbered instructions
- Added table of contents for easier navigation
- Separated dated notes into update log entries

### Fixed
- Fixed typos: "witch" → "which", "jjust" → "just", "accout" → "account", "throught" → "through"
- Corrected formatting of code blocks with proper syntax highlighting
- Clarified BIOS flashing command with board mismatch workaround
- Better organization of migration troubleshooting steps

---

## [1.0] - 2024-10-25

### Initial Release

#### Added
- BIOS upgrade documentation for APU2e4
- Serial console setup guide for macOS
- pfSense to OPNsense migration instructions
- TinyCore USB bootable drive preparation steps
- OPNsense image deployment process
- Notes on serial image compatibility issues

#### Known Issues
- APU2e4 may have compatibility problems with OPNsense serial image (AHCI errors, disk read failures)
- Workaround: Use amd64-nano image instead of serial image

#### Hardware Used
- PC Engines APU2e4 system board
- StarTech USB to Serial RS232 Adapter
- macOS for BIOS flashing and serial console access

#### Notes
- Original documentation by Tristan Greaves provided the foundation
- This document adds practical details from actual implementation experience
- Documentation updated through October 2024

---

## [0.1] - 2024-11-17

### Pre-release Notes
- Latest BIOS version references and flashrom commands documented
- BIOS flashing with `--fmap` and specific chip references
- Update script references from PC Engines (apu_fw_updater_opnsense.sh)
- Command format: `flashrom -p internal -w apu2_v4.19.0.1.rom --fmap -i COREBOOT -c W25Q64BV/W25Q64CV/W25Q64FV`

---

## Documentation Versioning

The documentation follows semantic versioning principles:
- **MAJOR**: Significant restructuring or breaking changes to procedures
- **MINOR**: New sections or substantially improved documentation
- **PATCH**: Typo fixes, formatting improvements, clarifications

## Update History

| Date | Updates | Version |
|------|---------|---------|
| 2024-10-25 | Initial release with comprehensive restructuring | 1.0 |
| 2024-11-17 | Pre-release notes with latest BIOS information | 0.1 |

## Contributing

When updating this documentation:
1. Update version number following semantic versioning
2. Add entry to this CHANGELOG.md
3. Update "Last updated" date in README.md
4. Commit changes with descriptive message
5. Include your name and date in notes if making significant changes

## References

- [PC Engines APU2 Documentation](https://www.pcengines.ch/pdf/apu2.pdf)
- [OPNsense Project](https://opnsense.org)
- [pfSense Project](https://www.pfsense.org)
