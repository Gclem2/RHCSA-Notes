# Important Basic Commands

---

## File System Navigation

```bash
pwd                  # Print working directory
cd /path/to/dir      # Change directory
ls -l                # List files with details
ls -a                # List all files, including hidden
tree /path           # Show directory tree (install tree package if needed)
```

## Viewing Files

```bash
cat file.txt         # Display file contents
less file.txt        # Scrollable view of file contents
head file.txt        # First 10 lines
tail file.txt        # Last 10 lines
```

## File and Directory Info

```bash
stat file.txt        # Detailed file info (permissions, size, timestamps)
file file.txt        # File type
du -h /path          # Disk usage of directory/file
df -h                # Filesystem disk usage
```

## System Information

```bash
uname -a             # Kernel version and system info
hostnamectl          # Hostname and OS info
cat /proc/cpuinfo    # CPU details
cat /proc/meminfo    # Memory usage
uptime               # System uptime and load averages
```

## Viewing Documentation

```bash
man command          # Manual page for a command
info command         # Info page
less /usr/share/doc/package/README  # Package documentation
```

## Process and Job Info (Intro)

```bash
ps aux               # List running processes
top                  # Interactive process viewer
jobs                 # Background jobs in current shell
```

## Compression and Archiving

```bash
# Tar commands
tar -cf archive.tar /path/to/dir         # Create tar archive
tar -xf archive.tar                      # Extract tar archive

# Tar with gzip or bzip2 compression
tar -czf archive.tar.gz /path/to/dir     # Create gzip compressed archive
tar -xzf archive.tar.gz                  # Extract gzip archive

tar -cjf archive.tar.bz2 /path/to/dir   # Create bzip2 compressed archive
tar -xjf archive.tar.bz2                 # Extract bzip2 archive

# Standalone compression
gzip file.txt       # Compress a file with gzip
gunzip file.txt.gz  # Decompress gzip file
bzip2 file.txt      # Compress a file with bzip2
bunzip2 file.txt.bz2 # Decompress bzip2 file
```

