---
title: "iOS shortcuts for microbin"
date: 2026-04-06T19:41:44+03:00
categories:
- self-hosting
- ios
tags:
- tutorial
- microbin
- pastebin

---

Hi there!

Recently, I've deployed [microbin](https://microbin.eu), minimalistic opensource "pastebin"-like service, into my homelab to share temporal files/text between devices. It works quite well.

A few days ago, I started wondering: could I upload files or paste text into Microbin directly from the iOS Share menu?

The short answer is yes. It could be done using iOS's builtin "Shortcuts".

<!--more-->

## Use cases

I have two general cases of using microbin from my corporate iPhone to my personal devices:

1. Share some text / link
2. Share file(s)

For simplicity, I decided to create dedicated shortcuts for each use case.

## Paste text into microbin

1. Open **Shortcuts** app and create new empty shortcut, rename it into "Paste into microbin".
2. Add **Share** action, click at *Input* and select **Shortcut Input**. New **Receive** action will pop up.
| ![Share action](/images/posts/ios-shortcuts/paste-1.png) | ![Share action](/images/posts/ios-shortcuts/paste-2.png) |  ![Receive action](/images/posts/ios-shortcuts/paste-3.png) |
|--------|-------|---|
3. Click at *Nowhere* in **Receive ... from** action, enable **Show in Share Sheet** toggle and disable **Show in Search**, click ✅ at the top right.
4. Next to the **if there's no input**, click *Continue* and select **Ask For** and **Text**
5. Delete **Share** action, we don't need it anymore.
| ![Share sheet](/images/posts/ios-shortcuts/paste-4.png) | ![Ask for Text](/images/posts/ios-shortcuts/paste-6.png) |  ![Delete share](/images/posts/ios-shortcuts/paste-13.png) |
|--------|-------|---|
6. Add **Get contents of URL** action.
7. Put your microbin url into *URL* field. For example `https://pub.microbin.eu/upload`
8. Expand action configuration by clicking ▶︎ button, then:

    - Set *Method* as **POST**
    - Add **Content-Type** header with value `multipart/form-data`
    - Select **Form** as *Request Body* type
    - Add **content** field as *File* type (important!) and select **Shortcut Input** as value.
| | ![Get contents of URL](/images/posts/ios-shortcuts/paste-10.png) | ![Configure URL](/images/posts/ios-shortcuts/paste-9.png) | |
|-|--------|-------|-|
9. That's all. Test it by clicking ▶︎ Play button in the bottom menu. If you did everything correctly you'll see preview with the link.
| | ![Play button](/images/posts/ios-shortcuts/paste-11.png) | ![Preview](/images/posts/ios-shortcuts/paste-12.png) | |
|-|--------|-------|-|

## Upload files into microbin

A shortcut for file upload is configured in very similar way. I won't describe it in detail but mention only notable changes:

- Duplicate the **Paste into Microbin** shortcut to not start from scratch.
- In **Receive** action, select **Files** type and allow *multiple files*
- Add **Repeat with each item** action loop to iterate over *Selected input*
- In **Get Contents of URL** action everything stays the same, except *content* field - it gets renamed into *file*

| | ![Play button](/images/posts/ios-shortcuts/files-1.PNG) | ![Preview](/images/posts/ios-shortcuts/files-2.PNG) | |
|-|--------|-------|-|

## Preview

| | |![Preview](/images/posts/ios-shortcuts/preview.PNG) | | |
|-|-|-|-|-|

## Summary

If you did everything from above and it did not work, or you don't want to bother with configuring it at all. You can import my shortcuts in your device and override microbin url:

- 📋 [Paste Into Microbin](https://www.icloud.com/shortcuts/8108f92bea454d20b5e68b8bd4f3c8db)

- 📤 [Upload Files to Microbin](https://www.icloud.com/shortcuts/1808fc6036e9436b8534781f7b182ab7)