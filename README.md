EClet by Riddle&Code
=====

[![Build Status](https://travis-ci.org/cryptotronix/EClet.png)](https://travis-ci.org/cryptotronix/EClet)
<a href="https://scan.coverity.com/projects/4781">
  <img alt="Coverity Scan Build Status"
       src="https://scan.coverity.com/projects/4781/badge.svg"/>
</a>

Extended driver for the Cryptotronix EClet

Status
---
This software is in ***BETA***. I have tested the below commands, but some there are some features and documentation that I would like to finish. If you use this software, it will configure your ECC108 in a non-reversible way. It will allow you to sign and verify with P256 keys but future features may be incompatible.

Releases
-----

You can download the original release [here](https://github.com/cryptotronix/EClet/releases/download/0.1.1/eclet-0.1.1.tar.gz). You will also need [this](https://github.com/cryptotronix/libcrypti2c/releases/download/v0.2/libcryptoauth-0.2.tar.gz) release of libcryptoauth.

Building
----

Install build-essential, autotools-dev, automake, autoconf, libtool, libxml2-dev, check, texinfo, and libgcrypt (libgcrypt11-dev on Debian variants)

Run `./autogen.sh`. This will generate the required README file from README.md as well as installing the required ***BETA*** version of [libcryptoauth-0.2](https://github.com/cryptotronix/libcrypti2c) (`sudo` is used to install this library)

You can run `eclet` locally, or install it to the system by running `sudo make install` 

In case this library is used on Raspberry Pi boards I2C functionality has to be activated first.

The I2C bus allows multiple devices to be connected to your Raspberry Pi, each with a unique address, that can often be set by changing jumper settings on the module. It is very useful to be able to see which devices are connected to your Pi as a way of making sure everything is working.

Run the following commands in the Terminal to install the i2c-tools utility.

```
1. sudo apt-get install -y python-smbus
2. sudo apt-get install -y i2c-tools
```

Run sudo raspi-config and follow the prompts to install i2c support for the ARM core and linux kernel

Go to Interfacing Options

<img alt="Raspberry Kernel Interface Options"
       src="https://github.com/RiddleAndCode/EClet/blob/master/doc/Screen%20Shot%202018-05-15%20at%2010.24.24.png?raw=true"/>

then I2C

<img alt="Raspberry Kernel I2C Select"
       src="https://github.com/RiddleAndCode/EClet/blob/master/doc/Screen%20Shot%202018-05-15%20at%2010.24.50.png?raw=true"/>

Enable!

<img alt="Raspberry Kernel I2C Confirm"
       src="https://github.com/RiddleAndCode/EClet/blob/master/doc/Screen%20Shot%202018-05-15%20at%2010.25.14.png?raw=true"/>
       
<img alt="Raspberry Kernel I2C Confirmed"
       src="https://github.com/RiddleAndCode/EClet/blob/master/doc/Screen%20Shot%202018-05-15%20at%2010.25.22.png?raw=true"/>

Then reboot!

*We also recommend going through the steps below to manually check everything was added by raspi-config!*

Now when you log in you can type the following command to see all the connected devices

```
1. i2cdetect 1
```

In  case this doesn't work ddd the following line to /boot/config.txt

```
1. dtparam=i2c_arm=on
```

Add the following line to /etc/modules

```
1. i2c-dev
```

Reboot

Once I2C is activated mount the secure element onto the Raspberry Pi board according to the image

<img alt="Riddle&Code secure element on Raspberry Pi"
       src="https://github.com/RiddleAndCode/EClet/blob/master/doc/CryptoBerry.png?raw=true"/>

Hardware
---

Riddle&Code built a new hardware secure element.

<img alt="Riddle&Code Secure Element"
       src="https://github.com/RiddleAndCode/EClet/blob/master/doc/IMG_0416.jpg?raw=true"/>
<img alt="Riddle&Code Secure Element Pinout"
       src="https://github.com/RiddleAndCode/EClet/blob/master/doc/SE%20(Pinout).png?raw=true"/>       
       

<img alt="Riddle&Code Secure Element"
       src="https://github.com/RiddleAndCode/EClet/blob/master/doc/SE%20(Pinout).png?raw=true"/>  

It replaces Josh Datko's original hardware.

The Hardware folder has an example board layout. This software will also work on the [CryptoCape](https://www.sparkfun.com/products/12773).

Running
---

see `./eclet --help` for full details.  The default I2C bus is
`/dev/i2c-1` and this can be changed with the `-b` option.

Kernel option
---

If you build
[libcryptoauth](https://github.com/cryptotronix/libcrypti2c) with the
`-DUSE_KERNEL` flag and install the kernel module, this utility will
use that module if you pass in: `-b /dev/atsha0`.

Root
---

You'll need to run as root to access `/dev/i2c*` initially.  You can change this by adding your user to the `i2c` group with:

`sudo usermod -aG i2c user`

Or:

`sudo chmod o+rw /dev/i2c*`


Currently supported commands:

### state
```bash
eclet state
Factory
```

This is the first command you should run and verify it's in the Factory state.  This provides the assurance that the device has not been tampered during transit.

### personalize
```bash
eclet personalize
```

This is the second command you should run.  On success it will not output anything. It configures all slots (0-16) to be holders for P-256 ECC private keys, except slot 8, which is reserved for future use. Keys are not generated at this time. Each key must be individually generated with the `gen-key` command.

***WARNING***

Until you personalize your device, the random number generator will produce a fixed test patterns of FFs and 00s. This is by design. However, it can be a bit suprising to see if you aren't expecting it.

### random
```bash
eclet random
62F95589AC76855A8F9204C9C6B8B85F06E6477D17C3888266AEE8E1CBD65319
```
### serial-num
```bash
eclet serial-num
0123XXXXXXXXXXXXEE
```
X's indicate the unique serial number.

### gen-key
```bash
eclet gen-key
04EED1CB629CF87F8BF6419986F990B92EA3DFA14CDAF70EB3E8DA8F9C9504DBC5B040D6480E88F895E9E1D4477970329B060450C80E1816EFED7B0FA49868CAEB
```

The device will internally create an P-256 ECC key and return the public key. The format of the public key is 0x04 + X + Y. Specify which slot to create a key (0-7, 9-15) with the `-k` option. Currently running this command multiple times will overwrite the public key, see this [issue](https://github.com/cryptotronix/EClet/issues/1).

### get-pub
```bash
eclet get-pub
04EED1CB629CF87F8BF6419986F990B92EA3DFA14CDAF70EB3E8DA8F9C9504DBC5B040D6480E88F895E9E1D4477970329B060450C80E1816EFED7B0FA49868CAEB
```
One can get the public key at any time calling the command get-pub. In case the public key of a specific slot has to be called use the parameter -k.

```bash
eclet get-pub -k 0
04EED1CB629CF87F8BF6419986F990B92EA3DFA14CDAF70EB3E8DA8F9C9504DBC5B040D6480E88F895E9E1D4477970329B060450C80E1816EFED7B0FA49868CAEB
```

### sign
```bash
eclet sign -f ChangeLog
3BAEB5705D8765B34B389F1768BAC783FCA786AB64A760D10DD133C86E5892A7A790E424C8E1540551C99FBE4F9F531B504A6004F08F3E0D4E42E96BBDE5C179
```

Performs an ECDSA signature. Data can be specified as a file with the `-f` option or passed via `stdin`. The data will be SHA256 hashed prior to signing. The result is the signature in the format: R + S.

### verify
```bash
eclet verify -f ChangeLog --signature C650D1A30194AD68F60F40C321FB084F6177BEDAC74D0F0C276ED35B00249AC8CF3E96FB7AB14AA48223FBA2E5DD9BCAE232BF963755C42F8FD9BD77FC145D41 --public-key 049B4A517704E16F3C99C6973E29F882EAF840DCD125C725C9552148A74349EB77BECB37AA2DB8056BAF0E236F6DCFEC2C5A9A0F23CEFD8A9DC1F4693718E725D2
```

Verifies an ECDSA signature using the device. You specify the data (which will be SHA256 hashed), the signature (R+S), and the public key (0x04+X+Y). Returns a `0` exit code on success.

### offline-verify-sign
```bash
eclet offline-verify-sign -f ChangeLog --signature C650D1A30194AD68F60F40C321FB084F6177BEDAC74D0F0C276ED35B00249AC8CF3E96FB7AB14AA48223FBA2E5DD9BCAE232BF963755C42F8FD9BD77FC145D41 --public-key 049B4A517704E16F3C99C6973E29F882EAF840DCD125C725C9552148A74349EB77BECB37AA2DB8056BAF0E236F6DCFEC2C5A9A0F23CEFD8A9DC1F4693718E725D2
```

Same as `verify` except it *does not* use the device and can be run on a system with one. It uses the software ECDSA implementation provided by `libcrypti2c`.

Options
---

Options are listed in the `--help` command, but a useful one, if there are issues, is the `-v` option.  This will dump all the data that travels across the I2C bus with the device.

Support
---

IRC: Join the `#cryptotronix` channel on freenode.

Mailing lists: `hashlet-announce` and `hashlet-users` are open for subscriptions [here](https://savannah.nongnu.org/mail/?group=hashlet).

![GPLv3](https://www.gnu.org/graphics/gplv3-127x51.png)
