# compress-pfam-data-benchmark-zstd
Minimal test to see how well [zstd](https://github.com/facebook/zstd) compression compares to gz for the datafile Pfam-A.full.gz downloadable from [ftp://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.full.gz](ftp://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.full.gz).


Instead of using the whole Pfam-A.full.gz, only the first 1000,000,000 bytes were considered.

The result:

| type          | file size | read speed | 
| ------------- |  ------: | ------: |
| uncompressed  | 9703136359| 527MiB/s |
| gz            | 1000000000 |  207MiB/s  |
| zstd          | 431850930 |  813MiB/s |


As for the details read below:

## Install zstd

### Install zstd on Fedora

Run `sudo dnf install zstd`

(To get the latest and greatest zstd, you will probable need to build from source code instead)


### Install zstd on Ubuntu

Run `sudo apt-get install zstd`

(To get the latest and greatest zstd, you will probable need to build from source code instead)

### Install zstd from source code

Right now (9 January 2020) the latest version is __1.4.4__.
If you read these instructions some time in the future replace __1.4.4__ with
the latest version.


1. Download the source package from https://github.com/facebook/zstd/releases into the local directory _~/Downloads/_
2. Install the ninja build system
   - On Fedora run `sudo dnf install ninja`
   - On Ubuntu run `sudo apt-get install ninja-build`
3. Run the commands

```
mkdir ~/installdir
builddir=$(mktemp -d)
sourcedir=$(mktemp -d)
tar -x -C $sourcedir -f ~/Downloads/zstd-1.4.4.tar.gz 
cd $builddir
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/home/testuser/installdir -G Ninja $sourcedir/zstd-1.4.4/build/cmake
ninja && ninja install
```

### Benchmark zstd on Pfam-A.full.gz

This benchmark was done on a computer with

* __Disk__: HDD (i.e. a normal SATA mechanical hard drive).
* __CPU__: AMD Ryzen 5 1600 Six-Core Processor
* __Operating system__: Ubuntu 18.04.3

The result: 

```
testuser@linuxdesktop:~$ ~/installdir/bin/zstdmt --version
*** zstd command line interface 64-bits v1.4.4, by Yann Collet ***
testuser@linuxdesktop:~$ curl -s ftp://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.full.gz  | head -c 1000000000 > pfam.gz
testuser@linuxdesktop:~$ gunzip -c pfam.gz > pfam.txt
testuser@linuxdesktop:~$ ~/installdir/bin/zstdmt -19 pfam.txt
pfam.txt              :  4.45%   (9703136359 => 431850930 bytes, pfam.txt.zst)   
testuser@linuxdesktop:~$ ls -l pfam.*
-rw-r--r-- 1 root root 1000000000 jan  9 14:29 pfam.gz
-rw-r--r-- 1 root root 9703136359 jan  9 14:38 pfam.txt
-rw-r--r-- 1 root root  431850930 jan  9 14:49 pfam.txt.zst
testuser@linuxdesktop:~$ sync && sudo -c "echo 3 > /proc/sys/vm/drop_caches"
testuser@linuxdesktop:~$ ~/installdir/bin/zstdmt -d -c pfam.txt.zst | pv --average-rate > /dev/null
pfam.txt.zst         : 9703136359 bytes                                         
[ 813MiB/s]
testuser@linuxdesktop:~$ sync && sudo -c "echo 3 > /proc/sys/vm/drop_caches"
testuser@linuxdesktop:~$ cat pfam.txt | pv --average-rate > /dev/null
[ 527MiB/s]
testuser@linuxdesktop:~$ sync && sudo -c "echo 3 > /proc/sys/vm/drop_caches"
testuser@linuxdesktop:~$ gunzip -c pfam.gz | pv --average-rate > /dev/null
[ 207MiB/s]
gzip: pfam.gz: unexpected end of file
[ 207MiB/s]
testuser@linuxdesktop:~$ 
```
