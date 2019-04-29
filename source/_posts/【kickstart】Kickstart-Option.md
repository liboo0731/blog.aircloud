---
title: 【kickstart】Kickstart Option
date: 2018-09-14 13:43:25
tags:
- kickstart
---

### [Kickstart Options](https://www.cnblogs.com/liboo/p/9646091.html)

- autopart  (optional)

  Automatically create partitions — 1 GB or more root (/) partition, a swap partition, and an appropriate boot partition for the architecture. One or more of the default partition sizes can be redefined with the part  directive.--encrypted  — Should all devices with support be encrypted by default? This is equivalent to checking the Encrypt checkbox on the initial partitioning screen.--passphrase=  — Provide a default system-wide passphrase for all encrypted devices.

- ignoredisk  (optional)

  Causes the installer to ignore the specified disks. This is useful if you use autopartition and want to be sure that some disks are ignored. For example, without ignoredisk, attempting to deploy on a SAN-cluster the kickstart would fail, as the installer detects passive paths to the SAN that return no partition table. The --only-use  option specifies that only the disks listed will be used during installion.The ignoredisk  option is also useful if you have multiple paths to your disks.The syntax is:ignoredisk --drives=*drive1,drive2*,...where *driveN* is one of sda, sdb,..., hda,... etc.--only-use  — specifies a list of disks for the installer to use. All other disks are ignored. For example, to use disk sda  during installation and ignore all other disks:ignoredisk --only-use=sda

- autostep  (optional)

  Similar to interactive  except it goes to the next screen for you. It is used mostly for debugging.--autoscreenshot  — Take a screenshot at every step during installation and copy the images over to /root/anaconda-screenshots  after installation is complete. This is most useful for documentation.

- auth  or authconfig  (required)

  Sets up the authentication options for the system. It is similar to the authconfig  command, which can be run after the install. By default, passwords are normally encrypted and are not shadowed.--enablemd5  — Use md5 encryption for user passwords.--enablenis  — Turns on NIS support. By default, --enablenis  uses whatever domain it finds on the network. A domain should almost always be set by hand with the --nisdomain=  option.--nisdomain=  — NIS domain name to use for NIS services.--nisserver=  — Server to use for NIS services (broadcasts by default).--useshadow  or --enableshadow  — Use shadow passwords.--enableldap  — Turns on LDAP support in /etc/nsswitch.conf, allowing your system to retrieve information about users (UIDs, home directories, shells, etc.) from an LDAP directory. To use this option, you must install the nss_ldap  package. You must also specify a server and a base DN (distinguished name) with --ldapserver=  and --ldapbasedn=.--enableldapauth  — Use LDAP as an authentication method. This enables the pam_ldap  module for authentication and changing passwords, using an LDAP directory. To use this option, you must have the nss_ldap  package installed. You must also specify a server and a base DN with --ldapserver=  and --ldapbasedn=.--ldapserver=  — If you specified either --enableldap  or --enableldapauth, use this option to specify the name of the LDAP server to use. This option is set in the /etc/ldap.conf  file.--ldapbasedn=  — If you specified either --enableldap  or --enableldapauth, use this option to specify the DN in your LDAP directory tree under which user information is stored. This option is set in the /etc/ldap.conf  file.--enableldaptls  — Use TLS (Transport Layer Security) lookups. This option allows LDAP to send encrypted usernames and passwords to an LDAP server before authentication.--enablekrb5  — Use Kerberos 5 for authenticating users. Kerberos itself does not know about home directories, UIDs, or shells. If you enable Kerberos, you must make users' accounts known to this workstation by enabling LDAP, NIS, or Hesiod or by using the /usr/sbin/useradd  command. If you use this option, you must have the pam_krb5  package installed.--krb5realm=  — The Kerberos 5 realm to which your workstation belongs.--krb5kdc=  — The KDC (or KDCs) that serve requests for the realm. If you have multiple KDCs in your realm, separate their names with commas (,).--krb5adminserver=  — The KDC in your realm that is also running kadmind. This server handles password changing and other administrative requests. This server must be run on the master KDC if you have more than one KDC.--enablehesiod  — Enable Hesiod support for looking up user home directories, UIDs, and shells. More information on setting up and using Hesiod on your network is in /usr/share/doc/glibc-2.x.x/README.hesiod, which is included in the glibc  package. Hesiod is an extension of DNS that uses DNS records to store information about users, groups, and various other items.--hesiodlhs  — The Hesiod LHS ("left-hand side") option, set in /etc/hesiod.conf. This option is used by the Hesiod library to determine the name to search DNS for when looking up information, similar to LDAP's use of a base DN.--hesiodrhs  — The Hesiod RHS ("right-hand side") option, set in /etc/hesiod.conf. This option is used by the Hesiod library to determine the name to search DNS for when looking up information, similar to LDAP's use of a base DN.NoteTo look up user information for "jim", the Hesiod library looks up *jim.passwd<LHS><RHS>*, which should resolve to a TXT record that looks like what his passwd entry would look like (jim:*:501:501:Jungle Jim:/home/jim:/bin/bash). For groups, the situation is identical, except *jim.group<LHS><RHS>* would be used.Looking up users and groups by number is handled by making "501.uid" a CNAME for "jim.passwd", and "501.gid" a CNAME for "jim.group". Note that the library does not place a period . in front of the LHS and RHS values when performing a search. Therefore the LHS and RHS values need to have a period placed in front of them in order if they require this.--enablesmbauth  — Enables authentication of users against an SMB server (typically a Samba or Windows server). SMB authentication support does not know about home directories, UIDs, or shells. If you enable SMB, you must make users' accounts known to the workstation by enabling LDAP, NIS, or Hesiod or by using the /usr/sbin/useradd  command to make their accounts known to the workstation. To use this option, you must have the pam_smb  package installed.--smbservers=  — The name of the server(s) to use for SMB authentication. To specify more than one server, separate the names with commas (,).--smbworkgroup=  — The name of the workgroup for the SMB servers.--enablecache  — Enables the nscd  service. The nscd  service caches information about users, groups, and various other types of information. Caching is especially helpful if you choose to distribute information about users and groups over your network using NIS, LDAP, or hesiod.--passalgo  — Enables SHA256 or SHA512 hashing for passphrases. Use --passalgo=sha256  or --passalgo=sha215  and remove the --enablemd5  if present.

- bootloader  (required)

  Specifies how the boot loader should be installed. This option is required for both installations and upgrades.--append=  — Specifies kernel parameters. To specify multiple parameters, separate them with spaces. For example:bootloader --location=mbr --append="hdd=ide-scsi ide=nodma"--driveorder  — Specify which drive is first in the BIOS boot order. For example:bootloader --driveorder=sda,hda--hvargs  — If using GRUB, specifies Xen hypervisor arguments. To specify multiple parameters, separate them with spaces. For example:bootloader --hvargs="dom0_mem=2G dom0_max_vcpus=4"--location=  — Specifies where the boot record is written. Valid values are the following: mbr  (the default), partition  (installs the boot loader on the first sector of the partition containing the kernel), or none  (do not install the boot loader).--password=  — If using GRUB, sets the GRUB boot loader password to the one specified with this option. This should be used to restrict access to the GRUB shell, where arbitrary kernel options can be passed.--md5pass=  — If using GRUB, similar to --password=  except the password should already be encrypted.--upgrade  — Upgrade the existing boot loader configuration, preserving the old entries. This option is only available for upgrades.

- clearpart  (optional)

  Removes partitions from the system, prior to creation of new partitions. By default, no partitions are removed.NoteIf the clearpart  command is used, then the --onpart  command cannot be used on a logical partition.

- cmdline  (optional)

  Perform the installation in a completely non-interactive command line mode. Any prompts for interaction halts the install. This mode is useful on IBM System z systems with the x3270 console.

- device  (optional)

  On most PCI systems, the installation program autoprobes for Ethernet and SCSI cards properly. On older systems and some PCI systems, however, kickstart needs a hint to find the proper devices. The device  command, which tells the installation program to install extra modules, is in this format:device *<type>* *<moduleName>* --opts=*<options>**<type>* — Replace with either scsi  or eth.*<moduleName>* — Replace with the name of the kernel module which should be installed.--opts=  — Mount options to use for mounting the NFS export. Any options that can be specified in /etc/fstab  for an NFS mount are allowed. The options are listed in the nfs(5)  man page. Multiple options are separated with a comma.

- driverdisk  (optional)

  Driver diskettes can be used during kickstart installations. You must copy the driver diskettes's contents to the root directory of a partition on the system's hard drive. Then you must use the driverdisk  command to tell the installation program where to look for the driver disk.driverdisk *<partition>* [--type=*<fstype>*]Alternatively, a network location can be specified for the driver diskette:driverdisk --source=ftp://path/to/dd.imgdriverdisk --source=http://path/to/dd.imgdriverdisk --source=nfs:host:/path/to/img*<partition>* — Partition containing the driver disk.--type=  — File system type (for example, vfat or ext2).

- firewall  (optional)

  This option corresponds to the Firewall Configuration screen in the installation program:firewall --enabled|--disabled [--trust=] *<device>* [--port=]--enabled  or --enable  — Reject incoming connections that are not in response to outbound requests, such as DNS replies or DHCP requests. If access to services running on this machine is needed, you can choose to allow specific services through the firewall.--disabled  or --disable  — Do not configure any iptables rules.--trust=  — Listing a device here, such as eth0, allows all traffic coming from that device to go through the firewall. To list more than one device, use --trust eth0 --trust eth1. Do NOT use a comma-separated format such as --trust eth0, eth1.*<incoming>* — Replace with one or more of the following to allow the specified services through the firewall.--ssh--telnet--smtp--http--ftp--port=  — You can specify that ports be allowed through the firewall using the port:protocol format. For example, to allow IMAP access through your firewall, specify imap:tcp. Numeric ports can also be specified explicitly; for example, to allow UDP packets on port 1234 through, specify 1234:udp. To specify multiple ports, separate them by commas.

- firstboot  (optional)

  Determine whether the Setup Agent starts the first time the system is booted. If enabled, the firstboot  package must be installed. If not specified, this option is disabled by default.--enable  or --enabled  — The Setup Agent is started the first time the system boots.--disable  or --disabled  — The Setup Agent is not started the first time the system boots.--reconfig  — Enable the Setup Agent to start at boot time in reconfiguration mode. This mode enables the language, mouse, keyboard, root password, security level, time zone, and networking configuration options in addition to the default ones.

- halt  (optional)

  Halt the system after the installation has successfully completed. This is similar to a manual installation, where anaconda displays a message and waits for the user to press a key before rebooting. During a kickstart installation, if no completion method is specified, this option is used as the default.The halt  option is roughly equivalent to the shutdown -h  command.For other completion methods, refer to the poweroff, reboot, and shutdown  kickstart options.

- graphical  (optional)

  Perform the kickstart installation in graphical mode. This is the default.

- install  (optional)

  Tells the system to install a fresh system rather than upgrade an existing system. This is the default mode. For installation, you must specify the type of installation from cdrom, harddrive, nfs, or url  (for FTP or HTTP installations). The install  command and the installation method command must be on separate lines.cdrom  — Install from the first CD-ROM drive on the system.harddrive  — Install from a Red Hat installation tree on a local drive, which must be either vfat or ext2.--biospart=BIOS partition to install from (such as 82).--partition=Partition to install from (such as sdb2).--dir=Directory containing the `variant`  directory of the installation tree.For example:harddrive --partition=hdb2 --dir=/tmp/install-treenfs  — Install from the NFS server specified.--server=Server from which to install (hostname or IP).--dir=Directory containing the `variant`  directory of the installation tree.--opts=Mount options to use for mounting the NFS export. (optional)For example:nfs --server=nfsserver.example.com --dir=/tmp/install-treeurl  — Install from an installation tree on a remote server via FTP or HTTP.For example:url --url http://*<server>*/*<dir>*or:url --url ftp://*<username>*:*<password>@<server>*/*<dir>*

- interactive  (optional)

  Uses the information provided in the kickstart file during the installation, but allow for inspection and modification of the values given. You are presented with each screen of the installation program with the values from the kickstart file. Either accept the values by clicking Next or change the values and click Next to continue. Refer to the autostep  command.

- iscsi  (optional)

  iscsi --ipaddr= [options].Specifies additional iSCSI storage to be attached during installation. If you use the *iscsi* parameter, you must also assign a name to the iSCSI node, using the *iscsiname* parameter. The *iscsiname* parameter must appear before the *iscsi* parameter in the kickstart file.We recommend that wherever possible you configure iSCSI storage in the system BIOS or firmware (iBFT for Intel systems) rather than use the *iscsi* parameter. Anaconda automatically detects and uses disks configured in BIOS or firmware and no special configuration is necessary in the kickstart file.If you must use the *iscsi* parameter, ensure that networking is activated at the beginning of the installation, and that the *iscsi* parameter appears in the kickstart file before you refer to iSCSI disks with parameters such as *clearpart* or *ignoredisk*.--port=  (mandatory) — the port number (typically, --port=3260)--user=  — the username required to authenticate with the target--password=  — the password that corresponds with the username specified for the target--reverse-user=  — the username required to authenticate with the initiator from a target that uses reverse CHAP authentication--reverse-password=  — the password that corresponds with the username specified for the initiator

- iscsiname  (optional)

  Assigns a name to an iSCSI node specified by the iscsi parameter. If you use the *iscsi* parameter in your kickstart file, this parameter is mandatory, and you must specify *iscsiname* in the kickstart file before you specify *iscsi*.

- key  (optional)

  Specify an installation key, which is needed to aid in package selection and identify your system for support purposes.--skip  — Skip entering a key. Usually if the key command is not given, anaconda will pause at this step to prompt for a key. This option allows automated installation to continue if you do not have a key or do not want to provide one.

- keyboard  (required)

  Sets system keyboard type. Here is the list of available keyboards on i386, Itanium, and Alpha machines:be-latin1, bg, br-abnt2, cf, cz-lat2, cz-us-qwertz, de, de-latin1,de-latin1-nodeadkeys, dk, dk-latin1, dvorak, es, et, fi, fi-latin1,fr, fr-latin0, fr-latin1, fr-pc, fr_CH, fr_CH-latin1, gr, hu, hu101,is-latin1, it, it-ibm, it2, jp106, la-latin1, mk-utf, no, no-latin1,pl, pt-latin1, ro_win, ru, ru-cp1251, ru-ms, ru1, ru2, ru_win,se-latin1, sg, sg-latin1, sk-qwerty, slovene, speakup, speakup-lt,sv-latin1, sg, sg-latin1, sk-querty, slovene, trq, ua, uk, us, us-acentosThe file /usr/lib/python2.2/site-packages/rhpl/keyboard_models.py  also contains this list and is part of the rhpl  package.

- lang  (required)

  Sets the language to use during installation and the default language to use on the installed system. For example, to set the language to English, the kickstart file should contain the following line:lang en_USThe file /usr/share/system-config-language/locale-list  provides a list of the valid language codes in the first column of each line and is part of the system-config-language  package.Certain languages (mainly Chinese, Japanese, Korean, and Indic languages) are not supported during text mode installation. If one of these languages is specified using the lang command, installation will continue in English though the running system will have the specified language by default.

- langsupport  (deprecated)

  The langsupport keyword is deprecated and its use will cause an error message to be printed to the screen and installation to halt. Instead of using the langsupport keyword, you should now list the support package groups for all languages you want supported in the %packages  section of your kickstart file. For instance, adding support for French means you should add the following to %packages:@french-support

- logvol  (optional)

  Create a logical volume for Logical Volume Management (LVM) with the syntax:logvol *<mntpoint>* --vgname=*<name>* --size=*<size>* --name=*<name>* *<options>*The options are as follows:--noformat  — Use an existing logical volume and do not format it.--useexisting  — Use an existing logical volume and reformat it.--fstype=  — Sets the file system type for the logical volume. Valid values are xfs, ext2, ext3, ext4, swap, vfat, and hfs.--fsoptions=  — Specifies a free form string of options to be used when mounting the filesystem. This string will be copied into the /etc/fstab  file of the installed system and should be enclosed in quotes.--bytes-per-inode=  — Specifies the size of inodes on the filesystem to be made on the logical volume. Not all filesystems support this option, so it is silently ignored for those cases.--grow=  — Tells the logical volume to grow to fill available space (if any), or up to the maximum size setting.--maxsize=  — The maximum size in megabytes when the logical volume is set to grow. Specify an integer value here, and do not append the number with MB.--recommended=  — Determine the size of the logical volume automatically.--percent=  — Specify the size of the logical volume as a percentage of available space in the volume group.Create the partition first, create the logical volume group, and then create the logical volume. For example:part pv.01 --size 3000volgroup myvg pv.01logvol / --vgname=myvg --size=2000 --name=rootvol

- logging  (optional)

  This command controls the error logging of anaconda during installation. It has no effect on the installed system.--host=  — Send logging information to the given remote host, which must be running a syslogd process configured to accept remote logging.--port=  — If the remote syslogd process uses a port other than the default, it may be specified with this option.--level=  — One of debug, info, warning, error, or critical.Specify the minimum level of messages that appear on tty3. All messages will still be sent to the log file regardless of this level, however.

- mediacheck  (optional)

  If given, this will force anaconda to run mediacheck on the installation media. This command requires that installs be attended, so it is disabled by default.

- monitor  (optional)

  If the monitor command is not given, anaconda will use X to automatically detect your monitor settings. Please try this before manually configuring your monitor.--hsync=  — Specifies the horizontal sync frequency of the monitor.--monitor=  — Use specified monitor; monitor name should be from the list of monitors in /usr/share/hwdata/MonitorsDB from the hwdata package. The list of monitors can also be found on the X Configuration screen of the Kickstart Configurator. This is ignored if --hsync or --vsync is provided. If no monitor information is provided, the installation program tries to probe for it automatically.--noprobe=  — Do not try to probe the monitor.--vsync=  — Specifies the vertical sync frequency of the monitor.

- mouse  (deprecated)

  The mouse keyword is deprecated.

- network  (optional)

  Configures network information for the system. If the kickstart installation does not require networking (in other words, it is not installed over NFS, HTTP, or FTP), networking is not configured for the system. If the installation does require networking and network information is not provided in the kickstart file, the installation program assumes that the installation should be done over eth0 via a dynamic IP address (BOOTP/DHCP), and configures the final, installed system to determine its IP address dynamically. The network  option configures networking information for kickstart installations via a network as well as for the installed system.--bootproto=  — One of dhcp, bootp, or static.It defaults to dhcp. bootp  and dhcp  are treated the same.The DHCP method uses a DHCP server system to obtain its networking configuration. As you might guess, the BOOTP method is similar, requiring a BOOTP server to supply the networking configuration. To direct a system to use DHCP:network --bootproto=dhcpTo direct a machine to use BOOTP to obtain its networking configuration, use the following line in the kickstart file:network --bootproto=bootpThe static method requires that you enter all the required networking information in the kickstart file. As the name implies, this information is static and is used during and after the installation. The line for static networking is more complex, as you must include all network configuration information on one line. You must specify the IP address, netmask, gateway, and nameserver.Note that although the presentation of this example on this page has broken the line, in a real kickstart file, you must include all this information on a single line with no break.network --bootproto=static --ip=10.0.2.15 --netmask=255.255.255.0--gateway=10.0.2.254 --nameserver=10.0.2.1If you use the static method, be aware of the following two restrictions:All static networking configuration information must be specified on *one* line; you cannot wrap lines using a backslash, for example.You can also configure multiple nameservers here. To do so, specify them as a comma-delimited list in the command line.Note that although the presentation of this example on this page has broken the line, in a real kickstart file, you must include all this information on a single line with no break.network --bootproto=static --ip=10.0.2.15 --netmask=255.255.255.0--gateway=10.0.2.254 --nameserver 192.168.2.1,192.168.3.1--device=  — Used to select a specific Ethernet device for installation. Note that using --device=  is not effective unless the kickstart file is a local file (such as ks=floppy), since the installation program configures the network to find the kickstart file. For example:network --bootproto=dhcp --device=eth0--ip=  — IP address for the machine to be installed.--gateway=  — Default gateway as an IP address.--nameserver=  — Primary nameserver, as an IP address.--nodns  — Do not configure any DNS server.--netmask=  — Netmask for the installed system.--hostname=  — Hostname for the installed system.--ethtool=  — Specifies additional low-level settings for the network device which will be passed to the ethtool program.--essid=  — The network ID for wireless networks.--wepkey=  — The encryption key for wireless networks.--onboot=  — Whether or not to enable the device at boot time.--dhcpclass=  — The DHCP class.--mtu=  — The MTU of the device.--noipv4  — Disable IPv4 on this device.--noipv6  — Disable IPv6 on this device.

- multipath  (optional)

  Specifies a multipath device in the format:multipath --name=mpath*X* --device=*device_name* --rule=*policy*For example:multipath --name=mpath0 --device=/dev/sdc --rule=failoverThe available options are:--name=  — the name for the multipath device, in the format mpath*X*, where *X* is an integer.--device=  — the block device connected as a multipath device.--rule=  — a multipath *policy*: failover, multibus, group_by_serial, group_by_prio, or group_by_node_name. Refer to the multipath manpage for a description of these policies.

- part  or partition  (required for installs, ignored for upgrades)

  Creates a partition on the system.If more than one Red Hat Enterprise Linux installation exists on the system on different partitions, the installation program prompts the user and asks which installation to upgrade.WarningAll partitions created are formatted as part of the installation process unless --noformat  and --onpart  are used.For a detailed example of part  in action, refer to [Section 31.4.1, "Advanced Partitioning Example"](http://docs.redhat.com/docs/en-US/Red_Hat_Enterprise_Linux/5/html/Installation_Guide/s1-kickstart2-options.html#s2-kickstart2-options-part-examples).*<mntpoint>* — The *<mntpoint>* is where the partition is mounted and must be of one of the following forms:/*<path>*For example, /, /usr, /home swapThe partition is used as swap space.To determine the size of the swap partition automatically, use the --recommended  option:swap --recommendedThe recommended maximum swap size for machines with less than 2GB of RAM is twice the amount of RAM. For machines with 2GB or more, this recommendation changes to 2GB plus the amount of RAM.raid.*<id>*The partition is used for software RAID (refer to raid).pv.*<id>*The partition is used for LVM (refer to logvol).--size=  — The minimum partition size in megabytes. Specify an integer value here such as 500. Do not append the number with MB.--grow  — Tells the partition to grow to fill available space (if any), or up to the maximum size setting.NoteIf you use --grow=  without setting --maxsize=  on a swap partition, Anaconda will limit the maximum size of the swap partition. For systems that have less than 2GB of physical memory, the imposed limit is twice the amount of physical memory. For systems with more than 2GB, the imposed limit is the size of physical memory plus 2GB.--maxsize=  — The maximum partition size in megabytes when the partition is set to grow. Specify an integer value here, and do not append the number with MB.--noformat  — Tells the installation program not to format the partition, for use with the --onpart  command.--onpart=  or --usepart=  — Put the partition on the *already existing* device. For example:partition /home --onpart=hda1puts /home  on /dev/hda1, which must already exist.--ondisk=  or --ondrive=  — Forces the partition to be created on a particular disk. For example, --ondisk=sdb  puts the partition on the second SCSI disk on the system.--asprimary  — Forces automatic allocation of the partition as a primary partition, or the partitioning fails.--type=  (replaced by fstype) — This option is no longer available. Use fstype.--fstype=  — Sets the file system type for the partition. Valid values are xfs, ext2, ext3, ext4, swap, vfat, and hfs.--start=  — Specifies the starting cylinder for the partition. It requires that a drive be specified with --ondisk=  or ondrive=. It also requires that the ending cylinder be specified with --end=  or the partition size be specified with --size=.--end=  — Specifies the ending cylinder for the partition. It requires that the starting cylinder be specified with --start=.--bytes-per-inode=  — Specifies the size of inodes on the filesystem to be made on the partition. Not all filesystems support this option, so it is silently ignored for those cases.--recommended  — Determine the size of the partition automatically.--onbiosdisk  — Forces the partition to be created on a particular disk as discovered by the BIOS.--encrypted  — Specifies that this partition should be encrypted.--passphrase=  — Specifies the passphrase to use when encrypting this partition. Without the above --encrypted  option, this option does nothing. If no passphrase is specified, the default system-wide one is used, or the installer will stop and prompt if there is no default.--fsoptions=  — Specifies a free form string of options to be used when mounting the filesystem. This string will be copied into the /etc/fstab  file of the installed system and should be enclosed in quotes.NoteIf partitioning fails for any reason, diagnostic messages appear on virtual console 3.

- poweroff  (optional)

  Shut down and power off the system after the installation has successfully completed. Normally during a manual installation, anaconda displays a message and waits for the user to press a key before rebooting. During a kickstart installation, if no completion method is specified, the halt  option is used as default.The poweroff  option is roughly equivalent to the shutdown -p  command.NoteThe poweroff  option is highly dependent on the system hardware in use. Specifically, certain hardware components such as the BIOS, APM (advanced power management), and ACPI (advanced configuration and power interface) must be able to interact with the system kernel. Contact your manufacturer for more information on you system's APM/ACPI abilities.For other completion methods, refer to the halt, reboot, and shutdown  kickstart options.

- raid  (optional)

  Assembles a software RAID device. This command is of the form:raid *<mntpoint>* --level=*<level>* --device=*<mddevice>* *<partitions\*>**<mntpoint>* — Location where the RAID file system is mounted. If it is /, the RAID level must be 1 unless a boot partition (/boot) is present. If a boot partition is present, the /boot  partition must be level 1 and the root (/) partition can be any of the available types. The *<partitions\*>* (which denotes that multiple partitions can be listed) lists the RAID identifiers to add to the RAID array.--level=  — RAID level to use (0, 1, 4, 5, 6, or 10).--device=  — Name of the RAID device to use (such as md0 or md1). RAID devices range from md0 to md15, and each may only be used once.--bytes-per-inode=  — Specifies the size of inodes on the filesystem to be made on the RAID device. Not all filesystems support this option, so it is silently ignored for those cases.--spares=  — Specifies the number of spare drives allocated for the RAID array. Spare drives are used to rebuild the array in case of drive failure.--fstype=  — Sets the file system type for the RAID array. Valid values are xfs, ext2, ext3, ext4, swap, vfat, and hfs.--fsoptions=  — Specifies a free form string of options to be used when mounting the filesystem. This string will be copied into the /etc/fstab file of the installed system and should be enclosed in quotes.--noformat  — Use an existing RAID device and do not format the RAID array.--useexisting  — Use an existing RAID device and reformat it.--encrypted  — Specifies that this RAID device should be encrypted.--passphrase=  — Specifies the passphrase to use when encrypting this RAID device. Without the above --encrypted  option, this option does nothing. If no passphrase is specified, the default system-wide one is used, or the installer will stop and prompt if there is no default.The following example shows how to create a RAID level 1 partition for /, and a RAID level 5 for /usr, assuming there are three SCSI disks on the system. It also creates three swap partitions, one on each drive.part raid.01 --size=60 --ondisk=sdapart raid.02 --size=60 --ondisk=sdbpart raid.03 --size=60 --ondisk=sdcpart swap --size=128 --ondisk=sdapart swap --size=128 --ondisk=sdbpart swap --size=128 --ondisk=sdcpart raid.11 --size=1 --grow --ondisk=sdapart raid.12 --size=1 --grow --ondisk=sdbpart raid.13 --size=1 --grow --ondisk=sdcraid / --level=1 --device=md0 raid.01 raid.02 raid.03raid /usr --level=5 --device=md1 raid.11 raid.12 raid.13For a detailed example of raid  in action, refer to [Section 31.4.1, "Advanced Partitioning Example"](http://docs.redhat.com/docs/en-US/Red_Hat_Enterprise_Linux/5/html/Installation_Guide/s1-kickstart2-options.html#s2-kickstart2-options-part-examples).

- reboot  (optional)

  Reboot after the installation is successfully completed (no arguments). Normally, kickstart displays a message and waits for the user to press a key before rebooting.The reboot  option is roughly equivalent to the shutdown -r  command.Specify reboot  to automate installation fully when installing in cmdline mode on System z.For other completion methods, refer to the halt, poweroff, and shutdown  kickstart options.The halt  option is the default completion method if no other methods are explicitly specified in the kickstart file.NoteUse of the reboot  option *may* result in an endless installation loop, depending on the installation media and method.

- repo  (optional)

  Configures additional yum repositories that may be used as sources for package installation. Multiple repo lines may be specified.repo --name=*<repoid>* [--baseurl=*<url>*| --mirrorlist=*<url>*]--name=  — The repo id. This option is required.--baseurl=  — The URL for the repository. The variables that may be used in yum repo config files are not supported here. You may use one of either this option or --mirrorlist, not both.--mirrorlist=  — The URL pointing at a list of mirrors for the repository. The variables that may be used in yum repo config files are not supported here. You may use one of either this option or --baseurl, not both.

- rootpw  (required)

  Sets the system's root password to the *<password>* argument.rootpw [--iscrypted] *<password>*--iscrypted  — If this is present, the password argument is assumed to already be encrypted.

- selinux  (optional)

  Sets the state of SELinux on the installed system. SELinux defaults to enforcing in anaconda.selinux [--disabled|--enforcing|--permissive]--enforcing  — Enables SELinux with the default targeted policy being enforced.

> #### Note
>
> > If the selinux  option is not present in the kickstart file, SELinux is enabled and set to --enforcing  by default.
> >
> > - --permissive  — Outputs warnings based on the SELinux policy, but does not actually enforce the policy.
> > - --disabled  — Disables SELinux completely on the system.

- services  (optional)

  Modifies the default set of services that will run under the default runlevel. The services listed in the disabled list will be disabled before the services listed in the enabled list are enabled.--disabled  — Disable the services given in the comma separated list.--enabled  — Enable the services given in the comma separated list.***Do not include spaces in the list of services***If you include spaces in the comma-separated list, kickstart will enable or disable only the services up to the first space. For example:services --disabled auditd, cups,smartd,nfslock will disable only the auditd service. To disable all four services, this entry should include no spaces between services:services --disabled auditd,cups,smartd,nfslock 

- shutdown  (optional)

  Shut down the system after the installation has successfully completed. During a kickstart installation, if no completion method is specified, the halt  option is used as default.The shutdown  option is roughly equivalent to the shutdown  command.For other completion methods, refer to the halt, poweroff, and reboot  kickstart options.

- skipx  (optional)

  If present, X is not configured on the installed system.

- text  (optional)

  Perform the kickstart installation in text mode. Kickstart installations are performed in graphical mode by default.

- timezone  (required)

  Sets the system time zone to *<timezone>* which may be any of the time zones listed by timeconfig.timezone [--utc] *<timezone>*--utc  — If present, the system assumes the hardware clock is set to UTC (Greenwich Mean) time.

- upgrade  (optional)

  Tells the system to upgrade an existing system rather than install a fresh system. You must specify one of cdrom, harddrive, nfs, or url  (for FTP and HTTP) as the location of the installation tree. Refer to install  for details.

- user  (optional)

  Creates a new user on the system.user --name=*<username>* [--groups=*<list>*] [--homedir=*<homedir>*] [--password=*<password>*] [--iscrypted] [--shell=*<shell>*] [--uid=*<uid>*]--name=  — Provides the name of the user. This option is required.--groups=  — In addition to the default group, a comma separated list of group names the user should belong to. The groups must exist before the user account is created.--homedir=  — The home directory for the user. If not provided, this defaults to /home/*<username>*.--password=  — The new user's password. If not provided, the account will be locked by default.--iscrypted=  — Is the password provided by --password already encrypted or not?--shell=  — The user's login shell. If not provided, this defaults to the system default.--uid=  — The user's UID. If not provided, this defaults to the next available non-system UID.

- vnc  (optional)

  Allows the graphical installation to be viewed remotely via VNC. This method is usually preferred over text mode, as there are some size and language limitations in text installs. With no options, this command will start a VNC server on the machine with no password and will print out the command that needs to be run to connect a remote machine.vnc [--host=*<hostname>*] [--port=*<port>*] [--password=*<password>*]--host=  — Instead of starting a VNC server on the install machine, connect to the VNC viewer process listening on the given hostname.--port=  — Provide a port that the remote VNC viewer process is listening on. If not provided, anaconda will use the VNC default.--password=  — Set a password which must be provided to connect to the VNC session. This is optional, but recommended.

- volgroup  (optional)

  Use to create a Logical Volume Management (LVM) group with the syntax:volgroup *<name>* *<partition>* *<options>*The options are as follows:--noformat  — Use an existing volume group and do not format it.--useexisting  — Use an existing volume group and reformat it.--pesize=  — Set the size of the physical extents.Create the partition first, create the logical volume group, and then create the logical volume. For example:part pv.01 --size 3000volgroup myvg pv.01logvol / --vgname=myvg --size=2000 --name=rootvolFor a detailed example of volgroup  in action, refer to [Section 31.4.1, "Advanced Partitioning Example"](http://docs.redhat.com/docs/en-US/Red_Hat_Enterprise_Linux/5/html/Installation_Guide/s1-kickstart2-options.html#s2-kickstart2-options-part-examples).

- xconfig  (optional)

  Configures the X Window System. If this option is not given, the user must configure X manually during the installation, if X was installed; this option should not be used if X is not installed on the final system.--driver  — Specify the X driver to use for the video hardware.--videoram=  — Specifies the amount of video RAM the video card has.--defaultdesktop=  — Specify either GNOME or KDE to set the default desktop (assumes that GNOME Desktop Environment and/or KDE Desktop Environment has been installed through %packages).--startxonboot  — Use a graphical login on the installed system.--resolution=  — Specify the default resolution for the X Window System on the installed system. Valid values are 640x480, 800x600, 1024x768, 1152x864, 1280x1024, 1400x1050, 1600x1200. Be sure to specify a resolution that is compatible with the video card and monitor.--depth=  — Specify the default color depth for the X Window System on the installed system. Valid values are 8, 16, 24, and 32. Be sure to specify a color depth that is compatible with the video card and monitor.

- %include  (optional)

  Use the %include */path/to/file*  command to include the contents of another file in the kickstart file as though the contents were at the location of the %include  command in the kickstart file.

### Advanced Partitioning Example

The following is a single, integrated example showing the clearpart, raid, part, volgroup, and logvol  kickstart options in action:

clearpart --drives=hda,hdc --initlabel

\# Raid 1 IDE config

part raid.11 --size 1000 --asprimary --ondrive=hda

part raid.12 --size 1000 --asprimary --ondrive=hda

part raid.13 --size 2000 --asprimary --ondrive=hda

part raid.14 --size 8000 --ondrive=hda

part raid.15 --size 1 --grow --ondrive=hda

part raid.21 --size 1000 --asprimary --ondrive=hdc

part raid.22 --size 1000 --asprimary --ondrive=hdc

part raid.23 --size 2000 --asprimary --ondrive=hdc

part raid.24 --size 8000 --ondrive=hdc

part raid.25 --size 1 --grow --ondrive=hdc



\# You can add --spares=x

raid / --fstype ext3 --device md0 --level=RAID1 raid.11 raid.21

raid /safe --fstype ext3 --device md1 --level=RAID1 raid.12 raid.22

raid swap --fstype swap --device md2 --level=RAID1 raid.13 raid.23

raid /usr --fstype ext3 --device md3 --level=RAID1 raid.14 raid.24

raid pv.01 --fstype ext3 --device md4 --level=RAID1 raid.15 raid.25



\# LVM configuration so that we can resize /var and /usr/local later

volgroup sysvg pv.01

logvol /var --vgname=sysvg --size=8000 --name=var

logvol /var/freespace --vgname=sysvg --size=8000 --name=freespacetouse

logvol /usr/local --vgname=sysvg --size=1 --grow --name=usrlocal

This advanced example implements LVM over RAID, as well as the ability to resize various directories for future growth.

You can use a kickstart file to install every available package by specifying @Everything  or simply *  in the %packages  section. Red Hat does not support this type of installation.

Moreover, using a kickstart file in this way will introduce package and file conflicts onto the installed system. Packages known to cause such problems are assigned to the @Conflicts  group. If you specify @Everything  in a kickstart file, be sure to exclude @Conflicts  or the installation will fail:

@Everything

-@Conflicts

Note that Red Hat does not support the use of @Everything  in a kickstart file, even if you exclude @Conflicts.

Use the %packages  command to begin a kickstart file section that lists the packages you would like to install (this is for installations only, as package selection during upgrades is not supported).

Packages can be specified by group or by individual package name, including with globs using the asterisk. The installation program defines several groups that contain related packages. Refer to the *variant*/repodata/comps-*.xml  file on the first Red Hat Enterprise Linux CD-ROM for a list of groups. Each group has an id, user visibility value, name, description, and package list. In the package list, the packages marked as mandatory are always installed if the group is selected, the packages marked default are selected by default if the group is selected, and the packages marked optional must be specifically selected even if the group is selected to be installed.

Available groups vary slightly between different variants of Red Hat Enterprise Linux 5, but include:

- Administration Tools
- Authoring and Publishing
- Development Libraries
- Development Tools
- DNS Name Server
- Eclipse
- Editors
- Engineering and Scientific
- FTP Server
- GNOME Desktop Environment
- GNOME Software Development
- Games and Entertainment
- Graphical Internet
- Graphics
- Java Development
- KDE (K Desktop Environment)
- KDE Software Development
- Legacy Network Server
- Legacy Software Development
- Legacy Software Support
- Mail Server
- Misc
- Multimedia
- MySQL Database
- Network Servers
- News Server
- Office/Productivity
- OpenFabrics Enterprise Distribution
- PostgreSQL Database
- Printing Support
- Server Configuration Tools
- Sound and Video
- System Tools
- Text-based Internet
- Web Server
- Windows File Server
- Windows PV Drivers
- X Software Development
- X Window System

In most cases, it is only necessary to list the desired groups and not individual packages. Note that the Core  and Base  groups are always selected by default, so it is not necessary to specify them in the %packages  section.

Here is an example %packages  selection:

%packages

@ X Window System

@ GNOME Desktop Environment

@ Graphical Internet

@ Sound and Video dhcp

As you can see, groups are specified, one to a line, starting with an @  symbol, a space, and then the full group name as given in the comps.xml  file. Groups can also be specified using the id for the group, such as gnome-desktop. Specify individual packages with no additional characters (the dhcp  line in the example above is an individual package).

You can also specify which packages not to install from the default package list:

-autofs

The following options are available for the %packages  option:

- --nobase 

  Do not install the @Base group. Use this option if you are trying to create a very small system.

- --resolvedeps 

  The --resolvedeps option has been deprecated. Dependencies are resolved automatically every time now.

- --ignoredeps 

  The --ignoredeps option has been deprecated. Dependencies are resolved automatically every time now.

- --ignoremissing 

  Ignore the missing packages and groups instead of halting the installation to ask if the installation should be aborted or continued. For example:%packages --ignoremissing

- interpreter */usr/bin/python* 

  Allows you to specify a different scripting language, such as Python. Replace */usr/bin/python* with the scripting language of your choice.

### Example

Here is an example %pre  section:

%pre

\#!/bin/sh

hds=""

mymedia=""

for file in /proc/ide/h* do

​    mymedia=`cat $file/media`

​    if [ $mymedia == "disk" ] ; then

​        hds="$hds `basename $file`"

​    fi

done

set $hds

numhd=`echo $#`

drive1=`echo $hds | cut -d' ' -f1`

drive2=`echo $hds | cut -d' ' -f2`

\#Write out partition scheme based on whether there are 1 or 2 hard drives

if [ $numhd == "2" ] ; then

​    \#2 drives

​    echo "#partitioning scheme generated in %pre for 2 drives" > /tmp/part-include

​    echo "clearpart --all" >> /tmp/part-include

​    echo "part /boot --fstype ext3 --size 75 --ondisk hda" >> /tmp/part-include

​    echo "part / --fstype ext3 --size 1 --grow --ondisk hda" >> /tmp/part-include

​    echo "part swap --recommended --ondisk $drive1" >> /tmp/part-include

​    echo "part /home --fstype ext3 --size 1 --grow --ondisk hdb" >> /tmp/part-include

else

​    \#1 drive

​    echo "#partitioning scheme generated in %pre for 1 drive" > /tmp/part-include

​    echo "clearpart --all" >> /tmp/part-include

​    echo "part /boot --fstype ext3 --size 75" >> /tmp/part-include

​    echo "part swap --recommended" >> /tmp/part-include

​    echo "part / --fstype ext3 --size 2048" >> /tmp/part-include

​    echo "part /home --fstype ext3 --size 2048 --grow" >> /tmp/part-include

fi

This script determines the number of hard drives in the system and writes a text file with a different partitioning scheme depending on whether it has one or two drives. Instead of having a set of partitioning commands in the kickstart file, include the line:

%include /tmp/part-include

The partitioning commands selected in the script are used.

#### Note

- The pre-installation script section of kickstart *cannot* manage multiple install trees or source media. This information must be included for each created ks.cfg file, as the pre-installation script occurs during the second stage of the installation process.

### Post-installation Script

You have the option of adding commands to run on the system once the installation is complete. This section must be at the end of the kickstart file and must start with the %post  command. This section is useful for functions such as installing additional software and configuring an additional nameserver.

#### Notes

- If you configured the network with static IP information, including a nameserver, you can access the network and resolve IP addresses in the %post  section. If you configured the network for DHCP, the /etc/resolv.conf  file has not been completed when the installation executes the %post  section. You can access the network, but you can not resolve IP addresses. Thus, if you are using DHCP, you must specify IP addresses in the %post  section.

- The post-install script is run in a chroot environment; therefore, performing tasks such as copying scripts or RPMs from the installation media do not work.

  - --nochroot 

    Allows you to specify commands that you would like to run outside of the chroot environment.The following example copies the file /etc/resolv.conf  to the file system that was just installed.%post --nochrootcp /etc/resolv.conf /mnt/sysimage/etc/resolv.conf

  - --interpreter */usr/bin/python* 

    Allows you to specify a different scripting language, such as Python. Replace */usr/bin/python* with the scripting language of your choice.

  - --log */path/to/logfile* 

    Logs the output of the post-install script. Note that the path of the log file must take into account whether or not you use the --nochroot  option. For example, without --nochroot:This command is available in Red Hat Enterprise Linux 5.5 and later.%post --log=/root/ks-post.logwith --nochroot:%post --nochroot --log=/mnt/sysimage/root/ks-post.log

### Examples

Register the system to a Red Hat Network Satellite, using a subshell to log the result in Red Hat Enterprise Linux 5.4 and earlier:

%post

( # Note that in this example we run the entire %post section as a subshell for logging.

wget -O- http://proxy-or-sat.example.com/pub/bootstrap_script | /bin/bash

/usr/sbin/rhnreg_ks --activationkey=*<activationkey>*

\# End the subshell and capture any output to a post-install log file.

) 1>/root/post_install.log 2>&1

Register the system to a Red Hat Network Satellite, using the --log  option to log the result in Red Hat Enterprise Linux 5.5 and later:

%post --log=/root/ks-post.log

wget -O- http://proxy-or-sat.example.com/pub/bootstrap_script | /bin/bash

/usr/sbin/rhnreg_ks --activationkey=*<activationkey>*

Run a script named runme  from an NFS share:

mkdir /mnt/temp

mount -o nolock 10.10.0.2:/usr/new-machines /mnt/temp open -s -w --

/mnt/temp/runme

umount /mnt/temp

### Note

NFS file locking is *not* supported while in kickstart mode, therefore -o nolock  is required when mounting an NFS mount.

### Making the Kickstart File Available

A kickstart file must be placed in one of the following locations:

- On a boot diskette
- On a boot CD-ROM
- On a network

Normally a kickstart file is copied to the boot diskette, or made available on the network. ***The network-based approach is most commonly used, as most kickstart installations tend to be performed on networked computers.***

Let us take a more in-depth look at where the kickstart file may be placed.

### 31.8.1. Creating Kickstart Boot Media

Diskette-based booting is no longer supported in Red Hat Enterprise Linux. Installations must use CD-ROM or flash memory products for booting. However, the kickstart file may still reside on a diskette's top-level directory, and must be named ks.cfg.

To perform a CD-ROM-based kickstart installation, the kickstart file must be named ks.cfg  and must be located in the boot CD-ROM's top-level directory. Since a CD-ROM is read-only, the file must be added to the directory used to create the image that is written to the CD-ROM. Refer to [Section 2.4.1, "Alternative Boot Methods"](http://docs.redhat.com/docs/en-US/Red_Hat_Enterprise_Linux/5/html/Installation_Guide/ch02s04.html#sect-New_Users-Alternative_Boot_Methods) for instructions on creating boot media; however, before making the file.iso  image file, copy the ks.cfg  kickstart file to the isolinux/  directory.

To perform a pen-based flash memory kickstart installation, the kickstart file must be named ks.cfg  and must be located in the flash memory's top-level directory. Create the boot image first, and then copy the ks.cfg  file.

### Note

Creation of USB flash memory pen drives for booting is possible, but is heavily dependent on system hardware BIOS settings. Refer to your hardware manufacturer to see if your system supports booting to alternate devices.

### Making the Kickstart File Available on the Network

Network installations using kickstart are quite common, because system administrators can easily automate the installation on many networked computers quickly and painlessly. In general, the approach most commonly used is for the administrator to have both a BOOTP/DHCP server and an NFS server on the local network. The BOOTP/DHCP server is used to give the client system its networking information, while the actual files used during the installation are served by the NFS server. Often, these two servers run on the same physical machine, but they are not required to.

To perform a network-based kickstart installation, you must have a BOOTP/DHCP server on your network, and it must include configuration information for the machine on which you are attempting to install Red Hat Enterprise Linux. The BOOTP/DHCP server provides the client with its networking information as well as the location of the kickstart file.

If a kickstart file is specified by the BOOTP/DHCP server, the client system attempts an NFS mount of the file's path, and copies the specified file to the client, using it as the kickstart file. The exact settings required vary depending on the BOOTP/DHCP server you use.

Here is an example of a line from the dhcpd.conf  file for the DHCP server:

filename *"/usr/new-machine/kickstart/"*; next-server *blarg.redhat.com;*

Note that you should replace the value after filename  with the name of the kickstart file (or the directory in which the kickstart file resides) and the value after next-server  with the NFS server name.

If the file name returned by the BOOTP/DHCP server ends with a slash ("/"), then it is interpreted as a path only. In this case, the client system mounts that path using NFS, and searches for a particular file. The file name the client searches for is:

*<ip-addr>*-kickstart

The `<ip-addr>`  section of the file name should be replaced with the client's IP address in dotted decimal notation. For example, the file name for a computer with an IP address of 10.10.0.1 would be 10.10.0.1-kickstart.

Note that if you do not specify a server name, then the client system attempts to use the server that answered the BOOTP/DHCP request as its NFS server. If you do not specify a path or file name, the client system tries to mount /kickstart  from the BOOTP/DHCP server and tries to find the kickstart file using the same *<ip-addr>*-kickstart  file name as described above.

### Making the Installation Tree Available

The kickstart installation must access an *installation tree*. An installation tree is a copy of the binary Red Hat Enterprise Linux CD-ROMs with the same directory structure.

If you are performing a CD-based installation, insert the Red Hat Enterprise Linux CD-ROM #1 into the computer before starting the kickstart installation.

If you are performing a hard drive installation, make sure the ISO images of the binary Red Hat Enterprise Linux CD-ROMs are on a hard drive in the computer.

If you are performing a network-based (NFS, FTP, or HTTP) installation, you must make the installation tree available over the network. Refer to [Section 2.5, "Preparing for a Network Installation"](http://docs.redhat.com/docs/en-US/Red_Hat_Enterprise_Linux/5/html/Installation_Guide/s1-steps-network-installs-x86.html) for details.

### Starting a Kickstart Installation

To begin a kickstart installation, you must boot the system from boot media you have made or the Red Hat Enterprise Linux CD-ROM #1, and enter a special boot command at the boot prompt. The installation program looks for a kickstart file if the ks  command line argument is passed to the kernel.

- CD-ROM #1 and Diskette

  The linux ks=floppy command also works if the ks.cfg  file is located on a vfat or ext2 file system on a diskette and you boot from the Red Hat Enterprise Linux CD-ROM #1.An alternate boot command is to boot off the Red Hat Enterprise Linux CD-ROM #1 and have the kickstart file on a vfat or ext2 file system on a diskette. To do so, enter the following command at the boot:  prompt:linux ks=hd:fd0:/ks.cfg

- With Driver Disk

  If you need to use a driver disk with kickstart, specify the dd option as well. For example, to boot off a boot diskette and use a driver disk, enter the following command at the boot:  prompt:linux ks=floppy dd

- Boot CD-ROM

  If the kickstart file is on a boot CD-ROM as described in [Section 31.8.1, "Creating Kickstart Boot Media"](http://docs.redhat.com/docs/en-US/Red_Hat_Enterprise_Linux/5/html/Installation_Guide/s1-kickstart2-putkickstarthere.html#s2-kickstart2-boot-media), insert the CD-ROM into the system, boot the system, and enter the following command at the boot:  prompt (where ks.cfg  is the name of the kickstart file):linux ks=cdrom:/ks.cfg

Other options to start a kickstart installation are as follows:

- askmethod 

  Do not automatically use the CD-ROM as the install source if we detect a Red Hat Enterprise Linux CD in your CD-ROM drive.

- autostep 

  Make kickstart non-interactive.

- debug 

  Start up pdb immediately.

- dd 

  Use a driver disk.

- dhcpclass=*<class>* 

  Sends a custom DHCP vendor class identifier. ISC's dhcpcd can inspect this value using "option vendor-class-identifier".

- dns=*<dns>* 

  Comma separated list of nameservers to use for a network installation.

- driverdisk 

  Same as 'dd'.

- expert 

  Turns on special features:allows partitioning of removable mediaprompts for a driver disk

- gateway=*<gw>* 

  Gateway to use for a network installation.

- graphical 

  Force graphical install. Required to have ftp/http use GUI.

- isa 

  Prompt user for ISA devices configuration.

- ip=*<ip>* 

  IP to use for a network installation, use 'dhcp' for DHCP.

- keymap=*<keymap>* 

  Keyboard layout to use. Valid values are those which can be used for the 'keyboard' kickstart command.

- ks=nfs:*<server>*:/*<path>* 

  The installation program looks for the kickstart file on the NFS server *<server>*, as file *<path>*. The installation program uses DHCP to configure the Ethernet card. For example, if your NFS server is server.example.com and the kickstart file is in the NFS share /mydir/ks.cfg, the correct boot command would be ks=nfs:server.example.com:/mydir/ks.cfg.

- ks=http://*<server>*/*<path>* 

  The installation program looks for the kickstart file on the HTTP server *<server>*, as file *<path>*. The installation program uses DHCP to configure the Ethernet card. For example, if your HTTP server is server.example.com and the kickstart file is in the HTTP directory /mydir/ks.cfg, the correct boot command would be ks=http://server.example.com/mydir/ks.cfg.

- ks=floppy 

  The installation program looks for the file ks.cfg  on a vfat or ext2 file system on the diskette in /dev/fd0.

- ks=floppy:/*<path>* 

  The installation program looks for the kickstart file on the diskette in /dev/fd0, as file *<path>*.

- ks=hd:*<device>*:/*<file>* 

  The installation program mounts the file system on *<device>* (which must be vfat or ext2), and look for the kickstart configuration file as *<file>* in that file system (for example, ks=hd:sda3:/mydir/ks.cfg).

- ks=file:/*<file>* 

  The installation program tries to read the file *<file>* from the file system; no mounts are done. This is normally used if the kickstart file is already on the initrd  image.

- ks=cdrom:/*<path>* 

  The installation program looks for the kickstart file on CD-ROM, as file *<path>*.

- ks 

  If ks  is used alone, the installation program configures the Ethernet card to use DHCP. The kickstart file is read from the "bootServer" from the DHCP response as if it is an NFS server sharing the kickstart file. By default, the bootServer is the same as the DHCP server. The name of the kickstart file is one of the following:If DHCP is specified and the boot file begins with a /, the boot file provided by DHCP is looked for on the NFS server.If DHCP is specified and the boot file begins with something other than a /, the boot file provided by DHCP is looked for in the /kickstart  directory on the NFS server.If DHCP did not specify a boot file, then the installation program tries to read the file /kickstart/1.2.3.4-kickstart, where *1.2.3.4* is the numeric IP address of the machine being installed.

- ksdevice=*<device>* 

  The installation program uses this network device to connect to the network. For example, consider a system connected to an NFS server through the eth1 device. To perform a kickstart installation on this system using a kickstart file from the NFS server, you would use the command ks=nfs:*<server>*:/*<path>* ksdevice=eth1  at the boot:  prompt.

- kssendmac 

  Adds HTTP headers to ks=http:// request that can be helpful for provisioning systems. Includes MAC address of all nics in CGI environment variables of the form: "X-RHN-Provisioning-MAC-0: eth0 01:23:45:67:89:ab".

- lang=*<lang>* 

  Language to use for the installation. This should be a language which is valid to be used with the 'lang' kickstart command.

- loglevel=*<level>* 

  Set the minimum level required for messages to be logged. Values for <level> are debug, info, warning, error, and critical. The default value is info.

- lowres 

  Force GUI installer to run at 640x480.

- mediacheck 

  Activates loader code to give user option of testing integrity of install source (if an ISO-based method).

- method=cdrom 

  Do a CDROM based installation.

- method=ftp://*<path>* 

  Use <path> for an FTP installation.

- method=hd:*<dev>*:*<path>* 

  Use <path> on <dev> for a hard drive installation.

- method=http://*<path>* 

  Use <path> for an HTTP installation.

- method=nfs:*<path>* 

  Use <path> for an NFS installation.

- netmask=*<nm>* 

  Netmask to use for a network installation.

- nofallback 

  If GUI fails exit.

- nofb 

  Do not load the VGA16 framebuffer required for doing text-mode installation in some languages.

- nofirewire 

  Do not load support for firewire devices.

- noipv6 

  Disable IPv6 networking during installation.This option is not available during PXE installationsDuring installations from a PXE server, IPv6 networking might become active before anaconda processes the Kickstart file. If so, this option will have no effect during installation.

- nomount 

  Don't automatically mount any installed Linux partitions in rescue mode.

- nonet 

  Do not auto-probe network devices.

- noparport 

  Do not attempt to load support for parallel ports.

- nopass 

  Don't pass keyboard/mouse info to stage 2 installer, good for testing keyboard and mouse config screens in stage2 installer during network installs.

- nopcmcia 

  Ignore PCMCIA controller in system.

- noprobe 

  Do not attempt to detect hw, prompts user instead.

- noshell 

  Do not put a shell on tty2 during install.

- nostorage 

  Do not auto-probe storage devices (SCSI, IDE, RAID).

- nousb 

  Do not load USB support (helps if install hangs early sometimes).

- nousbstorage 

  Do not load usbstorage module in loader. May help with device ordering on SCSI systems.

- rescue 

  Run rescue environment.

- resolution=*<mode>* 

  Run installer in mode specified, '1024x768' for example.

- serial 

  Turns on serial console support.

- skipddc 

  Skips DDC probe of monitor, may help if it's hanging system.

- syslog=*<host>*[:*<port>*] 

  Once installation is up and running, send log messages to the syslog process on *<host>*, and optionally, on port *<port>*. Requires the remote syslog process to accept connections (the -r option).

- text 

  Force text mode install.

- updates 

  Prompt for floppy containing updates (bug fixes).

- updates=ftp://*<path>* 

  Image containing updates over FTP.

- updates=http://*<path>* 

  Image containing updates over HTTP.

- upgradeany 

  Don't require an /etc/redhat-release that matches the expected syntax to upgrade.

- vnc 

  Enable vnc-based installation. You will need to connect to the machine using a vnc client application.

- vncconnect=*<host>*[:*<port>*] 

  Once installation is up and running, connect to the vnc client named *<host>*, and optionally use port *<port>*.Requires 'vnc' option to be specified as well.

- vncpassword=*<password>* 

  Enable a password for the vnc connection. This will prevent someone from inadvertently connecting to the vnc-based installation.Requires 'vnc' option to be specified as well.

## 

##  

