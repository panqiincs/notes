# How to install linux driver for wireless card

## lsusb

lsusb
Bus 003 Device 002: ID 0bda:818b Realtek Semiconductor Corp.

## Google

Realtek RTL8192EU ID 0BDA:818B

## Download source

git clone https://github.com/Mange/rtl8192eu-linux-driver

## Install kernel source

zypper in kernel-devel

## Install

cd rtl8192eu-linux-driver
make && make install
