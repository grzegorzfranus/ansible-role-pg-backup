# Changelog

All notable changes to this Ansible role will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.1] - 2025-12-05

### Added ✅

- Added Debian 11 (Bullseye) as officially supported platform
- Dynamic molecule configuration using `MOLECULE_DISTRO` environment variable

### Changed 🔄

- Removed Rocky Linux 9 from CI test matrix (Debian/Ubuntu only support)
- Added ansible-lint to CI pipeline
- Replaced `community.postgresql` modules with native `psql` commands in prepare.yml

### Fixed 🔧

- Fixed molecule converge.yml `until` condition syntax for Ansible 2.20 compatibility
- Fixed molecule.yml to properly use `MOLECULE_DISTRO` environment variable
- Fixed role reference in converge.yml using `MOLECULE_PROJECT_DIRECTORY` lookup
- Fixed syntax check failures by removing external collection dependencies

---

## [1.0.0] - 2025-12-04

### Added ✅

#### Molecule Testing
- Complete Molecule test suite for role validation
- Test scenarios for Ubuntu 22.04, Ubuntu 24.04, Debian 11, and Debian 12
- Automated PostgreSQL installation in test environment
- Comprehensive verification tests:
  - Script installation and permissions
  - .pgpass configuration security
  - Directory structure validation
  - Cron job configuration
  - Logrotate configuration
  - Dry-run backup execution
  - Actual backup execution with file verification
  - Globals backup verification

#### Professional Logging
- ASCII art banner on script start for clear identification
- Detailed end banner with execution summary
- Professional log format: `TIMESTAMP | LEVEL | COMPONENT | MESSAGE`
- Aligned log levels for better readability
- Total script execution time tracking
- Separate banners for SUCCESS, PARTIAL, and FAILED states

#### Fail-Fast Error Handling
- Immediate script termination on pre-flight check failures
- Sequential validation with clear error messages:
  1. Required commands check
  2. .pgpass authentication check
  3. PostgreSQL connectivity check (CRITICAL)
  4. Backup directory access check
  5. Disk space availability check
- Globals backup failure now aborts entire backup process
- Individual database failures logged but continue with others
- Proper exit codes for all failure scenarios

### Changed 🔄

- Improved log format from bracket style to pipe-separated style
- Enhanced error messages with specific failure context
- Script now returns exit code 1 if any database backup fails

### Fixed 🔧

- Fixed Jinja2 template syntax errors with `${#array[@]}` bash expressions
- Fixed `create` attribute access in logrotate template (reserved keyword)
- Fixed script syntax verification task missing `become: true`
- Escaped all bash array length expressions for Jinja2 compatibility

---

## [0.1.1] - 2025-12-03

### Added ✅

- Database exclusion list (`pg_backup_exclude_databases`) to skip specific databases from backup
- Default exclusion of PostgreSQL template databases (`template0`, `template1`)

### Changed 🔄

- Improved restore instructions with emphasis on restoring global objects (roles) first
- Removed script-based log rotation (now fully managed by logrotate)

### Fixed 🔧

- Clarified restore procedure to prevent permission issues when restoring databases

---

## [0.1.0] - 2025-12-03

### Added ✅

#### Core Features
- Initial release of PostgreSQL backup role
- Professional backup script with custom format (`-Fc`) for easy restoration
- Comprehensive pre-flight validation checks
  - PostgreSQL connectivity verification
  - Disk space availability check
  - `.pgpass` file permissions validation
  - Required commands availability check
- Automatic backup of all databases with date-stamped filenames
- Global objects backup (roles, tablespaces)
- Backup integrity verification using `pg_restore -l`

#### Configuration Management
- Full parameterization via Ansible variables
- Secure `.pgpass` file management with `0600` permissions
- Configurable backup directory structure
- Lock timeout handling to prevent indefinite hangs

#### Retention & Logging
- Automatic cleanup of old backups based on retention policy
- Comprehensive logging with timestamps and severity levels
- Logrotate integration for automatic log rotation
- Configurable log rotation options (frequency, count, compression)

#### Scheduling
- Cron job configuration for automated daily backups
- Configurable backup schedule (minute, hour, day, month, weekday)
- Separate cron output logging

#### Documentation
- Complete README with usage examples
- Detailed restore instructions in script header
- Variable documentation with descriptions
- Troubleshooting guide

#### Quality
- Variable validation with `assert.yml`
- Proper file permissions throughout
- Support for Ubuntu 22.04, 24.04, Debian 11, and Debian 12
