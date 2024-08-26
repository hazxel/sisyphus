# Unix

Unix 哲学 KISS：keep it simple, stupid。一切看上去十分复杂的逻辑功能，都用简单到不可思议的方式实现，甚至有些时候看上去很愚蠢。但仔细推敲，人们将会赞叹其精巧的设计，或许这就是大智若愚。

除 Linux 外，其他 Unix 系统还有 Solaris, FreeBSD, OpenBSD, NetBSD, MacOS 等。各种 Linux 版本大都遵从一些通用标准，如 IEEE's POSIX (Portable Operating Systems based on Unix),  X/Open's CAE (Common Application Environment, 公共应用环境)等，但这些标准并不包括内核的实现。



# File



# Service

- SSH: default to 22 port, *sshd* stand for SSH daemon and is the server-side process for SSH, configuration in `/etc/ssh/sshd_config`



# Users and Groups

### user

Linux user information is in `/etc/passwd`

### group

linux group information is in `/etc/group`

### permission

In Linux, file permissions are represented numerically with a three-digit octal value. Each digit corresponds to a different category of users:

1. **Owner**: The permissions for the file's owner (user).
2. **Group**: The permissions for the group that the owner belongs to.
3. **Others**: The permissions for all other users.

Each digit in the octal representation is an octal (base-8) number, which translates to a 3-bit binary number. This binary number represents the permissions as follows:

- **Read (r)**: The permission to read the file. (1 in the binary representation)
- **Write (w)**: The permission to modify the file. (2 in the binary representation)
- **Execute (x)**: The permission to execute the file. (4 in the binary representation)
