---
title: Linux 磁盘管理相关命令
date: 2019-04-01 13:45:32
tags: 
    - linux
    - 磁盘
---

**1. df命令**

```shell
# df命令用于显示磁盘分区上的可使用的磁盘空间。默认显示单位为KB。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

Usage: df [OPTION]... [FILE]...
Show information about the file system on which each FILE resides,
or all file systems by default.

Mandatory arguments to long options are mandatory for short options too.
  -a, --all             include dummy file systems
  -B, --block-size=SIZE  scale sizes by SIZE before printing them; e.g.,
                           '-BM' prints sizes in units of 1,048,576 bytes;
                           see SIZE format below
      --direct          show statistics for a file instead of mount point
      --total           produce a grand total
  -h, --human-readable  print sizes in human readable format (e.g., 1K 234M 2G)
  -H, --si              likewise, but use powers of 1000 not 1024
  -i, --inodes          list inode information instead of block usage
  -k                    like --block-size=1K
  -l, --local           limit listing to local file systems
      --no-sync         do not invoke sync before getting usage info (default)
      --output[=FIELD_LIST]  use the output format defined by FIELD_LIST,
                               or print all fields if FIELD_LIST is omitted.
  -P, --portability     use the POSIX output format
      --sync            invoke sync before getting usage info
  -t, --type=TYPE       limit listing to file systems of type TYPE
  -T, --print-type      print file system type
  -x, --exclude-type=TYPE   limit listing to file systems not of type TYPE
  -v                    (ignored)
      --help     display this help and exit
      --version  output version information and exit

Display values are in units of the first available SIZE from --block-size,
and the DF_BLOCK_SIZE, BLOCK_SIZE and BLOCKSIZE environment variables.
Otherwise, units default to 1024 bytes (or 512 if POSIXLY_CORRECT is set).

SIZE is an integer and optional unit (example: 10M is 10*1024*1024).  Units
are K, M, G, T, P, E, Z, Y (powers of 1024) or KB, MB, ... (powers of 1000).

FIELD_LIST is a comma-separated list of columns to be included.  Valid
field names are: 'source', 'fstype', 'itotal', 'iused', 'iavail', 'ipcent',
'size', 'used', 'avail', 'pcent', 'file' and 'target' (see info page).

GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
For complete documentation, run: info coreutils 'df invocation'
```

**2. fdisk命令**

```shell
# fdisk命令用于观察硬盘实体使用情况，也可对硬盘分区。它采用传统的问答式界面，而非类似DOS fdisk的cfdisk互动式操作界面，因此在使用上较为不便，但功能却丝毫不打折扣。

Usage:
 fdisk [options] <disk>    change partition table
 fdisk [options] -l <disk> list partition table(s)
 fdisk -s <partition>      give partition size(s) in blocks

Options:
 -b <size>             sector size (512, 1024, 2048 or 4096)
 -c[=<mode>]           compatible mode: 'dos' or 'nondos' (default)
 -h                    print this help text
 -u[=<unit>]           display units: 'cylinders' or 'sectors' (default)
 -v                    print program version
 -C <number>           specify the number of cylinders
 -H <number>           specify the number of heads
 -S <number>           specify the number of sectors per track

```

**3. lsblk命令**

```shell
# lsblk命令用于列出所有可用块设备的信息，而且还能显示他们之间的依赖关系，但是它不会列出RAM盘的信息。块设备有硬盘，闪存盘，cd-ROM等等。lsblk命令包含在util-linux-ng包中，现在该包改名为util-linux。这个包带了几个其它工具，如dmesg。

Usage:
 lsblk [options] [<device> ...]

Options:
 -a, --all            print all devices
 -b, --bytes          print SIZE in bytes rather than in human readable format
 -d, --nodeps         don't print slaves or holders
 -D, --discard        print discard capabilities
 -e, --exclude <list> exclude devices by major number (default: RAM disks)
 -I, --include <list> show only devices with specified major numbers
 -f, --fs             output info about filesystems
 -h, --help           usage information (this)
 -i, --ascii          use ascii characters only
 -m, --perms          output info about permissions
 -l, --list           use list format output
 -n, --noheadings     don't print headings
 -o, --output <list>  output columns
 -p, --paths          print complate device path
 -P, --pairs          use key="value" output format
 -r, --raw            use raw output format
 -s, --inverse        inverse dependencies
 -t, --topology       output info about topology
 -S, --scsi           output info about SCSI devices

 -h, --help     display this help and exit
 -V, --version  output version information and exit

Available columns (for --output):
        NAME  device name
       KNAME  internal kernel device name
     MAJ:MIN  major:minor device number
      FSTYPE  filesystem type
  MOUNTPOINT  where the device is mounted
       LABEL  filesystem LABEL
        UUID  filesystem UUID
   PARTLABEL  partition LABEL
    PARTUUID  partition UUID
          RA  read-ahead of the device
          RO  read-only device
          RM  removable device
       MODEL  device identifier
      SERIAL  disk serial number
        SIZE  size of the device
       STATE  state of the device
       OWNER  user name
       GROUP  group name
        MODE  device node permissions
   ALIGNMENT  alignment offset
      MIN-IO  minimum I/O size
      OPT-IO  optimal I/O size
     PHY-SEC  physical sector size
     LOG-SEC  logical sector size
        ROTA  rotational device
       SCHED  I/O scheduler name
     RQ-SIZE  request queue size
        TYPE  device type
    DISC-ALN  discard alignment offset
   DISC-GRAN  discard granularity
    DISC-MAX  discard max bytes
   DISC-ZERO  discard zeroes data
       WSAME  write same max bytes
         WWN  unique storage identifier
        RAND  adds randomness
      PKNAME  internal parent kernel device name
        HCTL  Host:Channel:Target:Lun for SCSI
        TRAN  device transport type
         REV  device revision
      VENDOR  device vendor

For more details see lsblk(8).

```

**4. blkid命令**

```shell
# 在Linux下可以使用blkid命令对查询设备上所采用文件系统类型进行查询。blkid主要用来对系统的块设备（包括交换分区）所使用的文件系统类型、LABEL、UUID等信息进行查询。要使用这个命令必须安装e2fsprogs软件包。

blkid from util-linux 2.23.2  (libblkid 2.23.0, 25-Apr-2013)
Usage:
 blkid -L <label> | -U <uuid>

 blkid [-c <file>] [-ghlLv] [-o <format>] [-s <tag>] 
       [-t <token>] [<dev> ...]

 blkid -p [-s <tag>] [-O <offset>] [-S <size>] 
       [-o <format>] <dev> ...

 blkid -i [-s <tag>] [-o <format>] <dev> ...

Options:
 -c <file>   read from <file> instead of reading from the default
               cache file (-c /dev/null means no cache)
 -d          don't encode non-printing characters
 -h          print this usage message and exit
 -g          garbage collect the blkid cache
 -o <format> output format; can be one of:
               value, device, export or full; (default: full)
 -k          list all known filesystems/RAIDs and exit
 -s <tag>    show specified tag(s) (default show all tags)
 -t <token>  find device with a specific token (NAME=value pair)
 -l          look up only first device with token specified by -t
 -L <label>  convert LABEL to device name
 -U <uuid>   convert UUID to device name
 -V          print version and exit
 <dev>       specify device(s) to probe (default: all devices)

Low-level probing options:
 -p          low-level superblocks probing (bypass cache)
 -i          gather information about I/O limits
 -S <size>   overwrite device size
 -O <offset> probe at the given offset
 -u <list>   filter by "usage" (e.g. -u filesystem,raid)
 -n <list>   filter by filesystem type (e.g. -n vfat,ext3)

```

**5. mkfs.xfs命令**

```shell
Usage: mkfs.xfs
/* blocksize */		[-b log=n|size=num]
/* metadata */		[-m crc=0|1,finobt=0|1,uuid=xxx]
/* data subvol */	[-d agcount=n,agsize=n,file,name=xxx,size=num,
			    (sunit=value,swidth=value|su=num,sw=num|noalign),
			    sectlog=n|sectsize=num
/* force overwrite */	[-f]
/* inode size */	[-i log=n|perblock=n|size=num,maxpct=n,attr=0|1|2,
			    projid32bit=0|1]
/* no discard */	[-K]
/* log subvol */	[-l agnum=n,internal,size=num,logdev=xxx,version=n
			    sunit=value|su=num,sectlog=n|sectsize=num,
			    lazy-count=0|1]
/* label */		[-L label (maximum 12 characters)]
/* naming */		[-n log=n|size=num,version=2|ci,ftype=0|1]
/* no-op info only */	[-N]
/* prototype file */	[-p fname]
/* quiet */		[-q]
/* realtime subvol */	[-r extsize=num,size=num,rtdev=xxx]
/* sectorsize */	[-s log=n|size=num]
/* version */		[-V]
			devicename
<devicename> is required unless -d name=xxx is given.
<num> is xxx (bytes), xxxs (sectors), xxxb (fs blocks), xxxk (xxx KiB),
      xxxm (xxx MiB), xxxg (xxx GiB), xxxt (xxx TiB) or xxxp (xxx PiB).
<value> is xxx (512 byte blocks).
```

**6. mkfs.ext4命令**

```shell
Usage: mkfs.ext4 [-c|-l filename] [-b block-size] [-C cluster-size]
	[-i bytes-per-inode] [-I inode-size] [-J journal-options]
	[-G flex-group-size] [-N number-of-inodes]
	[-m reserved-blocks-percentage] [-o creator-os]
	[-g blocks-per-group] [-L volume-label] [-M last-mounted-directory]
	[-O feature[,...]] [-r fs-revision] [-E extended-option[,...]]
	[-t fs-type] [-T usage-type ] [-U UUID] [-jnqvDFKSV] device [blocks-count]

```

**7. partprobe命令**

```shell
# partprobe命令用于重读分区表，当出现删除文件后，出现仍然占用空间。可以partprobe在不重启的情况下重读分区。

Usage: partprobe [OPTION] [DEVICE]...
Inform the operating system about partition table changes.

  -d, --dry-run    do not actually inform the operating system
  -s, --summary    print a summary of contents
  -h, --help       display this help and exit
  -v, --version    output version information and exit

When no DEVICE is given, probe all partitions.

Report bugs to <bug-parted@gnu.org>.
```

