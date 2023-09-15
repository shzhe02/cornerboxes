---
title: "[Part 3] Adding a Dongle"
date: 2023-09-15T16:19:34+03:00
draft: false
---
Note: This isn't going to be an explicit guide on how to add a dongle to any ZMK board. I will create a post for that later.
# Background

I've since daily driven the Swan40 for quite a while, and it's been working perfectly... on my desktop. Since it's been working out so well, I wanted to make another one to have on-the-go, for use with my laptop. However, this is where some problems appeared.

For some context, my laptop is running a dual-boot configuration, with Windows 11 and Fedora (Kinoite 38). I primarily use Linux and only use Windows due to certain pieces of software not working great on Linux. It has an Intel AX200 combo wireless adapter (WiFi 5 + Bluetooth 5.2).
## No connectivity

Due to the Swan40 being a ZMK keyboard, it's main communication method with any computer is over BLE. However, when I tried to pair my laptop to my Swan40, the keyboard simply did not show up in the Bluetooth scans. The Bluetooth scans showed all other devices, just not my keyboard.

## Troubleshooting

I tried the following:
- Connecting on Windows
	- This worked, but I rarely use Windows. This does confirm that the adapter is capable of picking up BLE devices.
- Unpairing on Windows and attempting to pair on Linux
	- This did not work.
- Using Blueman, Bluez-tools, Bluetuith
	- Bluetuith was suggested by [this comment in a GitHub issue about Bluetooth connectivity](https://github.com/zmkfirmware/zmk/issues/1487#issuecomment-1625394710)
	- This also did not work, but it eliminates the possibility that KDE's Bluetooth settings are broken.

At this point, I'd given up on finding an easy solution for fixing the connectivity issues.

It was time for a more drastic and experimental option.
# Using a Dongle

I had stumbled upon [Xudong Zheng's ErgoBlue 2 keyboard](https://www.xudongz.com/blog/2020/ergoblue/) which seems to use an nRF52840 dongle running ZMK to act as an intermediary between the keyboard halves and the computer, allowing the wireless keyboard to appear as a wired one in the computer.

## Why?

After looking into the ErgoBlue and similar boards, I discovered the following pros and cons:

Pros:
- Power savings and even battery drain.
	- Both keyboard halves only need to send data, in contrast to the traditional wireless split where one half has to receive data from the other half and transmit data to the computer. (Assuming both halves now act as purely peripheral halves, the [ZMK power profiiler] estimates there will be approximately a 6-8x battery life improvement.)
	- Since both halves are essentially doing the same thing now, the power drain on each half is far more even, meaning I can charge both halves at the same time every time either is running low.
- No more Bluetooth pairing shenanigans.
	- All communication between the computer and keyboard occurs over USB with the dongle, and the dongle auto-pairs with the keyboard.
	- This makes the keyboard basically plug-and-play.

Cons:
- The dongle is rather bulky.
	- Unlike other wireless dongles I have (that use proprietary 2.4Ghz protocols), the nRF52840 dongle is really large since it has lots of pins which could be used. This all goes unused as I'm solely using it as a wireless dongle.
- The dongle is super exposed.
	- All the little components and pads on the board are really exposed, which means it's rather susceptible to the elements. Not great for something I'll be carrying all over the place.
- I now have to carry around an easy-to-lose dongle.

## Dongle-ifying the Swan40

So, I got myself a pair of nRF52840 dongles (PCA100059, v2.1.1) for about 9€ a piece.

Unfortunately, as BLE dongles fro ZMK are still quite obscure, a tutorial doesn't really exist. I used the following projects as configuration reference:
- [Enki42 with dongle config](https://github.com/aroum/zmk-enki42-dongle)
	- This uses a nice!nano v2 as the dongle, but it did help with the setting up the Kconfig files.
- [Onekey](https://github.com/jibingeo/zmk-config-onekey)
	- This uses only an nRF52840 dongle as its board. I needed this as a reference since it appears ZMK does not have a board config for this.
	- It turns out that ZMK allows for use of boards supported by Zephyr, not just the ones in ZMK. The PCA100059 board is marked as `nrf52840dongle_nrf52840` in Zephyr.
	- This also showed me the directory configuration for a custom board configuration, which seems to be missing from the ZMK docs.

At this point, I had successfully gotten the ZMK GitHub Action to export the firmware for the halves and the dongle. This took about 5 hours and... well...

![failures](1.webp)

Fun.

### Risky Stuff

Unfortunately, I don't have any debuggers or extra equipment I can use to flash my dongle with a UF2 bootloader, so I was forced to use the nRF Connect software to flash stuff onto it with the DFU bootloader. However, this software only allows me to flash .hex files onto the board, while the ZMK GitHub Action exported a .bin file (even though I set the config to export a .hex file :/).

I'd like to note that I have basically no idea what .bin or .hex files are besides the fact that they contain software, which when flashed to some board makes cool stuff happen.

Here's where things got a little risky. I decided to blindly search up how to convert .bin to .hex, and ChatGPT told me that `srec_cat` was something I could look into. [This article (?)](https://carta.tech/man-pages/man1/srec_examples.1.html) helped me figure out that the command I was looking for was:

```
srec_cat swan40_dongle-nrf52840dongle_nrf52840-zmk.bin -binary -o dongle.hex -intel 
```

(By the way, I used srec_cat on Windows as nRF Connect didn't seem to want to play well on Linux.)

From my attempts at trying to load the Adafruit UF2 bootloader onto my dongle, I found out that I cannot overwrite the MBR section of the dongle, which goes from address 0x00000000 to 0x00000FFF, meaning that whatever I wanted to write had to start at 0x00001000. The `srec_cat` command I stated above would make ZMK start at 0x00000000, overlapping with the MBR section. So, I figured out that I needed to apply an offset, which I did by modifying the `srec_cat` command as follows:

```
srec_cat swan40_dongle-nrf52840dongle_nrf52840-zmk.bin -binary -offset 0x00001000 -o dongle.hex -intel 
```

Somehow, that actually worked, and by some miracle, i didn't have to brick anything in the process of figuring that out.

### Troubleshooting

Right off the bat, the keyboard just refused to connect to the dongle. I later discovered that I need to have an `#include "Swan40.dtsi"` even for the dongle overlay, otherwise things just did not want to pair.

After getting the dongle to actually attempt to pair with the keyboard halves, it turns out the keyboard only works if I flash the settings_reset UF2 firmware onto the halves before loading the actual firmware, but the halves immediately failed to connect the moment the dongle was unplugged and replugged into the computer.

With some help from @bravekarma and @petejohanson on the ZMK Discord server, it turns out that (supposedly) the keymap was only being stored in the dongle's memory, causing it to be erased when unpowered. After taking some of the settings from the [nRF52840-DK ZMK board configuration](https://github.com/zmkfirmware/zmk/blob/main/app/boards/nrf52840dk_nrf52840.conf), the dongle now works as a plug-and-play device.

# The End?

This marks the end of the Swan40 v1. Oh, but what's that? I smell a v1.1...