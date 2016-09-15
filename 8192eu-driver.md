# How to install WLAN card driver from source

I have a WLAN card **TL-WN823N**, it does not work properly under linux, I have to install driver manually.

## Step 1: Find out the chip set

Run `lsusb` you can get:

```bash
$ lsusb
Bus 003 Device 002: ID 0bda:818b Realtek Semiconductor Corp.
```
Google `0bda:818b`, you will know the USB stick bases on the Realtek chip set **RTL8192EU**:

```
Realtek RTL8192EU ID 0BDA:818B
```

## Step 2: Download source code

Clone it from [github](https://github.com/Mange/rtl8192eu-linux-driver):

```
$ git clone https://github.com/Mange/rtl8192eu-linux-driver
```

## Step 3: Compile and install 

You must install `kernel-devel` for compilation:

```bash
$ sudo zypper in kernel-devel
```

Compile and install:

```bash
$ cd rtl8192eu-linux-driver
$ make
$ sudo make install
```

## Step 4: Caution

After you update the kernel, the driver may not work, you may need to compile it again.
