### Device Files
The kernel presents the I/O interfaces of devices to user processes as files which are known as _device files_.
These device files are located in the `/dev` directory, using `ls -l` to see the device permissions on it looks something like this 
```
crw-------  1 root root       10,   124 Jun  4 11:07 acpi_thermal_rel
crw-r--r--  1 root root       10,   235 Jun  4 11:07 autofs
drwxr-xr-x  2 root root             320 Jun  4 11:07 block
crw-------  1 root root       10,   234 Jun  4 11:07 btrfs-control
drwxr-xr-x  3 root root              60 Jun  4 11:07 bus
drwxr-xr-x  2 root root            3640 Jun  4 11:07 char
crw--w----  1 root tty         5,     1 Jun  4 11:07 console
----------------------------------------------------------------------
```
Note that these files also have a special first character besides normal files, if this character is `b, c, p, or s` the file is a device, these letters stand for _block_, _character_, _pipe_ and _socket_.

__Block Device__
Programs access data from such devices in fixed chunks, consider `sda1`, which is a disk device, a type of block device. Disks are easily split into blocks of data and can be indexed easily, hence data is read in blocks from them.

__Character Device__
Character devices work with data streams, you can read characters from them or write characters to them. They don't have a size. Printers are represented by this. 

__Pipe Devices__
Named pipes are like character devices, with another process at the other end instead of a kernel driver.

__Socket Devices__
Sockets are used for inter process communication and are also found out of the `/dev` directory, these are Unix domain sockets.

- The numbers before dates in the `ls -l` are the major and minor device numbers used by the kernel to identify devices.

> Not all have device files, for example network interfaces.

### The sysf Device Path
To provide a uniform view for attached devices based on hardware attributes instead of the order in which they are found the `/sys` directory exists.

The directory has various sub directories and files which give access to various properties about devices, this is meant to be read by programs instead of humans, to find the sysf location of a device use the `udevadm` command as follows
```
$ udevadm info --query=all --name=/dev/sda
```

### dd and Devices
`dd` is useful when working with block and character devices. It reads from a input file or a stream and writes to a output file or a stream, it can read a section or also do some encoding along the way.

`dd` copies data in blocks of fixed size, using dd with character device is as follows
```
$ dd if=/dev/zero of=new_file bs=1024 count=1
```
The following `dd` options are important
- __`if=file`__: The input file, default is stdin
- __`of=file`__: The output file, default is stdout
- __`bs=size`__: The block size, to abbreviate large chunks of data use `b` and `k` to symbolise 512 and 1024 bytes.
- __`ibs=size,obs=size`__: The input and output block sizes, `bs` sets both of them to be the same.
- __`count=num`__: The number of blocks to be copied.
- __`skip=num`__: Skips past the first _num_ blocks and then starts copying.

##### Hard Disk: `/dev/sd*`
Most disks attached to linux systems have the `sd` prefix as `/dev/sda` or `/dev/sdb` and so on. These devices represent entire disks and partitions have their own device files `/dev/sda1` and so on.

The name comes from _SCSI disk_ which stands for _Small Computer System Interface_, which is an old communication protocol between disks and computers. To list `SCSI` devices us the `lsscsi` command.

Devices are assigned device files in the order of which they are found, if say we have `/dev/sda`, `/dev/sdb`, `/dev/sdc` and we remove `/dev/sdb`, then `/dev/sdc` is assigned `/dev/sdb`, causing issues in configuration files, thus many linux systems use UUID or LVM(logical volume manager.)
##### Virtual Disks: `/dev/xvd*, /dev/vd*`
Disk devices optimized for virtual machines have such prefixes. 
##### Non-Volatile Memory Drives: `/dev/nvme*`
Used for `nvme` drives which are used for solid state storage on devices. `nvme list` can be used to list these devices.
##### Device Mapper `/dev/dm-*, /dev/mapper/*`
Used for LVM's, which is a level up from direct block storage, uses a device mapper.
##### CD and DVD drives: `/dev/sr*`
Most optical storage drives are recognised as SCSI devices, besides devices with use PATA, generic SCSI devices also use `/dev/sg0`
##### PATA Hard disks: `/dev/hd*`
Used by devices which use PATA, sometimes a SATA device is recognised as such which means it is running in compatibility mode, which hinders performance, this can be switched using your bios.
##### Terminals: `/dev/tty*, /dev/pts/* and /dev/tty`
A terminal is just a way for a program to exchange text with a user, most terminals are _pseudoterminal_ devices unlike real terminals. They are pieces talk to software to emulate terminals.

The common terminal devices are `/dev/tty1`(first virtual console) and `/dev/pts/0`(first pseudoterminal device).

`/dev/tty` device is the controlling terminal of the current process. If a program is reading from and writing to a terminal, this device is a synonym for that terminal.

Linux has two display modes, text and graphical, it supports _virtual consoles_ to multiplex the display. Each console runs in either graphics or text mode and you can switch between them using ALT+F1/F2 or CTRL+ALT+F1/F2, depending on if you are in a text or graphical virtual console respectively.

Most text mode virtual consoles are running `getty` which is a login prompt.
##### Serial Ports: `/dev/ttyS*, /dev/ttyUSB*, /dev/ttyACM*`
RS-232 type devices and other serial ports are represented are true terminal devices. They can be connected to by a terminal using the `screen` command.
These are mostly used for microcontroller based applications such as circuit boards.
##### Parallel Ports: `/dev/lp0 and /dev/lp1`
These are for unidirectional parallel ports, can be used for say sending files to printers using the `cat` command, along with a form feed or a reset afterwards.

The bidirectional parallel ports are `/dev/parport0` and `/dev/parport1`
##### Audio Device: `/dev/snd/*, /dev/dsp, ..`
Linux has two sets of audio devices, these are the ones which use the ALSA(_Advanced Linux Sound Architecture_) and the OSS(_Open Sound System_).
ALSA devices are in `/dev/snd` and are difficult to work with directly, and also support OSS back compatibility, OSS devices may play wav files sent to it however this most likely wont work.
##### Device File Creation
Device files are usually created automatically, however can be created using `mknod` as follows
```
$ mknod /dev/sda1 b 8 1
```
 Specifying that it is a block device and has major and minor numbers 8 and 1.
 Manually creation of files is tedious and updates keep adding support for new files, 

 