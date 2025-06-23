---
title: "How to Use 3 Computers with One Monitor, Keyboard and Mouse – Without a KVM Switch"
date: 2025-06-23T15:00:00+03:00
categories:

- mac
- linux

tags:

- Tutorial
- Tips
- Mac
- Linux
- KVM
- DDC
---
Hi there! Today, I want to share how I organize my three-computer setup (MacBook Air, Mac mini, and Raspberry Pi) without a KVM switch, using a single keyboard, mouse, and monitor.

<!--more-->

## My devices

### Computers

- Macbook Air M2 14" – work Laptop.
- Mac Mini M1 – my personal computer.
- [ClockworkPi uConsole](https://www.clockworkpi.com/uconsole), based on [RPI CM4](https://www.raspberrypi.com/products/compute-module-4/) is a portable device for Linux tinkering.

### Peripheral devices

- The [UGREEN Vertical Wireless](https://www.amazon.com/UGREEN-Bluetooth-Ergonomic-Prevention-Compatible/dp/B0CM6FSPY3) - can connect up to three devices: Two via Bluetooth and one via a 2.4GHz USB dongle.
- The [Keychrone K15 Max](https://www.keychron.com/products/keychron-k15-max-alice-layout-qmk-wireless-custom-mechanical-keyboard?srsltid=AfmBOoo83yQiVI4ctH1noZzohk5v1fqAPMx_RbL86q7PxBWGvmSdUtw1) allows me to connect up to three devices via Bluetooth.
- The [Monitor Dell 27 4K UHD con USB-C (S2722QC)](https://www.dell.com/es-es/shop/monitor-dell-27-4k-uhd-con-usb-c-s2722qc/apd/210-bbrq/monitores-y-accesorios) with two HDMI inputs and one USB-C port which allows me to connect it up to three devices.

As you can see, all peripheral devices could connect to all my computers. At least in theory.

## Connecting devices

Historically my devices were numbered as follows:

1. uConsole
1. Macbook Air
1. Mac Mini

### Keyboard

The keyboard connection scheme is straightforward: uConsole connects via Bluetooth to the first device, the MacBook Air, as the second device, and the Mac Mini as the third device.

Later, I can use the `Fn + 1/2/3` hotkey to switch the keyboard between devices.

![Switching devices for keyboard](/images/posts/kvm/kvm-keyboard.png)

### Mouse

The first connection of the Ugreen mouse is reserved for the 2.4 GHz USB dongle. I plugged it into my uConsole, and then my MacBook Air and Mac mini connected as the second and third devices, respectively.

To switch devices with the mouse, press the red button on the bottom.

{{< youtube Dk_LNbkvECs >}}

### Monitor

The MacBook Air is connected to the USB-C port, which charges the computer.

The Mac Mini connects via HDMI-1, and the uConsole connects via HDMI-2.

#### Easiest way to switch devices

The easiest way to switch devices on the monitor is to press the small, round button in the bottom right corner to open the settings menu.

![Bottom corner buttons](/images/posts/kvm/kvm-monitor-2.jpg)

Then, choose the desired port.

![Monitor menu](/images/posts/kvm/kvm-monitor.jpg)

However, **"easiest"** does not mean **"most comfortable"**. Changing the monitor's input source this way requires pressing these small buttons several times. You also need to keep track of which devices are connected to HDMI-1 and HDMI-2.

#### The comfortable way to switch devices

Instead of pressing buttons on the monitor and navigating its menu, I could press a hotkey on my keyboard to trigger the monitor to change input sources via its API (Display Data Channel/Command Interface Standard (DDC/CI).).

I have explained in detail how one could do it on a Mac in my another post - [Monitor input source control on Mac](https://alikhil.dev/posts/monitor-input-source-control-mac/).

I won't go into detail here. I'll just say that on my Mac Mini and MacBook Air, I have configured the following:

**Mac Mini (#3)**

* On `CMD + F1` triggers switch to [uConsole](https://github.com/alikhil/configs/blob/master/raycast/switch-monitor-to-uconsole.sh)
* On `CMD + F2` triggers switch to [Macbook Air](https://github.com/alikhil/configs/blob/master/raycast/switch-monitor.sh)

**Macbook Air (#2)**

* On `CMD + F1` triggers switch to [uConsole](https://github.com/alikhil/configs/blob/master/raycast/switch-monitor-to-uconsole.sh)
* On `CMD + F3` triggers switch to [Mac mini](https://github.com/alikhil/configs/blob/master/raycast/switch-monitor.sh)

For **uConsole Linux machine (#1)** I used [ddcutil](https://github.com/rockowitz/ddcutil) and configured the following hotkeys:

* `Ctrl + F2` triggers switch to [Macbook Air](https://github.com/alikhil/configs/blob/master/linux/switch-to-macbook.sh)
* `Ctrl + F3` triggers switch to [Mac Mini](https://github.com/alikhil/configs/blob/master/linux/switch-to-mac-mini.sh)

## Switch sequence

Once all the devices are connected and the hotkeys are configured, the following sequence of actions is needed to switch to device `#N` (where N is 1, 2, or 3):

1. Switch the monitor input source by pressing `CMD + F#N`.
2. Switch the keyboard device by pressing `Fn + #N`.
3. Switch the mouse by pressing red button on the bottom of it until the indicator is under `#N` is blinking.

For example, let's assume I am currently using my third device – Mac Mini, to switch to Macbook Air (#2) I do:

1. Press `CMD + F2`
2. Press `Fn + 2`
3. Press red button on the bottom of the mouse until indicator under 2 is on.

Here is a demo video:

{{< youtube NgVESzIzpw4 >}}

## Pros and Cons

### Pros

* I don't need a special KVM switch to switch between devices.
* I use my keyboard to switch (for two-thirds of the KVM).

### Cons

* Switching the mouse takes longer since there is only one way iteration over the connected devices.
* I still need to manually reconnect other peripheral devices, e.g. web camera.

## Final thoughts

It has some limitations, but if you have a keyboard and mouse that support multiple connected devices, as well as a monitor, you won't need a special KVM switch to switch between computers.

Thank you for reading this!