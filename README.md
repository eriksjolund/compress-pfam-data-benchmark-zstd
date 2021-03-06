# compress-pfam-data-benchmark-zstd
A minimal benchmark test to see how [__zstd__](https://github.com/facebook/zstd) (__zstandard__) compression compares to [__gzip__](https://en.wikipedia.org/wiki/Gzip) for the datafile _Pfam-A.full.gz_ that is downloadable from [ftp://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.full.gz](ftp://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.full.gz).


Instead of using the whole Pfam-A.full.gz, only the first 1000,000,000 bytes were considered.

# Result:

| type          | file size | read speed | read command |
| ------------- |  ------: | ------: | -------------|
| uncompressed  | 9703136359| 527 MB/s |  cat file > /dev/null          |
| gzip  (using gunzip)   | 1000000000 |  220 MB/s  | gunzip -c file.gz  > /dev/null  |   
| gzip  (using pigz) | 1000000000 |  417 MB/s  | pigz -d -c file.gz > /dev/null |
| zstd          | 431850930 |  1536 MB/s |  zstdmt -d -c file.zst  > /dev/null |


This benchmark was performed on a computer with

* __Disk__: HDD (i.e. a standard SATA mechanical hard drive).
* __CPU__: _AMD Ryzen 5 1600_, a six-core processor from 2017
* __Operating system__: _Ubuntu 18.04.3_




# Details
## Install zstd

### Install zstd on Fedora

Run `sudo dnf install zstd`

(To get the latest and greatest zstd, you will probable need to build from source code instead)


### Install zstd on Ubuntu

Run `sudo apt-get install zstd`

(To get the latest and greatest zstd, you will probable need to build from source code instead)

### Install zstd from source code

Right now (9 January 2020) the latest version of zstd is __1.4.4__.
If you read these instructions some time in the future replace __1.4.4__ with
the latest version.


1. Download the source package _zstd-1.4.4.tar.gz_ from https://github.com/facebook/zstd/releases into the local directory _~/Downloads/_
2. Install the build requirements
   - On Fedora run `sudo dnf install cmake ninja-build`
   - On Ubuntu run `sudo apt-get install cmake ninja-build`
3. Run the commands

```
VERSION=1.4.4
mkdir ~/installdir
builddir=$(mktemp -d)
sourcedir=$(mktemp -d)
tar -x -C $sourcedir -f ~/Downloads/zstd-$VERSION.tar.gz 
cd $builddir
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=~/installdir -G Ninja $sourcedir/zstd-$VERSION/build/cmake
ninja && ninja install
```

### Benchmark zstd on Pfam-A.full.gz

Shell session of the benchmark:

```
testuser@linuxdesktop:~$ ~/installdir/bin/zstdmt --version
*** zstd command line interface 64-bits v1.4.4, by Yann Collet ***
testuser@linuxdesktop:~$ curl -s ftp://ftp.ebi.ac.uk/pub/databases/Pfam/current_release/Pfam-A.full.gz  | head -c 1000000000 > pfam.gz
testuser@linuxdesktop:~$ gunzip -c pfam.gz > pfam.txt
testuser@linuxdesktop:~$ ~/installdir/bin/zstdmt -19 pfam.txt
pfam.txt              :  4.45%   (9703136359 => 431850930 bytes, pfam.txt.zst)   
testuser@linuxdesktop:~$ ls -l pfam.*
-rw-r--r-- 1 testuser testuser 1000000000 jan  9 14:29 pfam.gz
-rw-r--r-- 1 testuser testuser 9703136359 jan  9 14:38 pfam.txt
-rw-r--r-- 1 testuser testuser  431850930 jan  9 14:49 pfam.txt.zst
testuser@linuxdesktop:~$ sync && sudo su -c "echo 3 > /proc/sys/vm/drop_caches"
testuser@linuxdesktop:~$ time ~/installdir/bin/zstdmt -d -c pfam.txt.zst  > /dev/null
pfam.txt.zst         : 9703136359 bytes                                         

real	0m6,023s
user	0m5,764s
sys	0m0,184s
testuser@linuxdesktop:~$ sync && sudo su -c "echo 3 > /proc/sys/vm/drop_caches"
testuser@linuxdesktop:~$ time cat pfam.txt  > /dev/null

real	0m17,545s
user	0m0,016s
sys	0m3,769s
testuser@linuxdesktop:~$ sync && sudo su -c "echo 3 > /proc/sys/vm/drop_caches"
testuser@linuxdesktop:~$ time gunzip -c pfam.gz  > /dev/null
gzip: pfam.gz: unexpected end of file

real	0m42,051s
user	0m41,592s
sys	0m0,420s
testuser@linuxdesktop:~$ sync && sudo su -c "echo 3 > /proc/sys/vm/drop_caches"
testuser@linuxdesktop:~$time pigz -d -c pfam.gz > /dev/null
pigz: skipping: pfam.gz: corrupted -- incomplete deflate data
pigz: abort: internal threads error

real	0m21,865s
user	0m32,466s
sys	0m2,007s
testuser@linuxdesktop:~$ 
```

To avoid that the Linux caching system has any influence on the  benchmark, the command
`sync && sudo su -c "echo 3 > /proc/sys/vm/drop_caches"` was run before each 
benchmarked command.
