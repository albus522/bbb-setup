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
   1. sudo vi /etc/apt/sources.list
      1. Uncomment wheezy-backports sources
   2. sudo aptitude update
   3. sudo aptitude upgrade
6. Install pre-reqs
   * sudo aptitude install build-essential zlib1g zlib1g-dev libreadline6 libreadline6-dev libssl-dev libyaml-dev libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison
7. Install ruby
   1. wget http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.1.tar.gz
   2. tar xzf ruby-2.1.1.tar.gz
   3. cd ruby-2.1.1
   4. ./configure --prefix=/usr/local
   5. make && sudo make install
   3. gem update --system

Other useful packages
=====================
* sudo apt-get install vim screen avahi-daemon nodejs libcurl4-openssl-dev
* sudo apt-get install postgresql libpq-dev

Disable Heartbeat LED
=====================
echo none > /sys/class/leds/beaglebone\:green\:usr0/trigger

Set timezone
============
sudo dpkg-reconfigure tzdata

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

Enable W1
=========
Requires dtc update above

1. Copy BB-W1-00A0.dts to the BBB
2. dtc -O dtb -o BB-W1-00A0.dtbo -b 0 -@ BB-W1-00A0.dts
3. cp BB-W1-00A0.dtbo /lib/firmware/
4. sudo -i
5. echo BB-W1:00A0 > /sys/devices/bone_capemgr.9/slots

Enable the PRUSS
================
echo BB-BONE-PRU-01 > /sys/devices/bone_capemgr.*/slots
modprobe uio_pruss

Read event file
===============
  ```ruby
  File.open("/dev/input/event1", "r") do |f|
    loop do
      str = f.read(32)
      evt = str.unpack('LLSSL')
      puts "#{evt[3]}: #{evt[4]} at #{Time.at(evt[0] + evt[1] / 1000000.0)}"
    end
  end
  ```

Pinmux Offsets and GPIO export numbers
======================================

<table>
<tr><td>       </td><td> Offset </td><td> Name       </td><td> Mode0             </td><td> GPIO </td></tr>
<tr><td> P8_1  </td><td>        </td><td> GND        </td><td>                   </td><td>      </td></tr>
<tr><td> P8_2  </td><td>        </td><td> GND        </td><td>                   </td><td>      </td></tr>
<tr><td> P8_3  </td><td> 0x018  </td><td> GPIO1_6    </td><td> gpmc_ad6          </td><td> 38   </td></tr>
<tr><td> P8_4  </td><td> 0x01C  </td><td> GPIO1_7    </td><td> gpmc_ad7          </td><td> 39   </td></tr>
<tr><td> P8_5  </td><td> 0x008  </td><td> GPIO1_2    </td><td> gpmc_ad2          </td><td> 34   </td></tr>
<tr><td> P8_6  </td><td> 0x00C  </td><td> GPIO1_3    </td><td> gpmc_ad3          </td><td> 35   </td></tr>
<tr><td> P8_7  </td><td> 0x090  </td><td> TIMER4     </td><td> gpmc_advn_ale     </td><td> 66   </td></tr>
<tr><td> P8_8  </td><td> 0x094  </td><td> TIMER7     </td><td> gpmc_oen_ren      </td><td> 67   </td></tr>
<tr><td> P8_9  </td><td> 0x09C  </td><td> TIMER5     </td><td> gpmc_be0n_cle     </td><td> 69   </td></tr>
<tr><td> P8_10 </td><td> 0x098  </td><td> TIMER6     </td><td> gpmc_wen          </td><td> 68   </td></tr>
<tr><td> P8_11 </td><td> 0x034  </td><td> GPIO1_13   </td><td> gpmc_ad13         </td><td> 45   </td></tr>
<tr><td> P8_12 </td><td> 0x030  </td><td> GPIO1_12   </td><td> GPMC_AD12         </td><td> 44   </td></tr>
<tr><td> P8_13 </td><td> 0x024  </td><td> EHRPWM2B   </td><td> gpmc_ad9          </td><td> 23   </td></tr>
<tr><td> P8_14 </td><td> 0x028  </td><td> GPIO0_26   </td><td> gpmc_ad10         </td><td> 26   </td></tr>
<tr><td> P8_15 </td><td> 0x03C  </td><td> GPIO1_15   </td><td> gpmc_ad15         </td><td> 47   </td></tr>
<tr><td> P8_16 </td><td> 0x038  </td><td> GPIO1_14   </td><td> gpmc_ad14         </td><td> 46   </td></tr>
<tr><td> P8_17 </td><td> 0x02C  </td><td> GPIO0_27   </td><td> gpmc_ad11         </td><td> 27   </td></tr>
<tr><td> P8_18 </td><td> 0x08C  </td><td> GPIO2_1    </td><td> gpmc_clk_mux0     </td><td> 65   </td></tr>
<tr><td> P8_19 </td><td> 0x020  </td><td> EHRPWM2A   </td><td> gpmc_ad8          </td><td> 22   </td></tr>
<tr><td> P8_20 </td><td> 0x084  </td><td> GPIO1_31   </td><td> gpmc_csn2         </td><td> 63   </td></tr>
<tr><td> P8_21 </td><td> 0x080  </td><td> GPIO1_30   </td><td> gpmc_csn1         </td><td> 62   </td></tr>
<tr><td> P8_22 </td><td> 0x014  </td><td> GPIO1_5    </td><td> gpmc_ad5          </td><td> 37   </td></tr>
<tr><td> P8_23 </td><td> 0x010  </td><td> GPIO1_4    </td><td> gpmc_ad4          </td><td> 36   </td></tr>
<tr><td> P8_24 </td><td> 0x004  </td><td> GPIO1_1    </td><td> gpmc_ad1          </td><td> 33   </td></tr>
<tr><td> P8_25 </td><td> 0x000  </td><td> GPIO1_0    </td><td> gpmc_ad0          </td><td> 32   </td></tr>
<tr><td> P8_26 </td><td> 0x07C  </td><td> GPIO1_29   </td><td> gpmc_csn0         </td><td> 61   </td></tr>
<tr><td> P8_27 </td><td> 0x0E0  </td><td> GPIO2_22   </td><td> lcd_vsync         </td><td> 86   </td></tr>
<tr><td> P8_28 </td><td> 0x0E8  </td><td> GPIO2_24   </td><td> lcd_pclk          </td><td> 88   </td></tr>
<tr><td> P8_29 </td><td> 0x0E4  </td><td> GPIO2_23   </td><td> lcd_hsync         </td><td> 87   </td></tr>
<tr><td> P8_30 </td><td> 0x0EC  </td><td> GPIO2_25   </td><td> lcd_ac_bias_en    </td><td> 89   </td></tr>
<tr><td> P8_31 </td><td> 0x0D8  </td><td> UART5_CTSN </td><td> lcd_data14        </td><td> 10   </td></tr>
<tr><td> P8_32 </td><td> 0x0DC  </td><td> UART5_RTSN </td><td> lcd_data15        </td><td> 11   </td></tr>
<tr><td> P8_33 </td><td> 0x0D4  </td><td> UART4_RTSN </td><td> lcd_data13        </td><td> 9    </td></tr>
<tr><td> P8_34 </td><td> 0x0CC  </td><td> UART3_RTSN </td><td> lcd_data11        </td><td> 81   </td></tr>
<tr><td> P8_35 </td><td> 0x0D0  </td><td> UART4_CTSN </td><td> lcd_data12        </td><td> 8    </td></tr>
<tr><td> P8_36 </td><td> 0x0C8  </td><td> UART3_CTSN </td><td> lcd_data10        </td><td> 80   </td></tr>
<tr><td> P8_37 </td><td> 0x0C0  </td><td> UART5_TXD  </td><td> lcd_data8         </td><td> 78   </td></tr>
<tr><td> P8_38 </td><td> 0x0C4  </td><td> UART5_RXD  </td><td> lcd_data9         </td><td> 79   </td></tr>
<tr><td> P8_39 </td><td> 0x0B8  </td><td> GPIO2_12   </td><td> lcd_data6         </td><td> 76   </td></tr>
<tr><td> P8_40 </td><td> 0x0BC  </td><td> GPIO2_13   </td><td> lcd_data7         </td><td> 77   </td></tr>
<tr><td> P8_41 </td><td> 0x0B0  </td><td> GPIO2_10   </td><td> lcd_data4         </td><td> 74   </td></tr>
<tr><td> P8_42 </td><td> 0x0B4  </td><td> GPIO2_11   </td><td> lcd_data5         </td><td> 75   </td></tr>
<tr><td> P8_43 </td><td> 0x0A8  </td><td> GPIO2_8    </td><td> lcd_data2         </td><td> 72   </td></tr>
<tr><td> P8_44 </td><td> 0x0AC  </td><td> GPIO2_9    </td><td> lcd_data3         </td><td> 73   </td></tr>
<tr><td> P8_45 </td><td> 0x0A0  </td><td> GPIO2_6    </td><td> lcd_data0         </td><td> 70   </td></tr>
<tr><td> P8_46 </td><td> 0x0A4  </td><td> GPIO2_7    </td><td> lcd_data1         </td><td> 71   </td></tr>
<tr><td colspan="5"></td></tr>
<tr><td>       </td><td> Offset </td><td> Name       </td><td> Mode0             </td><td> GPIO </td></tr>
<tr><td> P9_1  </td><td>        </td><td> GND        </td><td>                   </td><td>      </td></tr>
<tr><td> P9_2  </td><td>        </td><td> GND        </td><td>                   </td><td>      </td></tr>
<tr><td> P9_3  </td><td>        </td><td> DC_3.3V    </td><td>                   </td><td>      </td></tr>
<tr><td> P9_4  </td><td>        </td><td> DC_3.3V    </td><td>                   </td><td>      </td></tr>
<tr><td> P9_5  </td><td>        </td><td> VDD_5V     </td><td>                   </td><td>      </td></tr>
<tr><td> P9_6  </td><td>        </td><td> VDD_5V     </td><td>                   </td><td>      </td></tr>
<tr><td> P9_7  </td><td>        </td><td> SYS_5V     </td><td>                   </td><td>      </td></tr>
<tr><td> P9_8  </td><td>        </td><td> SYS_5V     </td><td>                   </td><td>      </td></tr>
<tr><td> P9_9  </td><td>        </td><td> PWR_BUT    </td><td>                   </td><td>      </td></tr>
<tr><td> P9_10 </td><td>        </td><td> SYS_RESETn </td><td> RESET_OUT         </td><td>      </td></tr>
<tr><td> P9_11 </td><td> 0x070  </td><td> UART4_RXD  </td><td> gpmc_wait0        </td><td> 30   </td></tr>
<tr><td> P9_12 </td><td> 0x078  </td><td> GPIO1_28   </td><td> gpmc_be1n         </td><td> 60   </td></tr>
<tr><td> P9_13 </td><td> 0x074  </td><td> UART4_TXD  </td><td> gpmc_wpn          </td><td> 31   </td></tr>
<tr><td> P9_14 </td><td> 0x048  </td><td> EHRPWM1A   </td><td> gpmc_a2           </td><td> 40   </td></tr>
<tr><td> P9_15 </td><td> 0x040  </td><td> GPIO1_16   </td><td> gpmc_a0           </td><td> 48   </td></tr>
<tr><td> P9_16 </td><td> 0x04C  </td><td> EHRPWM1B   </td><td> gpmc_a3           </td><td> 51   </td></tr>
<tr><td> P9_17 </td><td> 0x15C  </td><td> I2C1_SCL   </td><td> spi0_cs0          </td><td> 4    </td></tr>
<tr><td> P9_18 </td><td> 0x158  </td><td> I2C1_SDA   </td><td> spi0_d1           </td><td> 5    </td></tr>
<tr><td> P9_19 </td><td> 0x17C  </td><td> I2C2_SCL   </td><td> uart1_rtsn        </td><td>      </td></tr>
<tr><td> P9_20 </td><td> 0x178  </td><td> I2C2_SDA   </td><td> uart1_ctsn        </td><td>      </td></tr>
<tr><td> P9_21 </td><td> 0x154  </td><td> UART2_TXD  </td><td> spi0_d0           </td><td> 3    </td></tr>
<tr><td> P9_22 </td><td> 0x150  </td><td> UART2_RXD  </td><td> spi0_sclk         </td><td> 2    </td></tr>
<tr><td> P9_23 </td><td> 0x044  </td><td> GPIO1_17   </td><td> gpmc_a1           </td><td> 49   </td></tr>
<tr><td> P9_24 </td><td> 0x184  </td><td> UART1_TXD  </td><td> uart1_txd         </td><td> 15   </td></tr>
<tr><td> P9_25 </td><td> 0x1AC  </td><td> GPIO3_21   </td><td> mcasp0_ahclkx     </td><td> 117  </td></tr>
<tr><td> P9_26 </td><td> 0x180  </td><td> UART1_RXD  </td><td> uart1_rxd         </td><td> 14   </td></tr>
<tr><td> P9_27 </td><td> 0x1A4  </td><td> GPIO3_19   </td><td> mcasp0_fsr        </td><td> 125  </td></tr>
<tr><td> P9_28 </td><td> 0x19C  </td><td> SPI1_CS0   </td><td> mcasp0_ahclkr     </td><td> 123  </td></tr>
<tr><td> P9_29 </td><td> 0x194  </td><td> SPI1_D0    </td><td> mcasp0_fsx        </td><td> 121  </td></tr>
<tr><td> P9_30 </td><td> 0x198  </td><td> SPI1_D1    </td><td> mcasp0_axr0       </td><td> 122  </td></tr>
<tr><td> P9_31 </td><td> 0x190  </td><td> SPI1_SCLK  </td><td> mcasp0_aclkx      </td><td> 120  </td></tr>
<tr><td> P9_32 </td><td>        </td><td> VADC       </td><td>                   </td><td>      </td></tr>
<tr><td> P9_33 </td><td>        </td><td> AIN4       </td><td>                   </td><td>      </td></tr>
<tr><td> P9_34 </td><td>        </td><td> AGND       </td><td>                   </td><td>      </td></tr>
<tr><td> P9_35 </td><td>        </td><td> AIN6       </td><td>                   </td><td>      </td></tr>
<tr><td> P9_36 </td><td>        </td><td> AIN5       </td><td>                   </td><td>      </td></tr>
<tr><td> P9_37 </td><td>        </td><td> AIN2       </td><td>                   </td><td>      </td></tr>
<tr><td> P9_38 </td><td>        </td><td> AIN3       </td><td>                   </td><td>      </td></tr>
<tr><td> P9_39 </td><td>        </td><td> AIN0       </td><td>                   </td><td>      </td></tr>
<tr><td> P9_40 </td><td>        </td><td> AIN1       </td><td>                   </td><td>      </td></tr>
<tr><td> P9_41 </td><td> 0x1B0  </td><td> CLKOUT2    </td><td> xdma_event_intr1  </td><td> 20   </td></tr>
<tr><td> P9_42 </td><td> 0x164  </td><td> GPIO0_7    </td><td> eCAP0_in_PWM0_out </td><td> 7    </td></tr>
<tr><td> P9_43 </td><td>        </td><td> GND        </td><td>                   </td><td>      </td></tr>
<tr><td> P9_44 </td><td>        </td><td> GND        </td><td>                   </td><td>      </td></tr>
<tr><td> P9_45 </td><td>        </td><td> GND        </td><td>                   </td><td>      </td></tr>
<tr><td> P9_46 </td><td>        </td><td> GND        </td><td>                   </td><td>      </td></tr>
</table>

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
