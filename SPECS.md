# ZFS Root Installation - Technical Specifications

**Copyright © 2025 Michael C. Tinsay**  
Licensed under the GNU General Public License v3.0

> **🤖 AI-Generated Documentation**: This specification document was entirely created by AI without direct human editing or comprehensive review. Validate all technical details independently before relying on this information.

## System Requirements

### Hardware Requirements
- **Architecture**: x86_64 (AMD64)
- **Boot Mode**: UEFI (Legacy BIOS not supported)
- **Memory**: Minimum 4GB RAM (8GB recommended for ZFS)
- **Storage**: Minimum 20GB available disk space
- **Network**: Internet connection required for package downloads

### Software Requirements
- **Host OS**: Ubuntu 22.04 LTS or compatible Debian-based system
- **ZFS**: OpenZFS kernel modules and utilities
- **Debootstrap**: For base system installation
- **GRUB**: EFI bootloader support

## Installation Specifications

### Partition Layout (Auto Mode)

| Partition | Size | Type | Filesystem | Label | Mount Point |
|-----------|------|------|------------|-------|-------------|
| EFI | 512MB | EF00 | FAT32 | EFI | /boot/efi |
| Boot | 1GB | 8300 | ext4 | boot | /boot |
| Swap | Configurable | 8200 | swap | - | swap |
| Root | Remaining | BF00 | ZFS | - | / |

### ZFS Pool Configuration

```bash
# Default pool configuration
POOL_NAME="rpool"
ROOT_DATASET_NAME="ROOT/ubuntu"

# Pool properties
ashift=12                    # 4K sector alignment
compression=lz4             # Fast compression
atime=off                   # Disable access time updates
xattr=sa                    # System attribute storage
dnodesize=auto              # Automatic dnode sizing
```

### ZFS Dataset Structure

```
rpool/ROOT/ubuntu           # Root filesystem (/) - devices=on
├── rpool/ROOT/ubuntu/var   # Variable data (/var)
├── rpool/ROOT/ubuntu/home  # User home directories (/home)
└── rpool/ROOT/ubuntu/...   # Additional configurable datasets
```

### Dataset Properties

| Dataset | Property | Value | Purpose |
|---------|----------|-------|---------|
| Root | devices | on | Allow device files |
| Root | canmount | on | Enable mounting |
| Root | mountpoint | / | Root mount point |
| All | compression | lz4 | Fast compression |
| All | atime | off | Performance optimization |

## Package Specifications

### Base System Packages
```bash
# Essential packages installed via debootstrap
ubuntu-minimal
ubuntu-standard
linux-generic
zfs-initramfs
```

### Bootloader Packages
```bash
grub-efi-amd64              # Base GRUB EFI support
grub-efi-amd64-signed       # Signed GRUB for Secure Boot
shim-signed                 # Signed shim bootloader
```

### ZFS Packages
```bash
zfsutils-linux              # ZFS utilities
zfs-zed                     # ZFS Event Daemon
```

## Configuration File Specifications

### ubuntu-config.sh Structure

```bash
# Distribution Configuration
DEBOOTSTRAP_SUITE="jammy"
DEBOOTSTRAP_COMPONENTS="main restricted universe multiverse"
DEBOOTSTRAP_MIRROR="http://archive.ubuntu.com/ubuntu"

# System Configuration
HOSTNAME="ubuntu-zfs"
USERNAME="user"
TIMEZONE="UTC"
LOCALE="en_US.UTF-8"

# ZFS Configuration
POOL_NAME="rpool"
ROOT_DATASET_NAME="ROOT/ubuntu"
ZFS_DATASETS=(
    "var:/var:compression=lz4"
    "tmp:/tmp:compression=lz4"
    "home:/home:compression=lz4"
)

# Disk Configuration
PARTITION_MODE="auto"
SWAP_SIZE="4G"
```

## Network Configuration

### DNS Resolution (Chroot)
```bash
# /etc/resolv.conf content
nameserver 1.1.1.1
nameserver 8.8.8.8
```

### Repository Configuration
```bash
# /etc/apt/sources.list
deb http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-updates main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-security main restricted universe multiverse
```

## Security Specifications

### Secure Boot Support
- **Shim Bootloader**: Microsoft-signed first-stage bootloader
- **Signed GRUB**: Ubuntu-signed second-stage bootloader
- **Kernel Verification**: Signed kernel modules and initramfs

### File Permissions
```bash
# Critical file permissions
/boot/efi           755 root:root
/boot               755 root:root
/etc/fstab          644 root:root
/etc/crypttab       600 root:root (if encryption used)
```

## Performance Specifications

### ZFS Tuning Parameters
```bash
# Recommended ZFS module parameters
options zfs zfs_arc_max=2147483648    # 2GB ARC limit
options zfs zfs_prefetch_disable=1    # Disable prefetch
options zfs zfs_txg_timeout=5         # 5-second transaction timeout
```

### I/O Scheduler
```bash
# Recommended I/O scheduler for ZFS
echo mq-deadline > /sys/block/sdX/queue/scheduler
```

## Compatibility Matrix

### Supported Ubuntu Versions
| Version | Codename | Support Status |
|---------|----------|----------------|
| 22.04 LTS | Jammy | Full Support |
| 20.04 LTS | Focal | Compatible |
| 24.04 LTS | Noble | Future Support |

### ZFS Version Compatibility
| ZFS Version | Ubuntu Version | Features |
|-------------|----------------|----------|
| 2.1.x | 22.04 | Full feature set |
| 2.0.x | 20.04 | Legacy support |
| 2.2.x | 24.04+ | Future features |

## Error Codes and Recovery

### Exit Codes
| Code | Meaning | Recovery Action |
|------|---------|-----------------|
| 0 | Success | None |
| 1 | Configuration error | Check ubuntu-config.sh |
| 2 | Disk operation failed | Verify disk accessibility |
| 3 | Network error | Check internet connection |
| 4 | Package installation failed | Check repositories |
| 5 | ZFS operation failed | Check ZFS modules |

### Recovery Procedures
1. **Stage 1 Failure**: Re-run ubuntu-stage1.sh with -D flag
2. **Stage 2 Failure**: Manual chroot and run ubuntu-stage2.sh
3. **Stage 3 Failure**: Run ubuntu-stage3.sh independently
4. **Boot Failure**: Boot from live media and repair GRUB

## Testing Specifications

### Test Environments
- **Virtual Machines**: QEMU/KVM, VirtualBox, VMware
- **Physical Hardware**: Various UEFI systems
- **Cloud Instances**: AWS, Azure, GCP (where applicable)

### Test Cases
1. **Fresh Installation**: Clean disk, auto mode
2. **Existing System**: Manual mode with existing partitions
3. **Error Recovery**: Simulated failures and recovery
4. **Performance**: I/O benchmarks and ZFS performance
5. **Security**: Secure Boot verification and file permissions#
# Recent Technical Enhancements

### Disk Safety Specifications

**Pre-operation Safety Checks**:
```bash
# Comprehensive partition unmounting
unmount_all_partitions_on_disk() {
    # 1. Disable all swap partitions on target disk
    # 2. Unmount all mounted filesystems
    # 3. Force lazy unmount for stubborn mounts
}
```

**Safety Features**:
- **Swap detection**: Uses `swapon --show` to identify active swap partitions
- **Mount detection**: Uses `mount | grep` to find mounted filesystems
- **Lazy unmount**: Uses `umount -l` as fallback for busy filesystems
- **Error tolerance**: Continues operation even if some unmounts fail

### Enhanced fstab Specifications

**fstab Entry Format**:
```bash
# Boot partition
/dev/disk/by-uuid/12345678-1234-1234-1234-123456789012 /boot ext4 defaults 0 2

# EFI partition  
/dev/disk/by-uuid/ABCD-EFGH /boot/efi vfat umask=0077 0 1
```

**Benefits**:
- **Explicit paths**: More explicit than `UUID=` shorthand
- **Tool compatibility**: Works with all Linux filesystem utilities
- **Debug friendly**: Easy to verify device path existence

### ZFS Root Dataset Specifications

**Root Dataset Properties**:
```bash
# Root dataset creation with device access
zfs create -o mountpoint=/ -o canmount=on -o devices=on $POOL_NAME/$ROOT_DATASET_NAME
```

**Device Access Requirements**:
- **`devices=on`**: Allows device file creation and access
- **Essential for root**: Required for `/dev/null`, `/dev/zero`, etc.
- **Boot compatibility**: Prevents device access issues during startup

### Chroot Environment Specifications

**DNS Configuration**:
```bash
# Reliable DNS setup (no symlink copying)
if [[ -f "$INSTALL_ROOT/etc/resolv.conf" ]]; then
    mv "$INSTALL_ROOT/etc/resolv.conf" "$INSTALL_ROOT/etc/resolv.conf.backup"
fi
cat > "$INSTALL_ROOT/etc/resolv.conf" << 'EOF'
nameserver 1.1.1.1
nameserver 8.8.8.8
EOF
```

**APT Cache Management**:
```bash
# Copy entire apt cache (no bind mounting)
cp -r /var/cache/apt "$INSTALL_ROOT/var/cache/"
```

**Security Improvements**:
- **No EFI variable access**: Eliminates `efivarfs` mounting in chroot
- **Backup preservation**: Saves original `resolv.conf` before replacement
- **Cache independence**: Copied cache survives host system changes

### Package Configuration Specifications

**Non-interactive Configuration**:
```bash
# Already-installed packages use dpkg-reconfigure
dpkg-reconfigure -f noninteractive keyboard-configuration
dpkg-reconfigure -f noninteractive console-setup
```

**GRUB EFI Package Set**:
```bash
# Complete GRUB EFI installation
apt install -y grub-efi-amd64 grub-efi-amd64-signed shim-signed
```

### Mount Cleanup Specifications

**Comprehensive Cleanup Algorithm**:
```bash
# Advanced mount cleanup
mount | grep -v zfs | tac | awk '/\/mnt/ {print $3}' | xargs -i{} umount -lf {}
```

**Cleanup Process**:
1. **`mount`**: List all current mounts
2. **`grep -v zfs`**: Exclude ZFS filesystems  
3. **`tac`**: Reverse order for proper unmounting sequence
4. **`awk '/\/mnt/ {print $3}'`**: Extract mount points under `/mnt`
5. **`xargs -i{} umount -lf {}`**: Lazy force unmount each mount point

**Benefits**:
- **Comprehensive**: Catches all non-ZFS mounts automatically
- **Order-aware**: Unmounts in reverse mount order
- **Force capable**: Uses `-lf` flags for stubborn filesystems
- **ZFS-safe**: Handles ZFS mounts separately## Advanc
ed Error Handling Specifications

### Bash Options Configuration

**Enhanced Error Handling**:
```bash
set -Euo pipefail
```

| Option | Purpose | Benefit |
|--------|---------|---------|
| `-E` | ERR trap inheritance | Functions and subshells inherit error handling |
| `-u` | Unset variable errors | Prevents typos and undefined variable issues |
| `-o pipefail` | Pipeline failure detection | Catches failures in command pipelines |

### Debug System Specifications

**Debug Activation Matrix**:

| Config DEBUG | Command Line | Result | Timing |
|--------------|-------------|--------|---------|
| `false` | None | No debug | N/A |
| `true` | None | Debug enabled | After parameter processing |
| `false` | `-D` | Debug enabled | After parameter processing |
| `true` | `-D` | Debug enabled | After parameter processing |

**Debug Tracing Features**:
```bash
# Command visibility when DEBUG=true
set -x  # Shows every command execution
```

**Debug Propagation**:
- **Stage1 → Stage2**: Passes `-D` flag when DEBUG=true
- **Stage1 → Stage3**: Passes `-D` flag when DEBUG=true
- **Consistent behavior**: All stages respect DEBUG setting

## File Handling Specifications

### Symlink-Safe Operations

**resolv.conf Detection**:
```bash
# Comprehensive existence check
if [[ -f "$INSTALL_ROOT/etc/resolv.conf" ]] || [[ -L "$INSTALL_ROOT/etc/resolv.conf" ]]; then
    # Handle both files and symlinks
fi
```

**File Type Compatibility**:
- **Regular files**: Standard file handling
- **Symbolic links**: Proper symlink detection and handling
- **systemd-resolved**: Compatible with `/run/systemd/resolve/stub-resolv.conf`
- **NetworkManager**: Handles dynamic DNS configurations

### Command Execution Specifications

**Space-Safe Execution**:
```bash
# Problem: Word splitting with spaces in paths
$command_with_spaces

# Solution: Proper evaluation
eval "$command_with_spaces"
```

**Execution Safety Features**:
- **Path handling**: Supports script paths with spaces
- **Argument preservation**: Maintains command structure
- **Quote safety**: Preserves quoting throughout execution

## Mount Management Specifications

### Advanced Cleanup Algorithm

**Comprehensive Mount Detection**:
```bash
mount | grep -v zfs | tac | awk '/\/mnt/ {print $3}' | xargs -i{} umount -lf {}
```

**Algorithm Breakdown**:
1. **`mount`**: List all current mounts
2. **`grep -v zfs`**: Exclude ZFS filesystems (handled separately)
3. **`tac`**: Reverse order for dependency-safe unmounting
4. **`awk '/\/mnt/ {print $3}'`**: Extract mount points under `/mnt`
5. **`xargs -i{} umount -lf {}`**: Lazy force unmount each mount point

**Unmount Options**:
- **`-l` (lazy)**: Detach filesystem immediately, cleanup when not busy
- **`-f` (force)**: Force unmount even if busy
- **Error tolerance**: Continues even if some unmounts fail

## fstab Management Specifications

### UUID Reference Format

**Enhanced fstab Entries**:
```bash
# Boot partition
/dev/disk/by-uuid/12345678-1234-1234-1234-123456789012 /boot ext4 defaults 0 2

# EFI partition
/dev/disk/by-uuid/ABCD-EFGH /boot/efi vfat umask=0077 0 1
```

**Format Benefits**:
- **Explicit paths**: More explicit than `UUID=` shorthand
- **Tool compatibility**: Works with all Linux utilities
- **Debug support**: Easy to verify path existence
- **Recovery friendly**: Clear device identification

## ZFS Dataset Specifications

### Root Dataset Properties

**Essential Root Dataset Configuration**:
```bash
zfs create -o mountpoint=/ -o canmount=on -o devices=on $POOL_NAME/$ROOT_DATASET_NAME
```

**Property Specifications**:
| Property | Value | Purpose |
|----------|-------|---------|
| `mountpoint` | `/` | Root filesystem mount point |
| `canmount` | `on` | Enable mounting capability |
| `devices` | `on` | Allow device file access |

**Device Access Requirements**:
- **Essential device files**: `/dev/null`, `/dev/zero`, `/dev/random`
- **System compatibility**: Standard Linux device expectations
- **Application support**: Required for many system utilities
- **Boot reliability**: Prevents startup failures

## Package Configuration Specifications

### Non-Interactive Configuration

**dpkg-reconfigure Usage**:
```bash
dpkg-reconfigure -f noninteractive package-name
```

**Configuration Parameters**:
- **`-f noninteractive`**: Suppress all user prompts
- **Package-specific**: Applied to already-installed packages
- **Consistent results**: Predictable configuration outcomes

**Supported Packages**:
- **locales**: Language and character set configuration
- **keyboard-configuration**: Keyboard layout setup
- **console-setup**: Console font and encoding
- **tzdata**: Timezone configuration

## Security Specifications

### Chroot Environment Security

**Isolation Features**:
- **No EFI variable access**: Prevents accidental firmware modification
- **Independent APT cache**: Copied instead of bind-mounted
- **Reliable DNS**: Fixed nameservers (1.1.1.1, 8.8.8.8)
- **Configuration backup**: Original files preserved

**Security Benefits**:
- **Reduced attack surface**: Limited host system access
- **Accident prevention**: No critical system modification
- **Recovery support**: Original configurations preserved
- **Network reliability**: DNS works regardless of host configuration## Par
tition Processing Specifications

### Disk Setup Sequence

**Stage 1 Partition Processing Pipeline**:

| Step | Function | Purpose | Output |
|------|----------|---------|---------|
| 1 | `partition_disk()` | Create all partitions | Partition devices |
| 2 | `format_boot()` | Format boot partition | ext4 filesystem + UUID |
| 3 | `format_efi()` | Format EFI partition | FAT32 filesystem + UUID |
| 4 | `format_swap()` | Format swap partition | Swap signature + UUID |
| 5 | `create_zfs_pools()` | Create ZFS pool | ZFS pool |
| 6 | `create_datasets()` | Create ZFS datasets | ZFS datasets |
| 7 | `configure_dataset_mounting()` | Generate fstab | Complete fstab file |

### Swap Partition Specifications

**Swap Formatting Parameters**:
```bash
# Auto mode (destructive)
mkswap -f $SWAP_PARTITION

# Manual mode (standard)
mkswap $SWAP_PARTITION
```

**Swap fstab Entry Format**:
```bash
/dev/disk/by-uuid/SWAP-UUID none swap sw 0 0
```

**Swap Configuration Matrix**:

| SWAP_SIZE | SWAP_PARTITION | Auto Mode Result | Manual Mode Result |
|-----------|----------------|------------------|-------------------|
| "0" | - | No swap partition | No swap setup |
| "4G" | - | 4GB swap partition | Error (no partition) |
| - | "/dev/sda3" | Uses existing partition | Formats partition |

### UUID Generation Specifications

**UUID Availability Timeline**:
```
Partition Creation → Filesystem Formatting → UUID Generation → fstab Creation
```

**UUID Detection Commands**:
```bash
# Boot partition UUID
boot_uuid=$(blkid -s UUID -o value "$BOOT_PARTITION" 2>/dev/null || echo "")

# EFI partition UUID  
efi_uuid=$(blkid -s UUID -o value "$EFI_PARTITION" 2>/dev/null || echo "")

# Swap partition UUID
swap_uuid=$(blkid -s UUID -o value "$SWAP_PARTITION" 2>/dev/null || echo "")
```

**fstab Entry Specifications**:
```bash
# Consistent format for all partition types
/dev/disk/by-uuid/$uuid $mountpoint $fstype $options $dump $pass
```

### Error Handling Specifications

**Partition Processing Error Handling**:
- **Creation failures**: Stop before formatting
- **Format failures**: Report specific partition and continue
- **UUID detection failures**: Fall back to device path
- **fstab generation**: Always succeeds with available information

**Swap-Specific Error Handling**:
```bash
# Auto mode with no SWAP_SIZE
if [[ "$SWAP_SIZE" == "0" || -z "$SWAP_SIZE" ]]; then
    # No swap partition created or formatted
fi

# Manual mode with missing partition
if [[ -n "$SWAP_PARTITION" && ! -b "$SWAP_PARTITION" ]]; then
    error "Swap partition $SWAP_PARTITION does not exist"
fi
```

### Performance Specifications

**Disk I/O Optimization**:
- **Sequential operations**: All formatting happens consecutively
- **Minimal seeks**: Operations grouped by disk area
- **Efficient UUID detection**: Single `blkid` call per partition
- **Batch fstab generation**: All entries created in one operation

**Memory Usage**:
- **UUID caching**: UUIDs stored in variables for reuse
- **Minimal overhead**: No unnecessary filesystem operations
- **Clean state**: No temporary files or mounts during setup## Advance
d Integration Specifications

### ZFS Mount Management Specifications

**Native ZFS Mounting Commands**:
```bash
# Root dataset mounting
zfs mount $POOL_NAME/$ROOT_DATASET_NAME

# Existing dataset configuration
zfs set mountpoint=/ $EXISTING_ROOT_DATASET
zfs set canmount=on $EXISTING_ROOT_DATASET
zfs mount $EXISTING_ROOT_DATASET
```

**ZFS Mount Properties**:
| Property | Value | Purpose |
|----------|-------|---------|
| `mountpoint` | `/` | Root filesystem mount point |
| `canmount` | `on` | Enable automatic mounting |
| `devices` | `on` | Allow device file access |

**Mount Behavior Matrix**:
| Scenario | Command | Result |
|----------|---------|---------|
| New dataset | `zfs mount $DATASET` | Mounts at `/` with ZFS management |
| Existing dataset | `zfs set mountpoint=/ && zfs mount` | Updates properties and mounts |
| Auto mode | `zfs mount` after creation | Immediate mounting with proper properties |

### GRUB Configuration Specifications

**Parameter Escaping Rules**:
```bash
# Sed delimiter escaping for ZFS paths
Original: $POOL_NAME/$ROOT_DATASET_NAME
Escaped:  $POOL_NAME\/$ROOT_DATASET_NAME
```

**GRUB Parameter Format**:
```bash
# Correct root parameter format
root=ZFS=rpool\/ROOT\/ubuntu
```

**Escaping Requirements**:
- **Forward slashes**: Must be escaped as `\/` in sed commands
- **Delimiter conflicts**: Prevents sed parsing errors
- **Path safety**: Works with any dataset naming convention

### File System Integration Specifications

**os-release Handling Algorithm**:
```bash
# Detection and copying logic
1. Check if /etc/os-release is symlink: [[ -L "/etc/os-release" ]]
2. If symlink: readlink -f to get target, copy target file
3. If regular file: direct copy
4. If missing: log warning and continue
```

**Symlink Resolution**:
```bash
# Common symlink patterns
/etc/os-release → /usr/lib/os-release
/etc/os-release → ../usr/lib/os-release
```

**File Copy Specifications**:
| Source Type | Detection | Action | Result |
|-------------|-----------|--------|---------|
| Symlink | `[[ -L ]]` | `readlink -f` + copy target | Content copied |
| Regular file | `[[ -f ]]` | Direct copy | File copied |
| Missing | `! [[ -f ]]` | Warning logged | Installation continues |

### Execution Flow Specifications

**Stage 1 Optimized Sequence**:
```
Step 1: partition_disk()           → All partitions created
Step 2: create_zfs_pools()         → ZFS pool infrastructure
Step 3: create_datasets()          → ZFS datasets with native mounting
Step 4: chroot_setup()            → Early chroot environment
Step 5: format_boot/efi/swap()    → Traditional partition formatting
Step 6: configure_dataset_mounting() → fstab generation with UUIDs
Step 7: install_base_system()     → Ubuntu base installation
Step 8: configure_system()        → System config with os-release
```

**Timing Dependencies**:
- **ZFS before formatting**: Ensures ZFS infrastructure exists first
- **Chroot after datasets**: Environment ready for subsequent operations
- **fstab after formatting**: UUIDs available for proper references
- **os-release in config**: System identity established during setup

### Error Handling Specifications

**ZFS Mount Error Handling**:
```bash
# Mount failure scenarios
- Dataset doesn't exist: zfs mount fails gracefully
- Mount point busy: ZFS handles conflicts automatically
- Permission issues: Proper error reporting with context
```

**GRUB Configuration Error Handling**:
```bash
# Sed operation safety
- Delimiter conflicts: Prevented by proper escaping
- Missing parameters: Conditional addition prevents duplicates
- Malformed paths: Escaping handles all valid ZFS dataset names
```

**File Operation Error Handling**:
```bash
# os-release copy resilience
- Broken symlinks: Warning logged, installation continues
- Permission denied: Error logged with context
- Missing source: Warning logged, system remains functional
```## Command
 Line Interface Specifications

### Parameter Processing Specifications

**Supported Parameters**:
| Parameter | Short | Long | Argument | Purpose |
|-----------|-------|------|----------|---------|
| Yes | `-y` | `--yes` | None | Skip confirmation prompts |
| Debug | `-D` | `--debug` | None | Enable debug tracing |
| Hostname | `-h` | `--hostname` | Required | Override hostname |
| Disk | `-d` | `--disk` | Required | Override disk device |

**Parameter Validation Rules**:
```bash
# Argument validation for parameters requiring values
if [[ -n "$2" && "$2" != -* ]]; then
    # Valid argument provided
else
    # Error: missing or invalid argument
fi
```

**Validation Specifications**:
- **Argument presence**: Must provide non-empty argument
- **Argument format**: Must not start with `-` (not another parameter)
- **Error handling**: Clear error message and exit on validation failure
- **Shift behavior**: `shift 2` for parameters with arguments, `shift` for flags

### Configuration Override Specifications

**Override Behavior Matrix**:
| Config Variable | Command Line | Result | Precedence |
|----------------|--------------|--------|------------|
| `HOSTNAME="server"` | `-h web01` | `HOSTNAME="web01"` | Command line wins |
| `DISK="/dev/sda"` | `-d /dev/nvme0n1` | `DISK="/dev/nvme0n1"` | Command line wins |
| `DEBUG="false"` | `-D` | `DEBUG="true"` | Command line wins |

**Override Implementation**:
- **Direct assignment**: Parameters directly modify config variables
- **Timing**: Override happens after config loading, before validation
- **Persistence**: Override values used throughout entire installation

### Stage 2 Execution Specifications

**Optimized Execution Order**:
```
1. validate_stage2_config()     → Configuration validation
2. configure_locale()           → System locale setup
3. configure_timezone()         → Timezone configuration
4. update_system()             → System updates
5. install_kernel()            → Kernel installation
6. install_grub()              → Bootloader setup
7. configure_zfs()             → ZFS configuration
8. configure_zfs_mounting()    → ZFS mount setup
9. install_openssh()           → SSH server setup
10. install_additional_packages() → Additional package installation
11. create_users()             → User account creation
12. final_configuration()      → Final system setup
```

**User Creation Timing Benefits**:
- **Package availability**: All packages installed before user setup
- **Tool access**: User creation scripts have access to all installed tools
- **System completeness**: Users created on fully configured system
- **Dependency resolution**: No missing dependencies during user setup

### Usage Specifications

**Command Line Examples**:
```bash
# Basic installation
sudo ./ubuntu-stage1.sh

# Automated installation with custom hostname
sudo ./ubuntu-stage1.sh -y -h webserver

# Debug installation with custom disk
sudo ./ubuntu-stage1.sh -D -d /dev/nvme0n1

# Full customization
sudo ./ubuntu-stage1.sh -y -D -h database01 -d /dev/sdb
```

**Parameter Combination Rules**:
- **All parameters optional**: Script works with any combination
- **Order independent**: Parameters can be specified in any order
- **Multiple values**: Last value wins for repeated parameters
- **Validation**: Each parameter validated independently## Ad
vanced ZFS and System Identity Specifications

### ZFS Dataset Mount Management Specifications

**Manual Mode Mount Requirements**:
```bash
# Existing root dataset mounting
zfs mount "$EXISTING_ROOT_DATASET"

# Reused pool default dataset mounting  
zfs mount "$POOL_NAME/$ROOT_DATASET_NAME"
```

**Mount Decision Logic**:
| Pool State | Dataset State | Action | Command |
|------------|---------------|--------|---------|
| New | New dataset | Auto-mount | None (canmount=on) |
| Existing | Existing dataset | Manual mount | `zfs mount` |
| Reused | Existing dataset | Manual mount | `zfs mount` |
| Reused | New dataset | Auto-mount | None (canmount=on) |

**Mount Timing Specifications**:
- **After property setting**: Mount commands execute after `zfs set` operations
- **Before system operations**: Datasets mounted before subsequent installation steps
- **Pool import recovery**: Manual mounting handles post-import mount states

### System Identity Management Specifications

**Identity Preservation Pipeline**:
```
Stage 1: Host → $INSTALL_ROOT/tmp/os-release
Stage 2: /tmp/os-release → /etc/os-release (before update-grub)
```

**File Handling Specifications**:
```bash
# Symlink-safe detection
[[ -f "/etc/os-release" ]] || [[ -L "/etc/os-release" ]]

# Backup creation
mv "/etc/os-release" "/etc/os-release.backup"

# Identity replacement
mv "/tmp/os-release" "/etc/os-release"
```

**Identity File States**:
| Source Type | Detection | Backup Action | Result |
|-------------|-----------|---------------|---------|
| Regular file | `[[ -f ]]` | Move to .backup | File backed up |
| Symlink | `[[ -L ]]` | Move to .backup | Symlink backed up |
| Missing | Neither | No action | No backup needed |

### Error Handling Specifications

**ZFS Mount Error Handling**:
- **Dataset missing**: `zfs mount` fails gracefully, installation continues
- **Mount conflicts**: ZFS handles mount point conflicts automatically
- **Permission issues**: Proper error reporting with context

**Identity Management Error Handling**:
```bash
# Recovery mechanism
if [[ ! -f "/tmp/os-release" ]]; then
    warn "Host os-release file not found in /tmp/os-release"
    if [[ -f "/etc/os-release.backup" ]] || [[ -L "/etc/os-release.backup" ]]; then
        mv "/etc/os-release.backup" "/etc/os-release"
    fi
fi
```

**Error Recovery Features**:
- **Missing host file**: Restores original identity if host file unavailable
- **Backup restoration**: Handles both file and symlink backups
- **Graceful degradation**: Installation continues with original identity

### Integration Specifications

**GRUB Configuration Integration**:
- **Timing**: Identity replacement occurs immediately before `update-grub`
- **Purpose**: Ensures GRUB configuration uses host system identity
- **Compatibility**: Works with GRUB's os-release detection mechanisms

**ZFS Integration Points**:
- **Pool import**: Manual mounting after pool import operations
- **Property management**: Mount commands after property modifications
- **State synchronization**: Ensures consistent dataset mount states

### Performance Specifications

**Mount Operation Efficiency**:
- **Conditional mounting**: Only mounts when necessary
- **State checking**: Verifies dataset existence before mount attempts
- **Batch operations**: Groups related ZFS operations together

**File Operation Efficiency**:
- **Single copy**: Host os-release copied once in Stage 1
- **Atomic replacement**: Uses `mv` for atomic file replacement
- **Minimal I/O**: Reduces file system operations through staging## Fin
al Production Specifications

### Debug System Specifications

**Debug Flow Coverage**:
| Stage | Debug Feature | Trigger | Purpose |
|-------|---------------|---------|---------|
| Stage 1 | Pre-chroot pause | `DEBUG=true` | System inspection before chroot |
| Stage 2 | Pre-exit pause | `DEBUG=true` | Chroot environment inspection |
| Stage 3 | Verbose output | `DEBUG=true` | Detailed cleanup operations |

**Debug Activation Methods**:
```bash
# Configuration file
DEBUG="true"  # In ubuntu-config.sh

# Command line parameter
./ubuntu-stage1.sh -D

# Parameter propagation
Stage1 -D → Stage2 -D → Stage3 -D
```

### Enhanced Configuration Specifications

**Partition Size Updates**:
```bash
EFI_SIZE="1G"     # Increased from 512M
BOOT_SIZE="4G"    # Increased from 1G
```

**User Account Configuration**:
```bash
USERS=(
    "localadmin:adm,cdrom,dip,lpadmin,lxd,plugdev,sambashare,sudo:Golden Soda Pop!"
)
```

**Configuration Benefits**:
- **EFI compatibility**: Space for multiple bootloaders and UEFI applications
- **Boot capacity**: Room for multiple kernels and large initramfs files
- **Administrative access**: Full system administration capabilities

### Advanced Cleanup Specifications

**Pre-Cleanup Validation Checks**:
```bash
# Open file detection
lsof +D "$INSTALL_ROOT" 2>/dev/null

# Process working directory check  
lsof +D "$INSTALL_ROOT" -a -d cwd 2>/dev/null

# Active swap detection
swapon --show --noheadings 2>/dev/null

# ZFS pool status check
zpool status | grep -E "(DEGRADED|FAULTED|OFFLINE|UNAVAIL)"
```

**Unmounting Pipeline Specifications**:
```bash
# Non-ZFS filesystem unmounting
mount | grep -v zfs | awk '$3 ~ "^'$INSTALL_ROOT'" {print $3}' | sort | tac | \
    xargs -i{} umount $umount_flags {}

# ZFS legacy dataset unmounting
zfs list -H -o name,mountpoint | awk '$2 == "legacy" {print $1}' | \
    xargs -r -i{} sh -c 'mount | grep "^{} " | awk "{print \$3}"' | sort | tac | \
    xargs -r -i{} umount $umount_flags {}
```

**Pipeline Components**:
1. **Filter**: Select appropriate filesystems/datasets
2. **Extract**: Get mount points or dataset names
3. **Sort**: Alphabetical ordering for consistency
4. **Reverse**: `tac` for dependency-safe unmounting order
5. **Execute**: Conditional verbose unmounting

### Error Handling Specifications

**Strict Error Detection**:
- **No lazy unmount**: Removed `-l` flags to detect busy filesystems
- **Explicit failures**: Commands fail visibly instead of silently continuing
- **User notification**: Clear error messages with resolution guidance

**Validation Response Matrix**:
| Issue Type | Detection Method | User Response | Script Behavior |
|------------|------------------|---------------|-----------------|
| Open files | `lsof +D` | Continue/Abort | Pause for decision |
| Process locks | `lsof -d cwd` | Continue/Abort | Pause for decision |
| Active swap | `swapon --show` | Warning only | Continue with warning |
| ZFS problems | `zpool status` | Continue/Abort | Pause for decision |

### GRUB Installation Specifications

**Optimized Installation Timing**:
```
Stage 2 Execution Order:
1. System configuration (locale, timezone, packages)
2. User account creation
3. GRUB installation (install_grub)
4. GRUB configuration (update-grub)
5. Final system preparation
```

**Timing Benefits**:
- **Complete system visibility**: GRUB sees all installed components
- **Proper dependencies**: All packages available for bootloader configuration
- **User integration**: Boot menu reflects all created user accounts
- **Final state**: Bootloader configured for production system state## Z
FS Cache Management and Cleanup Specifications

### ZFS Cache File Management Specifications

**Cache File Lifecycle**:
```bash
# Stage 1: Pool creation/import
zpool create ... $POOL_NAME $ROOT_PARTITION
zpool set cachefile=/etc/zfs/zpool.cache $POOL_NAME

# Stage 3: Pre-export cleanup
zpool set cachefile= $POOL_NAME
zpool export -a
```

**Cache File Application Matrix**:
| Pool Scenario | Stage 1 Action | Cache File Set | Purpose |
|---------------|----------------|----------------|---------|
| Auto mode (new) | Create pool | Yes | Enable auto-import |
| Manual mode (new) | Create pool | Yes | Enable auto-import |
| Manual mode (reused) | Import/altroot | Yes | Enable auto-import |

**Cache File Benefits**:
- **Automatic import**: ZFS services can find and import pools on boot
- **System integration**: Proper integration with systemd ZFS services
- **Boot reliability**: Ensures pools are available during system startup
- **Configuration persistence**: Pool settings preserved across reboots

### Advanced Cleanup Validation Specifications

**Pre-Cleanup Check Matrix**:
| Check Type | Command | Purpose | Response |
|------------|---------|---------|----------|
| Open files | `lsof +D "$INSTALL_ROOT"` | Detect file handles | Pause with guidance |
| Working directories | `lsof +D "$INSTALL_ROOT" -a -d cwd` | Detect process locks | Pause with guidance |
| Active swap | `swapon --show --noheadings` | Detect swap conflicts | Warning only |
| ZFS pool status | `zpool status \| grep -E "..."` | Detect pool issues | Pause with guidance |

**Validation Response Specifications**:
```bash
# Issue detection and user interaction
if [[ "$issues_found" == "true" ]]; then
    error "Potential unmount/export issues detected:"
    echo -e "$issue_details"
    warn "Press Enter to continue anyway, or Ctrl+C to abort..."
    read -p ""
fi
```

### Unmounting Pipeline Specifications

**Consistent Pipeline Architecture**:
```
Input → Filter → Extract → Sort → Reverse → Execute
```

**Pipeline Implementation**:
```bash
# Non-ZFS mounts
mount | grep -v zfs | awk '$3 ~ "^'$INSTALL_ROOT'" {print $3}' | sort | tac | \
    xargs -i{} umount $umount_flags {}

# ZFS legacy datasets
zfs list -H -o name,mountpoint | awk '$2 == "legacy" {print $1}' | \
    xargs -r -i{} sh -c 'mount | grep "^{} " | awk "{print \$3}"' | sort | tac | \
    xargs -r -i{} umount $umount_flags {}
```

**Pipeline Component Specifications**:
| Component | Purpose | Benefit |
|-----------|---------|---------|
| `sort` | Alphabetical ordering | Consistent processing |
| `tac` | Reverse order | Dependency-safe unmounting |
| `$umount_flags` | Conditional verbosity | Debug-aware output |
| `-r` flag | Empty input handling | Prevents errors with no matches |

### Error Handling Specifications

**Strict Error Detection**:
- **No lazy unmount**: Removed `-l` flags from all umount commands
- **Explicit failures**: Commands fail immediately on busy filesystems
- **Verbose debugging**: `-v` flag when DEBUG=true for detailed output
- **User notification**: Clear error messages with resolution guidance

**Error Response Matrix**:
| Error Type | Detection | User Response | Script Behavior |
|------------|-----------|---------------|-----------------|
| Busy filesystem | umount failure | Manual intervention | Script exits |
| Open files | lsof detection | Continue/Abort | User choice |
| Process locks | lsof detection | Continue/Abort | User choice |
| ZFS pool issues | zpool status | Continue/Abort | User choice |

### Debug Integration Specifications

**Conditional Verbosity**:
```bash
# Debug-aware umount flags
local umount_flags=""
if [[ "${DEBUG:-false}" == "true" ]]; then
    umount_flags="-v"
fi
```

**Debug Output Benefits**:
- **Clean normal operation**: No verbose output during standard cleanup
- **Detailed debugging**: Comprehensive output when troubleshooting
- **Consistent behavior**: Same debug pattern across all operations
- **User control**: Debug output controlled by single DEBUG variable## Z
FS Cache Management and Cleanup Specifications

### ZFS Cache File Management Specifications

**Cache File Application Matrix**:
| Pool Scenario | Stage | Command | Purpose |
|---------------|-------|---------|---------|
| Auto mode new pool | Stage 1 | `zpool set cachefile=/etc/zfs/zpool.cache $POOL_NAME` | Enable ZFS service auto-import |
| Manual mode new pool | Stage 1 | `zpool set cachefile=/etc/zfs/zpool.cache $POOL_NAME` | Enable ZFS service auto-import |
| Reused existing pool | Stage 1 | `zpool set cachefile=/etc/zfs/zpool.cache $POOL_NAME` | Enable ZFS service auto-import |
| Pre-export cleanup | Stage 3 | `zpool set cachefile= $POOL_NAME` | Clear cache for clean handoff |

**Cache File Management Requirements**:
- **Universal coverage**: All pool creation and import scenarios must set cache file
- **System integration**: Cache file must be set to `/etc/zfs/zpool.cache` for ZFS service compatibility
- **Clean export**: Cache file must be cleared before pool export to installed system
- **Boot reliability**: Proper cache file ensures pools available during system startup

### Advanced Cleanup Validation Specifications

**Pre-Cleanup Check Matrix**:
| Check Type | Command | Detection Target | User Guidance |
|------------|---------|------------------|---------------|
| Open files | `lsof +D "$INSTALL_ROOT"` | Processes holding files open | Kill processes or close files |
| Working directories | `lsof +D "$INSTALL_ROOT" -a -d cwd` | Processes with CWD in install area | Change working directories |
| Active swap | `swapon --show --noheadings` | Swap on same disk | Disable swap if on target disk |
| ZFS pool status | `zpool status \| grep -E "(DEGRADED\|FAULTED\|OFFLINE\|UNAVAIL)"` | Pool health issues | Resolve ZFS pool problems |

**Validation Flow Requirements**:
1. **Issue detection**: All check types must be executed before cleanup
2. **User reporting**: Detected issues must be reported with specific details
3. **Resolution guidance**: Each issue type must provide actionable resolution steps
4. **User choice**: User must be able to continue despite warnings or abort to fix issues
5. **Comprehensive coverage**: All potential unmount/export failure sources must be checked

### Unmounting Pipeline Specifications

**Pipeline Component Details**:
| Component | Implementation | Purpose | Requirements |
|-----------|----------------|---------|--------------|
| Data source | `mount` or `zfs list -H -o name,mountpoint` | Provide mount information | Must include all relevant mounts |
| Filter | `grep -v zfs` or `awk '$2 == "legacy"'` | Select target mounts | Must exclude irrelevant entries |
| Extract | `awk '$3 ~ "^'$INSTALL_ROOT'" {print $3}'` | Extract mount points | Must handle path matching correctly |
| Sort | `sort` | Consistent ordering | Must provide alphabetical ordering |
| Reverse | `tac` | Dependency-safe order | Must unmount children before parents |
| Execute | `xargs -i{} umount $umount_flags {}` | Perform unmounting | Must respect debug verbosity settings |

**Unmounting Strategy Requirements**:
- **Non-ZFS filesystems**: Must use mount table parsing with install root path filtering
- **ZFS legacy datasets**: Must use ZFS dataset enumeration with legacy mountpoint filtering
- **Ordering consistency**: All unmounting must use sort-then-reverse pattern
- **Debug integration**: Verbose output must be conditional on DEBUG setting
- **Error visibility**: Unmount failures must be visible and not suppressed

### Error Handling Specifications

**Strict Error Detection Requirements**:
| Error Type | Previous Handling | New Handling | Specification |
|------------|------------------|--------------|---------------|
| Busy filesystem unmount | Lazy unmount with `-l` | Explicit failure without `-l` | Must detect and report busy filesystems |
| Command failures | Suppressed with `\|\| true` | Visible with error propagation | Must allow failures to propagate visibly |
| Debug output | Fixed verbosity level | Conditional on DEBUG setting | Must use `-v` only when DEBUG=true |
| Error messages | Generic error reporting | Specific guidance messages | Must provide actionable resolution steps |

**Error Handling Flow**:
1. **Immediate detection**: Errors must be detected at point of failure
2. **Clear reporting**: Error messages must identify specific problem and location
3. **User guidance**: Each error type must provide specific resolution steps
4. **Recovery support**: Users must be able to understand and resolve detected issues
5. **Debug support**: Additional detail must be available when DEBUG mode is enabled

### Debug Integration Specifications

**Conditional Verbosity Requirements**:
- **Umount operations**: Use `-v` flag only when `DEBUG=true`
- **Mount operations**: Use `-v` flag only when `DEBUG=true`
- **ZFS operations**: Provide additional output only when `DEBUG=true`
- **Pipeline operations**: Show intermediate results only when `DEBUG=true`

**Debug Mode Activation**:
- **Configuration file**: `DEBUG="true"` in ubuntu-config.sh
- **Command line**: `-D` parameter on any stage script
- **Parameter precedence**: Command line `-D` overrides configuration file setting
- **Stage propagation**: DEBUG setting must propagate from stage1 through stage3## Final
 Production Optimization Specifications

### Aggressive Process Management Specifications

**Process Detection and Elimination Matrix**:
| Process Type | Detection Method | Termination Sequence | Error Handling |
|--------------|------------------|---------------------|----------------|
| Open files | `lsof +D "$INSTALL_ROOT"` | TERM → 2s delay → KILL | `\|\| true` |
| Working directory | `lsof +D "$INSTALL_ROOT" -a -d cwd` | TERM → 2s delay → KILL | `\|\| true` |
| ZFS processes | `grep "$POOL_NAME" /proc/*/mounts` | TERM → 2s delay → KILL | `\|\| true` |
| All pool processes | `zpool list -H -o name` iteration | TERM → 2s delay → KILL | `\|\| true` |

**Process Management Requirements**:
- **Comprehensive detection**: Must identify all processes that could interfere with unmounting
- **Graceful termination**: Must attempt TERM signal before KILL signal
- **Timing control**: Must wait 2 seconds between TERM and KILL signals
- **Error tolerance**: Must continue operation even if process killing fails
- **ZFS awareness**: Must handle processes using ZFS mounts specifically

**Swap Management Specifications**:
- **Universal disable**: Must run `swapoff -a` to disable all active swap
- **Error tolerance**: Must use `|| true` to continue if swap disable fails
- **Timing**: Must occur after process cleanup but before filesystem sync

### Enhanced Package Management Specifications

**rsync Package Copying Requirements**:
| Parameter | Value | Purpose |
|-----------|-------|---------|
| Source | `/var/cache/apt/` | Host system apt cache |
| Target | `$INSTALL_ROOT/var/cache/apt/` | Installation target cache |
| Flags | `-a` (archive) | Preserve all attributes |
| Error handling | Default rsync behavior | Allow rsync to handle errors |

**Package Selection Specifications**:
```bash
ADDITIONAL_PACKAGES=(
    "ubuntu-minimal"     # Required: Base Ubuntu system
    "ubuntu-standard"    # Required: Standard Ubuntu packages  
    "ubuntu-desktop"     # Required: Standard desktop environment
    "ssh"               # Required: Remote access capability
    "hollywood"         # Optional: Development tools
    "sanoid"           # Optional: ZFS snapshot management
)
```

**ZFS Dataset Configuration Requirements**:
- **Minimal default**: Only home datasets active by default
- **User discretion**: All other datasets commented with clear instructions
- **Documentation**: Comprehensive comments explaining each dataset purpose
- **Flexibility**: Easy uncomment mechanism for additional datasets

### Advanced ZFS Cache Management Specifications

**ZFS List Cache Setup Requirements**:
| Step | Command | Purpose | Error Handling |
|------|---------|---------|----------------|
| Pool cache copy | `cp /etc/zfs/zpool.cache "$INSTALL_ROOT/etc/zfs/"` | Transfer pool cache | Continue on failure |
| Directory creation | `mkdir -p "/etc/zfs/zfs-list.cache" "$INSTALL_ROOT/etc/zfs/zfs-list.cache"` | Create cache directories | Must succeed |
| Cache initialization | `truncate -s 0 /etc/zfs/zfs-list.cache/$POOL_NAME` | Initialize empty cache | Continue on failure |
| Event generation | `env -i` with ZFS variables + cacher script | Generate cache content | Continue on failure |
| Path translation | `sed -E "s\|\t$INSTALL_ROOT/?\|\t/\|g"` | Fix mount paths | Continue on failure |
| Cleanup | `rm -f "/etc/zfs/zfs-list.cache/$POOL_NAME"` | Remove temp files | Continue on failure |

**ZFS Environment Variables**:
- `ZEVENT_POOL=$POOL_NAME`: Target pool name
- `ZED_ZEDLET_DIR=/etc/zfs/zed.d`: ZFS event daemon directory
- `ZEVENT_SUBCLASS=history_event`: Event type classification
- `ZFS=zfs`: ZFS command path
- `ZEVENT_HISTORY_INTERNAL_NAME=create`: Event name for cache generation

### Error Handling and Cleanup Specifications

**Graceful Error Handling Requirements**:
| Operation | Error Handling | Justification |
|-----------|---------------|---------------|
| Process killing | `\|\| true` | Non-critical cleanup operation |
| Swap disable | `\|\| true` | May not exist or already disabled |
| ZFS cache setup | Continue on failure | Non-essential for basic boot |
| Pool export | `\|\| true` | Cleanup operation, shouldn't block completion |
| Package cache clean | Default behavior | Should succeed but not critical |

**Cleanup Sequence Specifications**:
1. **Process elimination**: Must kill all interfering processes first
2. **Swap disable**: Must disable swap after process cleanup
3. **Filesystem sync**: Must sync after process cleanup, before unmounting
4. **Unmount operations**: Must unmount in dependency-safe order
5. **ZFS cache setup**: Must setup cache after unmounting, before export
6. **Pool export**: Must export pools last with error tolerance

**Timing Requirements**:
- **Process kill delay**: 2 seconds between TERM and KILL signals
- **Sync placement**: After all process cleanup, before unmounting
- **Cache setup timing**: After unmounting, before pool export
- **Error tolerance**: All non-critical operations must use `|| true`

### System Optimization Specifications

**Package Cache Cleanup Requirements**:
- **Command**: `apt clean` in stage 2 after debug pause
- **Timing**: Before exiting chroot environment
- **Purpose**: Remove unnecessary package cache from final installation
- **Error handling**: Allow default apt behavior

**Filesystem Synchronization Requirements**:
- **Command**: `sync` in check_unmount_readiness function
- **Timing**: After process cleanup, before unmounting
- **Purpose**: Ensure all pending writes flushed to disk
- **Error handling**: Must succeed (no error tolerance needed)## U
ltimate Cleanup Architecture and Pool State Management Specifications

### Two-Stage Cleanup Orchestration Specifications

**Cleanup Execution Order Requirements**:
| Stage | Function | Purpose | Conditional Actions |
|-------|----------|---------|-------------------|
| 1 | `validate_stage3_config()` | Configuration validation | Must succeed |
| 2 | `cleanup_mounts()` | Initial unmount and pool export | Must complete |
| 3 | `aggressive_cleanup()` | Process elimination | Track process kills |
| 4 | Conditional re-export | Pool re-export if needed | Only if processes killed |
| 5 | `verify_installation()` | Final verification | Must complete |

**Process Tracking Specifications**:
```bash
# Required process tracking implementation
local processes_killed=false

# Track across all cleanup operations
if [[ -n "$process_pids" ]]; then
    # Perform cleanup operations
    processes_killed=true
fi

# Return status requirements
if [[ "$processes_killed" == "true" ]]; then
    return 0  # Processes were killed - triggers re-export
else
    return 1  # No processes killed - no re-export needed
fi
```

**Conditional Re-export Requirements**:
- **Trigger condition**: Only when `aggressive_cleanup()` returns 0 (processes killed)
- **Command**: `zpool export -a || true` (with error tolerance)
- **Logging**: Must log reason for re-export operation
- **Error handling**: Must use `|| true` to prevent script failure

### Enhanced Process Safety Specifications

**Numeric PID Filtering Requirements**:
| PID Source | Filtering Pattern | Purpose | Error Handling |
|------------|------------------|---------|----------------|
| lsof open files | `grep '^[0-9]\+$'` | Validate numeric PIDs | `\|\| true` |
| lsof working directory | `grep '^[0-9]\+$'` | Validate numeric PIDs | `\|\| true` |
| /proc mounts root pool | `grep '^[0-9]\+$'` | Validate numeric PIDs | `\|\| true` |
| /proc mounts other pools | `grep '^[0-9]\+$'` | Validate numeric PIDs | `\|\| true` |

**Filtering Pattern Specifications**:
- **Pattern**: `^[0-9]\+$` (start of line, one or more digits, end of line)
- **Application**: Must be applied to all PID extraction operations
- **Purpose**: Prevent "invalid signal specification" errors
- **Error tolerance**: All filtering operations must use `|| true`

**Lazy Unmount Specifications**:
```bash
# Required umount flag configuration
local umount_flags="-l"                    # Base: lazy unmount
if [[ "${DEBUG:-false}" == "true" ]]; then
    umount_flags="-l -v"                   # Debug: lazy + verbose
fi
```

**Lazy Unmount Requirements**:
- **Base flags**: `-l` (lazy unmount) must be default
- **Debug flags**: `-l -v` (lazy + verbose) when DEBUG=true
- **Application**: Must be applied to all umount operations
- **Error handling**: Lazy unmounts should not fail, but use `|| true` for safety

### Advanced Debug Integration Specifications

**Debug Pause Requirements**:
| Debug Point | Trigger Condition | Message Format | User Action |
|-------------|------------------|----------------|-------------|
| Open files cleanup | After process killing | "DEBUG: Completed killing processes with open files. Press Enter to continue..." | Wait for Enter |
| Working directory cleanup | After process killing | "DEBUG: Completed killing processes with working directory. Press Enter to continue..." | Wait for Enter |
| Swap disable | After swap operations | "DEBUG: Completed disabling swap. Press Enter to continue..." | Wait for Enter |
| Root pool cleanup | After process killing | "DEBUG: Completed killing processes using root pool. Press Enter to continue..." | Wait for Enter |
| Other pools cleanup | After process killing | "DEBUG: Completed killing processes using other ZFS pools. Press Enter to continue..." | Wait for Enter |

**Debug Implementation Requirements**:
```bash
# Required debug pause implementation
if [[ "${DEBUG:-false}" == "true" ]]; then
    warn "DEBUG: [operation description]. Press Enter to continue..."
    read -p ""
fi
```

**Debug Integration Specifications**:
- **Activation**: Only when `DEBUG=true` (from config or command line)
- **Placement**: After each major cleanup operation completion
- **User control**: Must wait for Enter key press to continue
- **Abort capability**: User can use Ctrl+C to abort at any pause
- **Message format**: Must use `warn` function with consistent format

### Pool State Management Specifications

**Clean Pool Initialization Requirements**:
| Step | Command | Purpose | Error Handling |
|------|---------|---------|----------------|
| Export | `zpool export $POOL_NAME` | Remove pool from system | Must succeed |
| Re-import | `zpool import -f -R $INSTALL_ROOT $POOL_NAME` | Re-import with clean state | Must succeed |

**Pool State Management Implementation**:
```bash
# Required export/re-import sequence
log "Exporting and re-importing pool $POOL_NAME for clean state..."
zpool export $POOL_NAME
zpool import -f -R $INSTALL_ROOT $POOL_NAME
```

**Pool State Requirements**:
- **Timing**: Must occur after pool creation and cache file setting
- **Scope**: Must be applied to both auto mode and manual mode pool creation
- **Logging**: Must log the export/re-import operation
- **Error handling**: Both commands must succeed (no error tolerance)
- **Flags**: Re-import must use `-f` (force) and `-R $INSTALL_ROOT` (altroot)

### Function Architecture Specifications

**Function Naming Requirements**:
- **Old name**: `check_unmount_readiness()` (deprecated)
- **New name**: `aggressive_cleanup()` (required)
- **Rationale**: Name must reflect actual aggressive behavior
- **Consistency**: All function calls must use new name

**Function Behavior Specifications**:
- **Process elimination**: Must kill processes with open files and working directories
- **Swap management**: Must disable all active swap
- **ZFS process cleanup**: Must kill processes using ZFS pools
- **Filesystem sync**: Must sync filesystems before completion
- **Status tracking**: Must track and return process kill status

### System Reliability Specifications

**Zero-Failure Requirements**:
| Operation Type | Error Handling | Rationale |
|----------------|---------------|-----------|
| Process killing | `\|\| true` | Non-critical cleanup |
| Swap disable | `\|\| true` | May not exist |
| Pool export (conditional) | `\|\| true` | Cleanup operation |
| Numeric filtering | `\|\| true` | Input validation |
| Debug pauses | No error handling | User-controlled |

**Reliability Implementation Requirements**:
- **Critical path protection**: Core installation operations must not use `|| true`
- **Cleanup tolerance**: All cleanup operations must use `|| true`
- **User experience**: No manual intervention required for common scenarios
- **Completion guarantee**: Script must always reach successful completion state#
# Advanced System Configuration and SSH Management Specifications

### SSH Installation Control Specifications

**SSH Configuration Requirements**:
| Configuration Item | Default Value | Override Method | Validation |
|-------------------|---------------|-----------------|------------|
| `INSTALL_SSH` | `"true"` | `--ssh`/`--nossh` parameters | Required variable |
| SSH installation | Enabled | Command-line override | Conditional execution |
| Parameter conflict | Not allowed | Error detection | Early failure |
| Cross-stage flow | Required | Parameter propagation | Consistent behavior |

**SSH Parameter Specifications**:
```bash
# Required parameter validation pattern
local ssh_param_used="false"

# Conflict detection implementation
if [[ "$ssh_param_used" == "true" ]]; then
    echo "Error: --ssh and --nossh cannot be used together"
    exit 1
fi
```

**SSH Installation Conditional Logic**:
```bash
# Required conditional installation
if [[ "$INSTALL_SSH" == "true" ]]; then
    install_openssh
else
    log "Skipping SSH installation (INSTALL_SSH=false)"
fi
```

**SSH Control Requirements**:
- **Configuration control**: `INSTALL_SSH` variable must control default behavior
- **Override capability**: `--ssh`/`--nossh` parameters must override configuration
- **Conflict prevention**: Cannot use both SSH parameters simultaneously
- **Cross-stage consistency**: Same parameter handling in stage 1 and stage 2
- **Clear feedback**: Must log SSH installation decisions explicitly

### System Optimization Specifications

**Package Management Requirements**:
| Operation | Command | Timing | Purpose |
|-----------|---------|--------|---------|
| Distribution upgrade | `apt dist-upgrade -y` | Before SSH installation | Latest packages |
| Package minimization | `apt-mark minimize-manual` | After upgrade | Clean package state |
| Kernel installation | `apt install -y linux-generic linux-generic-hwe${hwe_suffix} zfs-initramfs` | Single command | Efficiency |
| Configuration | `dpkg-reconfigure -f noninteractive keyboard-configuration console-setup` | Single command | Efficiency |

**Optimization Requirements**:
- **System upgrade**: Must perform `apt dist-upgrade -y` before service installation
- **Package minimization**: Must run `apt-mark minimize-manual` after upgrade
- **Merged operations**: Related packages must be installed in single commands
- **Atomic operations**: Related configurations must be performed together
- **Error handling**: All operations must include appropriate error handling

### Parameter Propagation Specifications

**Cross-Stage Parameter Flow Requirements**:
| Stage | Parameter Source | Parameter Target | Validation Required |
|-------|-----------------|------------------|-------------------|
| Stage 1 | Command line | Internal variables | Conflict detection |
| Stage 1 → Stage 2 | Internal variables | Command construction | Parameter tracking |
| Stage 2 | Command line | Internal variables | Conflict detection |
| Stage 2 | Internal variables | Installation logic | Conditional execution |

**Parameter Propagation Implementation**:
```bash
# Stage 1: Command construction requirements
local stage2_cmd="/root/ubuntu-stage2.sh"
if [[ "$DEBUG" == "true" ]]; then
    stage2_cmd="$stage2_cmd -D"
fi
if [[ "$ssh_param_used" == "true" ]]; then
    stage2_cmd="$stage2_cmd $ssh_param_value"
fi
```

**Propagation Requirements**:
- **Parameter tracking**: Must track which parameters were used
- **Command construction**: Must build stage 2 command with appropriate parameters
- **Validation consistency**: Same conflict detection in both stages
- **Override persistence**: Command-line overrides must flow through entire installation

### Configuration Validation Specifications

**Enhanced Validation Requirements**:
```bash
# Required variable list with SSH control
local required_vars=(
    "HOSTNAME"
    "TIMEZONE"
    "LOCALE"
    "NETWORK_INTERFACE"
    "DEBOOTSTRAP_SUITE"
    "INSTALL_SSH"          # Must be included
    "RED" "GREEN" "YELLOW" "NC"
)
```

**Validation Implementation Requirements**:
- **SSH configuration**: `INSTALL_SSH` must be included in required variables
- **Complete validation**: All configuration variables must be validated
- **Early failure**: Missing configuration must be detected before system modification
- **Clear reporting**: Must provide specific identification of missing variables
- **Array validation**: Must validate configuration arrays (ADDITIONAL_PACKAGES, USERS, ZFS_DATASETS)

### Code Optimization Specifications

**Line Merging Requirements**:
| Original Pattern | Merged Pattern | Benefit |
|-----------------|----------------|---------|
| Multiple apt install | Single apt install with multiple packages | Reduced overhead |
| Multiple dpkg-reconfigure | Single dpkg-reconfigure with multiple packages | Efficiency |
| Multi-line process operations | Single-line with semicolon separation | Conciseness |

**Optimization Implementation Examples**:
```bash
# Required kernel installation merging
apt install -y linux-generic linux-generic-hwe${hwe_suffix} zfs-initramfs

# Required configuration merging
dpkg-reconfigure -f noninteractive keyboard-configuration console-setup

# Required process operation merging
echo "$pids" | xargs -r kill -TERM 2>/dev/null || true; sleep 2; echo "$pids" | xargs -r kill -KILL 2>/dev/null || true
```

**Optimization Requirements**:
- **Performance improvement**: Must reduce command execution overhead
- **Functionality preservation**: Merged operations must maintain identical functionality
- **Error handling**: Must preserve error handling in merged operations
- **Readability**: Merged operations must remain readable and maintainable
- **Consistency**: Similar operations must use consistent merging patterns## Fina
l System Optimizations and Framework Specifications

### Streamlined Debug Framework Specifications

**Debug Framework Requirements**:
| Component | Status | Behavior | Purpose |
|-----------|--------|----------|---------|
| debug_pause functions | Removed | No longer called | Eliminated interactive interruptions |
| debug_break functions | Available | Manual insertion only | Targeted debugging capability |
| Exit code 99 | Active | Cross-stage detection | Proper debug break handling |
| Enhanced error handling | Active | Stage 1 detection | Debug break propagation |

**Debug Break Implementation Requirements**:
```bash
debug_break() {
    if [[ "${DEBUG:-false}" == "true" ]]; then
        warn "DEBUG BREAK: $1"
        warn "Exiting with error status for debugging..."
        exit 99  # Special debug exit code
    fi
}
```

**Debug Framework Specifications**:
- **Automated execution**: Installation must run without interactive pauses
- **Manual debugging**: debug_break functions available for manual insertion
- **Cross-stage handling**: Stage 1 must detect debug breaks from stages 2 and 3
- **Exit code consistency**: Debug breaks must use exit code 99 across all stages

### Intelligent Cleanup Architecture Specifications

**Conditional Cleanup Requirements**:
| Cleanup Stage | Success Condition | Failure Action | Re-export Required |
|---------------|------------------|----------------|-------------------|
| Initial cleanup | zpool export succeeds | Skip aggressive cleanup | No |
| Initial cleanup | zpool export fails | Run aggressive cleanup | Yes |
| Aggressive cleanup | Processes killed | Re-export pools | Yes |
| Aggressive cleanup | No processes killed | Retry export | Yes |

**Cleanup Function Return Codes**:
```bash
# cleanup_mounts return specifications
if zpool export -a; then
    return 0  # Export succeeded
else
    return 1  # Export failed
fi

# aggressive_cleanup return specifications
if [[ "$processes_killed" == "true" ]]; then
    return 0  # Processes were killed
else
    return 1  # No processes were killed
fi
```

**Enhanced Unmount Specifications**:
- **Recursive unmounting**: Must use `-R` flag for complete nested mount cleanup
- **Lazy unmounting**: Must use `-l` flag to prevent "device busy" errors
- **Debug verbosity**: Must use `-v` flag only when DEBUG=true
- **Flag combination**: `umount_flags="-l -R"` or `umount_flags="-l -R -v"`

### Advanced Package Management Specifications

**Package Configuration Requirements**:
```bash
# Default minimal configuration
ADDITIONAL_PACKAGES=(
    # "ubuntu-minimal"
    # "ubuntu-standard"
    # "ubuntu-desktop"
    # "hollywood"
    # "sanoid"
)
```

**Package Validation Specifications**:
```bash
# Required validation implementation
if ! declare -p ADDITIONAL_PACKAGES &>/dev/null; then
    missing_vars+=("ADDITIONAL_PACKAGES")
fi
```

**Package Management Requirements**:
- **Minimal defaults**: All additional packages must be commented out by default
- **SSH separation**: SSH installation controlled via INSTALL_SSH, not ADDITIONAL_PACKAGES
- **Validation intelligence**: Must distinguish missing variables from empty arrays
- **Conditional installation**: Must skip installation when array is empty
- **Non-interactive operations**: All package commands must use -y flag

### Flexible Bind Mount System Specifications

**Bind Mount Function Requirements**:
```bash
bind_mount() {
    local source_dir="$1"      # Required: Source directory path
    local target_dir="$2"      # Required: Target directory path
    local recursive="${3:-false}"  # Optional: Recursive behavior (default false)
    
    # Required basic operations
    mkdir -p "$target_dir"
    mount -v --bind "$source_dir" "$target_dir"
    mount -v --make-private "$target_dir"
    
    # Conditional recursive operations
    if [[ "$recursive" == "true" ]]; then
        # Submount discovery and mounting
    fi
}
```

**Recursive Mount Discovery Specifications**:
| Step | Command | Purpose | Requirements |
|------|---------|---------|--------------|
| Discovery | `mount \| awk -v src="$source_dir" '$3 ~ "^" src "/" {print $3}' \| sort` | Find submounts | Must find all mounted subdirectories |
| Path calculation | `relative_path="${subdir#$source_dir}"` | Calculate relative paths | Must preserve directory structure |
| Target creation | `mkdir -p "$target_subdir"` | Create target directories | Must create all necessary directories |
| Submount binding | `mount -v --bind "$subdir" "$target_subdir"` | Bind mount submount | Must bind each submount individually |
| Privacy setting | `mount -v --make-private "$target_subdir"` | Set mount privacy | Must make each submount private |

**Chroot Setup Requirements**:
```bash
# Required chroot filesystem mounts
bind_mount "/dev" "$INSTALL_ROOT/dev" true
bind_mount "/proc" "$INSTALL_ROOT/proc" true
bind_mount "/sys" "$INSTALL_ROOT/sys" true
```

**Bind Mount Specifications**:
- **Flexible recursion**: Must support both recursive and non-recursive mounting
- **Discovery-based**: Must automatically find all mounted subdirectories
- **Individual privacy**: Each mount must get individual --make-private setting
- **Verbose output**: Must use -v flag for all mount operations
- **Directory creation**: Must create target directories before mounting
- **Sorted processing**: Must process submounts in sorted order for consistency

### Configuration Architecture Specifications

**Configuration Organization Requirements**:
- **Contextual placement**: Important notes must be placed adjacent to related configuration
- **Logical grouping**: Related configuration items must be grouped together
- **Clear separation**: Configuration sections must have appropriate spacing
- **User guidance**: Constraints and requirements positioned where most relevant

**Enhanced Validation Requirements**:
- **Variable existence**: Must use `declare -p` to check variable declaration
- **Content flexibility**: Must allow empty arrays while catching missing declarations
- **Error specificity**: Must provide specific error messages for different issues
- **User guidance**: Must distinguish between missing variables and empty arrays

**File Structure Requirements**:
- **Proper formatting**: Configuration files must have appropriate line spacing
- **Logical flow**: Configuration items ordered by usage and dependency
- **Clear documentation**: Each configuration section must have clear explanations
- **Consistent style**: All configuration files must follow consistent formatting