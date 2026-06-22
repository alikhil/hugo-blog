---
title: "Goodbye, nix-darwin!"
date: 2026-06-22T20:00:00+02:00
description: "Why I stopped using nix-darwin on macOS after slow rebuilds, painful updates, installer script issues, and corporate tooling conflicts."
categories:
- mac
- note
tags:
- nix
- nix-darwin
- home-manager
- homebrew
- dotfiles
---

Two years ago, my work laptop was force-updated by the IT team and got broken so badly that I had to set it up from scratch. I was frustrated. As an engineer, I don't like repeating the same actions multiple times, and this gave me the motivation to set up my laptop as code.

<!--more-->

## Why I Tried Nix

Some of my friends and colleagues were using Nix. When I decided to take a look, it looked promising: NixOS as a fully declarative OS, the nixpkgs package repository, reproducible environments, and the idea that almost everything can be described as code. There was also support for macOS: nix-darwin and home-manager, including Homebrew integration.

I decided to give it a try.

I set up my work laptop with it. Later, when I bought a Mac mini for personal use, I was able to reuse most of the config there too. This was cool. I could keep my shell, packages, editor settings, and many small tools in one repository. It felt like I finally had a proper source of truth for my machines.

But it also made simple things more complicated.

## The Cost of Reproducibility

Before Nix, when I needed to add something new, I could install almost any app by running:

```bash
brew install <app>
```

With nix-darwin, the same action became a source code change followed by a slow rebuild command. Every change was tracked, reproducible, and easy to reuse on another machine. That was the whole point. But sometimes I just wanted to install a tool, try it for five minutes, and move on. Waiting for a rebuild each time made the whole setup feel heavier than I wanted.

Over time, the cost became more visible in three places: updates, breakages after updates, and tools that expected a normal mutable macOS home directory.

### Updates Took Too Long

Updating packages was probably the most annoying part. In my setup, I usually did not update one small app directly. I updated the whole Nix configuration: flake inputs, nixpkgs, nix-darwin, home-manager, and then applied everything with `darwin-rebuild switch`.

A small update could turn into a 30, 40, or 50 minute process. Nix had to evaluate the configuration, fetch new package versions, download or build what changed, update Homebrew packages through the generated Brewfile, and activate the new generation. I would start with "I need a newer version of this app" and end up maintaining my whole laptop.

### Updates Broke Things

A new nixpkgs revision could change package options, rename something, move a config path, or slightly change how a module worked. Home Manager and nix-darwin also had their own options and behavior that changed over time.

So after a big update, I often had to read error messages, search through changelogs or GitHub issues, and patch my config before the system could switch to the new generation. The rollback story is nice, and it is one of the strongest parts of Nix, but I still had to spend time understanding why the new generation did not build or activate. Package updates started to feel like small migration projects.

### New Releases Were Often Late

nixpkgs itself was also inconvenient for me. It is a huge community-maintained package repository, and I respect the amount of work behind it, but new releases do not always appear there quickly. Sometimes the package I needed was behind the latest version. Sometimes it was missing a feature that had already been released upstream. And sometimes installing the latest version was possible, but required overrides, flakes, or other Nix-specific work that I did not want to do for a simple desktop app.

### Installer Scripts Did Not Like My Setup

This problem became much more visible during the last half year, when many AI tools started appearing. A lot of these tools are distributed through installer scripts: curl this shell script, run it, let it modify your shell config, add something to `.zshrc`, install a binary somewhere, and maybe patch your environment. On a normal macOS setup, this usually works.

But my shell config was managed by Nix and Home Manager. Some files were symlinks to generated files from the Nix store, some paths were not supposed to be edited manually, and the installer scripts had no idea about that. They tried to modify files that were managed elsewhere, failed, or produced broken changes.

So while everyone else could try a new tool in a minute, I often had to stop and translate its installer into Nix config first. The setup stopped feeling helpful and started feeling like extra work.

## The Breaking Point

The breaking point happened when I changed my job.

I tried to set up my new corporate laptop with Nix, but the company's internal tooling wasn't ready for that at all. Some tools expected the default macOS layout. Some scripts assumed Homebrew paths. Some security and management software did not play nicely with my setup. I spent some time trying to make it work, but it felt like I was fighting the company environment instead of doing my actual job.

So I gave up and set up the laptop manually, without Nix.

I kept using Nix at home for a while. But after that, my work and personal configurations diverged a lot. The main benefit of my setup was supposed to be reuse between machines. Once that disappeared, I had much less motivation to maintain the Nix config while still suffering from all the drawbacks described above.

## Moving Back to a Normal macOS Setup

Eventually I decided that enough was enough.

I pointed Codex to my Nix repository and gave it a task to de-Nix my Mac mini.

Codex wrote a few migration scripts. It helped me move Homebrew packages out of the Nix-managed setup and back into normal Homebrew. It also helped me extract shell configuration into regular dotfiles. My Homebrew is Nix-free now, and I can directly edit my `.zshrc` file again.

I'm still scared to remove the `/nix` directory though. It stays there for now. I have no problem with that.

## Final Thoughts

nix-darwin is an interesting tool. I still understand why people like it. Having your machine described as code is a nice concept.

But for me, it was not user-friendly enough for daily macOS usage.

I don't want to debug my laptop configuration every time I need a new app. I don't want to wait for a rebuild just to install a small tool. And I don't want my personal setup to fight with corporate tooling.

At this point, I would rather reinstall my system from scratch manually than spend 30 minutes installing a new Nix package.
