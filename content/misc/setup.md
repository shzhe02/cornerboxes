---
title: "My Digital Setup"
date: 2023-07-11T12:27:05+03:00
draft: false
---

I really enjoy semi-obscure technologies, and I try implementing many of them into my own computer. So, I thought it would be a little fun to make a little write-up about what components my setup consists of and why they exist.

# Setup v1 (as of 2023-07-11)

(I'll add a diagram here later, but for now, here's a bullet-point representation)

- Main desktop
	- Running Fedora Kinoite 38
	- Runs 3 containers (via Toolbx)
		- Dev container
			- Running Fedora 38 (container version)
			- Contains all the additional packages I need for software development
		- Play container
			- Running Fedora 38 (container version)
			- Primarily for running games (on Steam)
		- Utils container
			- Running Alpine Linux Edge (container version)
- Home server (repurposed old laptop)
	- Running Rocky Linux 9
	- Runs 3 containers (Podman + podman-compose)
		- Nextcloud
		- Penpot
		- Forgejo
	- All 3 containers are connected to a subdomain of mine via Cloudflare Tunnels

As for some personal preferences:
- Preferred shell: [nushell](https://nushell.sh)
- Preferred terminal multiplexer: [zellij](https://zellij.dev)
- Preferred terminal emulator: [alacritty](https://alacritty.org)

---

# Some explanation

Now that you've seen what the setup consist of, I'll try walking through some of my choices.

## Distro choices?

### Main desktop

I went with Kinoite due to the following reasons:
- KDE is just what I'm used to.
- Containerized workflow
	- If I mess up setting up a new environment for whatever purpose, I can just delete the whole container and start a new one up from my last custom image.
- Atomic
	- I find the rpm-ostree package manager (?) really cool! I like how I can just pin versions of my system and reboot to them in the future if needed. This also brings about better system stability.
- Fedora-ish
	- Since prior to this, I've been using Fedora 36, 37 and 38, I wanted to still have something from the Fedora ecosystem. I also wanted to have the almost bleeding-edge package updates while maintaining pretty good stability (which is why I didn't go for RHEL or similar).
- Why not?
	- Heck, it's free! And if it's more complicated, that just means more things to learn :D

### Home server

I just wanted something stable so I could just chuck my laptop on a shelf and have it run 24/7 with no issues. From some research, it seemed like RHEL was the gold standard for stability, but since I like open-source, I went with one of its derivatives, Rocky Linux 9.

## What's with my preferences?

...I'm totally not biased towards applications built with Rust...

### Nushell

Nushell may not be the most well-supported shell out there, but since I am more of a casual Linux user, I care more about my experience more over raw utility. The main reasons why I just went with nushell is because of the super pretty error messages and the structured responses for many commands.

Additionally, I can couple this with [Starship](https://starship.rs) and have super pretty terminal prompts!

Extra: If I ever need to, I can always just type `bash` and do what I need to do.

### Zellij

I previously tried tmux, but didn't really find it very easy to use. I ended up discovering Zellij from a recommendation, and it basically lets me do everything I need it to do. All the controls are on-screen, so I don't really have to memorize them.

I'm also not really a multiplexer power user, so just the function of multiple panes is basically the only thing I need.

If I ever need better aesthetics, I also have the option to remove unnecessary portions of the interface to make it look super minimal like tmux.

### Alacritty

Although Konsole (the default emulator that comes with KDE Plasma) works, I wanted a little less clutter. Alacritty just works, feels very responsive, and only contains the terminal itself with no extra buttons.

