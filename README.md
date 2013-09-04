BeagleBone Black Setup
===================

This is where I will document the complete setup for a BeagleBone Black for use with [Ruby](http://www.ruby-lang.org/) development.

0 to Ruby in 7 Steps
====================

1. Setup Debian Wheezy [original here](http://circuitco.com/support/index.php?title=Debian_On_BeagleBone_Black)
   1. Download image file
      * Install to SD [here](http://www.armhf.com/index.php/download/)
      * Install to eMMC [here](http://elinux.org/BeagleBoardDebian#eMMC:_BeagleBone_Black)
   2. Unpack the image
      * Mac (requires p7zip installed via homebrew)
         * 7z x debian-wheezy-7.0.0-armhf-3.8.13-bone20.img.xz
   3. Copy the image to the sd card
      * Mac (terminal):
         1. diskutil list
         2. Insert SD card
         3. diskutil list (note the new disk which should be your SD card)
         4. diskutil unmountDisk /dev/disk2 (replace 2 with appropriate disk number from the last step)
         5. sudo dd if=debian-wheezy-7.0.0-armhf-3.8.13-bone20.img of=/dev/rdisk2 bs=1m (use the downloaded filename and disk number from above)
         6. diskutil eject /dev/disk2 (replace 2 with the above disk number)
         7. Remove SD card
   4. Follow [these](http://circuitco.com/support/index.php?title=Debian_On_BeagleBone_Black) instructions to finish booting your BeagleBoard Black
2. Setup your ssh public key (optional)
   * Local:
      * scp ~/.ssh/id_rsa.pub debian@192.168.7.2:~/
         * Default password is "debian"
   * Remote:
      1. mkdir .ssh
      2. cat id_rsa.pub >> .ssh/authorized_keys
      3. chmod 700 .ssh
      4. chmod 600 .ssh/authorized_keys
3. Sudo without a password (optional)
   1. sudo visudo
      * Default password is "debian"
   2. Add this on its own line at the end of the file
      * debian ALL=(ALL) NOPASSWD: ALL
4. Resize the partition to use all available space [reference](http://www.armhf.com/index.php/expanding-linux-partitions-part-2-of-2/)
   1. sudo fdisk /dev/mmcblk0
      1. d        (delete partition)
      2. 2        (2nd partition)
      3. n        (new partition)
      4. p        (primary partition)
      5. 2        (2nd partition)
      6. (Enter)  (Default value)
      7. (Enter)  (Default value)
      8. w        (write partition table)
   2. sudo reboot
   3. sudo resize2fs /dev/mmcblk0p2
5. Update system
   1. sudo aptitude update
   2. sudo aptitude upgrade
6. Install pre-reqs
   * sudo aptitude install build-essential openssl libreadline6 libreadline6-dev zlib1g zlib1g-dev libssl-dev libyaml-dev libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison
7. Install ruby
   1. wget ftp://ftp.ruby-lang.org/pub/ruby/2.0/ruby-2.0.0-p247.tar.gz
   2. tar xzf ruby-2.0.0-p247.tar.gz
   3. cd ruby-2.0.0-p247
   4. ./configure --prefix=/usr/local
   5. make && sudo make install
   3. gem update --system

Install DTC (device tree compiler)
==================================
1. wget -c https://raw.github.com/RobertCNelson/tools/master/pkgs/dtc.sh
2. chmod +x dtc.sh
3. ./dtc.sh
4. cd git/dtc/
5. git checkout master -f
6. git pull git://github.com/RobertCNelson/dtc.git dtc-fixup-65cc4d2
7. make PREFIX=/usr/local/ CC=gcc CROSS_COMPILE= all
8. make PREFIX=/usr/local/ install

Contributions
=============

BeagleBone Black Setup would love your contributions! No contribution is too small. Please consider:

* updating for new releases
* updating documentation
* adding examples
* fixing typos

For the best chance of having your changes merged, please:

1. Ask us! We'd love to hear what you're up to.
2. Fork the project.
3. Commit your changes.
4. Submit a pull request with a thorough explanation and at least one animated GIF.

Thanks
======

Various resources used to produce this document:

[link](http://example.com)
