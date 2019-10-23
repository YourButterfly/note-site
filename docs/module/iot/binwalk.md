# binwalk

## issues

### failed to run external extractor 'sasquatch -p 1 -le -d 'squashfs-root'

#### description

Miss binary ***sasquatch***

```txt
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             DLOB firmware header, boot partition: "dev=/dev/mtdblock/1"
112           0x70            LZMA compressed data, properties: 0x5D, dictionary size: 33554432 bytes, uncompressed size: 4624518 bytes
1441904       0x160070        PackImg section delimiter tag, little endian size: 2131456 bytes; big endian size: 8790016 bytes

WARNING: Extractor.execute failed to run external extractor 'sasquatch -p 1 -le -d 'squashfs-root' '%e'': [Errno 2] No such file or directory: 'sasquatch': 'sasquatch', 'sasquatch -p 1 -le -d 'squashfs-root' '%e'' might not be installed correctly

WARNING: Extractor.execute failed to run external extractor 'sasquatch -p 1 -be -d 'squashfs-root' '%e'': [Errno 2] No such file or directory: 'sasquatch': 'sasquatch', 'sasquatch -p 1 -be -d 'squashfs-root' '%e'' might not be installed correctly
1441936       0x160090        Squashfs filesystem, little endian, non-standard signature, version 3.0, size: 8789043 bytes, 2427 inodes, blocksize: 65536 bytes, created: 2012-11-02 04:51:50
```

#### solution

```shell
cd /path/to/directory
git https://github.com/devttys0/    .git
cd sasquatch
./build

# Phaaga , failed to build due to network error
# vim build.sh
# if [ ! -e squashfs4.3.tar.gz ]
# then
#     wget  https://nchc.dl.sourceforge.net/project/squashfs/squashfs/squashfs4.3/squashfs4.3.tar.gz
#     #wget https://downloads.sourceforge.net/project/squashfs/squashfs/squashfs4.3/squashfs4.3.tar.gz
# fi

```

### failed to find lzma in python2 virtualenv

description

```txt
WARNING: The Python LZMA module could not be found. It is *strongly* recommended that you install this module for binwalk to provide proper LZMA identification and extraction results.

WARNING: The Python LZMA module could not be found. It is *strongly* recommended that you install this module for binwalk to provide proper LZMA identification and extraction results.
```

#### solution

```shell
# workon virtualenv
pip install pyliblzma
# It`s not the 'pylzma'
```

### binwalk in python virtualenv

#### description

```text
After installing binwalk for python3, need to install binwalk (python2) for FAT.
```

#### solution

Warnning: It only contain python2 environment.

更方便的方法，在root下进入python虚环境后执行binwalk安装脚本

```shell
# workon virtua_ env
sudo apt-get install git build-essential libqt4-opengl mtd-utils gzip bzip2 tar arj lhasa p7zip p7zip-full cabextract cramfsswap squashfs-tools zlib1g-dev liblzma-dev liblzo2-dev sleuthkit default-jdk lzop srecord cpio

sudo apt-get install python-crypto python-lzo python-lzma python-pip python-tk

pip install matplotlib capstone

git clone https://github.com/devttys0/yaffshiv
(cd yaffshiv && python2 setup.py install)
rm -rf yaffshiv

# install_sasquatch for local bin, not python2 package

git clone https://github.com/sviehb/jefferson
(cd jefferson &&  python2 setup.py install)
rm -rf jefferson

# install_unstuff , install_cramfstools installed in /usr/local/bin, not python2 package
git clone https://github.com/jrspruitt/ubi_reader
(cd ubi_reader && git reset --hard 0955e6b95f07d849a182125919a1f2b6790d5b51 && python setup.py install)
rm -rf ubi_reader
```
