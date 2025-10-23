# Security Policy

**Copyright © 2025 Michael C. Tinsay**  
Licensed under the GNU General Public License v3.0

> **🤖 AI-Generated Security Policy**: This security policy was entirely created by AI without direct human security review. The AI-generated code in this project has not undergone professional security auditing.

## Supported Versions

We provide security updates for the following versions:

| Version | Supported          |
| ------- | ------------------ |
| 0.1.x   | :white_check_mark: |

## Reporting a Vulnerability

We take security vulnerabilities seriously. If you discover a security vulnerability in the ZFS Root Installation Scripts, please report it responsibly.

### How to Report

1. **Do NOT create a public GitHub issue** for security vulnerabilities
2. **Email security details** to the maintainers (contact information in README)
3. **Include detailed information**:
   - Description of the vulnerability
   - Steps to reproduce the issue
   - Potential impact assessment
   - Suggested fix if you have one

### What to Expect

**Expect nothing.** This is a personal learning experiment with no commitment to maintenance, support, or security updates. While you can report vulnerabilities, there is no guarantee of response, acknowledgment, or fixes.

## Security Considerations

### Installation Security

This project involves system-level operations that require root privileges. Please be aware of the following security considerations:

**Note**: This is version 0.1 - an initial release and learning experiment. The author has no plans for long-term maintenance and will not be responsive to issues, support requests, or pull requests. Use at your own risk and discretion.

#### Root Privileges Required
- The installation scripts must run as root to perform disk partitioning, filesystem creation, and system configuration
- Always review the scripts before running them with root privileges
- Use in isolated environments (VMs) for testing before production use

#### Disk Operations
- The scripts perform destructive disk operations in auto mode
- Always backup important data before running the installation
- Verify disk device paths in configuration before execution
- Use manual mode for existing systems with data to preserve

#### Network Operations
- Scripts download packages from Ubuntu repositories
- Ensure you trust your network connection and DNS resolution
- Consider using local mirrors in security-sensitive environments
- Review package lists before installation

#### SSH Configuration
- SSH installation is optional and can be disabled for security
- Default SSH configuration follows Ubuntu security best practices
- Consider disabling SSH for air-gapped or high-security environments
- Review SSH configuration after installation

### Configuration Security

#### Sensitive Information
- User passwords in configuration files are stored in plain text
- Protect configuration files with appropriate file permissions
- Consider using environment variables for sensitive data
- Remove or secure configuration files after installation

#### File Permissions
- Scripts create files and directories with secure permissions
- Review created user accounts and their group memberships
- Verify filesystem permissions after installation
- Check ZFS dataset permissions and inheritance

### Runtime Security

#### Chroot Environment
- Scripts set up a secure chroot environment for installation
- Bind mounts are properly isolated with private namespaces
- Temporary files are cleaned up after installation
- No persistent backdoors or unauthorized access methods

#### Process Management
- Aggressive cleanup may terminate processes during installation
- Process termination uses standard signals (TERM, then KILL)
- No privilege escalation beyond the required root access
- Clean process state after installation completion

## Security Best Practices

### Before Installation
1. **Review all scripts** and configuration files
2. **Test in virtual machines** before production use
3. **Backup important data** before running installation
4. **Verify checksums** of downloaded scripts if available
5. **Use trusted networks** for package downloads

### During Installation
1. **Monitor the installation process** for unexpected behavior
2. **Review log output** for any security-related messages
3. **Verify network connections** are to expected repositories
4. **Check process behavior** if using debug mode

### After Installation
1. **Review created user accounts** and their permissions
2. **Check SSH configuration** and access controls
3. **Verify filesystem permissions** and ownership
4. **Update system packages** to latest security patches
5. **Remove installation scripts** and configuration files
6. **Review system logs** for any security-related events

## Threat Model

### In Scope
- Code execution vulnerabilities in the installation scripts
- Privilege escalation beyond intended root access
- Information disclosure through log files or temporary files
- Unauthorized network connections or data exfiltration
- Persistent backdoors or unauthorized access methods

### Out of Scope
- Vulnerabilities in Ubuntu packages or ZFS itself
- Physical security of the installation environment
- Social engineering attacks against users
- Vulnerabilities in the underlying hardware or firmware
- Network security beyond the installation process

## Security Updates

Any security updates may be included in the next release, whenever it may be, and potentially never at all. This is a personal learning experiment with no commitment to ongoing maintenance or security patching.

## Acknowledgments

We appreciate the security research community's efforts to improve the security of open source projects. Security researchers who responsibly disclose vulnerabilities will be acknowledged in our security advisories and project documentation.