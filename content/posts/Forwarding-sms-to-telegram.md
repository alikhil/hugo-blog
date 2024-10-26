---
title: "Forwarding SMS to Telegram"
date: 2024-10-26T14:11:43+03:00
categories:
- linux
- selfhosting
- opensource
tags:
- sim
- sms
- telegram

---

After extensive travel, I’ve accumulated several mobile numbers and, naturally, physical SIM cards. Switching them out each time became tedious, even after buying a basic Nokia with two SIM slots, which only helped temporarily. When a friend asked if I could set up a Spanish number for account registrations, I realized it was time to automate the process.

If you’re dealing with multiple SIM cards and want to receive SMS in Telegram, I have a straightforward approach. You’ll need a Linux machine that’s always online, connected to the internet, and about $10.

<!--more-->

## What You’ll Need

- A physical SIM card
- A USB modem that’s supported by the [Gammu library](https://wammu.eu/phones/)
- A Telegram bot token, chat or channel ID
- A Linux machine with a free USB port
- Docker and Docker Compose installed

## Finding the Right Modem

> If you have a USB modem at home, check if it’s supported by [Gammu](https://wammu.eu/phones/).

For our purposes, we don’t need an expensive 4G modem with advanced features. Any basic 2G/3G modem will work, and these are easy to find at a discounted price on sites like eBay or Wallapop.

Search for “Huawei USB modem,” sort by price, and look for unlocked options or ones with compatible firmware.

For instance:
![Ebay Screenshot](/images/posts/sms-to-telegram/ebay.png)

Next, go to the [Gammu](https://wammu.eu/phones/) website and look up the device. Make sure it appears on the list and that “SMS” is included in the "Supported features" column:

![E3131](/images/posts/sms-to-telegram/e3131.png)

If the device meets these requirements, it’s good to go!

## Setup Instructions

> Before starting the setup, it’s best to connect the modem with the SIM card already inserted to your PC and check that it’s functioning properly.

### Identify Device Path

Run the following command to identify the device path:

```bash
tree /dev/serial/by-id/
```

You should see a paths similar to:

```shell
/dev/serial/by-id
├── usb-HUAWEI_HUAWEI_Mobile-if00-port0 -> ../../ttyUSB0
├── usb-HUAWEI_HUAWEI_Mobile-if02-port0 -> ../../ttyUSB1
├── usb-HUAWEI_HUAWEI_Mobile-if03-port0 -> ../../ttyUSB2
```

Choose a path that ends with `ttyUSB0`, in my case it's `/dev/serial/by-id/usb-HUAWEI_HUAWEI_Mobile-if00-port0`.


### Running the Service

Using Docker Compose, set up your configuration:

```yaml
services:
  gammu:
    build:
      context: https://github.com/kutovoys/sms-to-telegram.git#main
      dockerfile: Dockerfile.alpine
    volumes:
    - type: bind
      source: /dev/serial/by-id/usb-HUAWEI_HUAWEI_Mobile-if00-port0 # Change this to your device path
      target: /dev/modem
    privileged: true
    environment:
      - BOT_TOKEN=<put your telegram bot token here>
      - PIN=<your sim card pin>
      - CHAT_ID=<telegram chat/channel ID>
      - DEVICE=/dev/modem
    cap_add:
    - NET_ADMIN
    - SYS_MODULE
```

Save the configuration to a `docker-compose.yml` file and run:

```bash
docker compose up -d
docker compose logs -f gammu
```

If everything is set up correctly, you should see the following log messages:

```shell
gammu-1  | Fri 2024/10/04 17:50:56 gammu-smsd[12]: Created POSIX RW shared memory at 0x7fcf90b21000
gammu-1  | Fri 2024/10/04 17:50:56 gammu-smsd[12]: Starting phone communication...
gammu-1  | Fri 2024/10/04 17:55:30 gammu-smsd[12]: Ignoring incoming SMS info as not a Status Report in SR memory.
gammu-1  | Fri 2024/10/04 17:55:33 gammu-smsd[12]: Read 1 messages
gammu-1  | Fri 2024/10/04 17:55:33 gammu-smsd[12]: Received IN20241004_195517_00_Celerity_00.txt
gammu-1  | Fri 2024/10/04 17:55:33 gammu-smsd[13]: Starting run on receive: /etc/sms_to_telegram.sh IN20241004_195517_00_Celerity_00.txt
gammu-1  | Fri 2024/10/04 17:55:33 gammu-smsd[12]: Process finished successfully
```

To test SMS reception, you can use free online SMS-sending services (search for "send SMS for free") or try logging into Telegram, your bank account, etc.

![Telegram Screenshot](/images/posts/sms-to-telegram/screenshot.png)

## How It Works

The [Gammu library](https://wammu.eu/libgammu/) provides a unified interface for working with phones and modems from various manufacturers.

On top of that, there's the [Gammu SMS Daemon](https://wammu.eu/smsd/), which receives SMS messages and triggers a custom script—in our case, [script](https://github.com/kutovoys/sms-to-telegram/blob/main/sms_to_telegram.sh) to send the messages to Telegram.

## Final Thoughts

Thanks to [@kutovoys](https://github.com/kutovoys) for the idea and Docker image!

This is a simple, affordable, and scalable solution—especially if you’re into self-hosting.

> This post was originally written for [vas3k.club](https://vas3k.club/post/25926/).
