+++
title = "Getting Nushell to run in an Alpine Linux container"
date = 2023-07-11T13:11:22+03:00
tags = [ "Guide" ]
draft = false
+++

From my post about my digital setup, I wanted to quickly write a small post about getting nushell and starship working in an Alpine Linux [Toolbx](https://containertoolbx.org/) container.

So, the problem is that Alpine Linux uses the musl C library instead of glibc. This causes a compatibility problem when running Rust programs that were compiled on the main system within the Alpine Linux container. In addition, the PATH environment variable is inherited from the host system due to how Toolbx works.

Here's the workaround I found that seems to work:
- Inside the Alpine Linux container:
	- Enable the `testing` repository
	- Install both starship and nushell using the apk package manager.
	- In the file `~/.cache/starship/init.nu`, replace all occurrences of `^/home/<user>/.cargo/bin/starship` to just `starship`, and make sure that the starship executable is in the system path.
- On the host system:
	- Find the cargo env file located at `~/.cargo/env`
	- Edit the line with `export PATH` such that it appends the `~/.cargo/bin` directory to the path instead of prepending it.
		- As the container will inherit the PATH from the host system, the `nu` and `starship` executables within the `~/.cargo/bin` directory will be deprioritized, allowing the system packages to take priority.

However, one problem with this workaround is that when using `toolbox enter <alpine linux container name>` from nushell on the host system, it will try continuing to use the same nushell executable that's currently being used by the host system within the container. To work around this, you should first create a bash session with `bash`, then entering your toolbx container, and then using `nu` to start up nushell.

P.S. Since this is me trying to backtrack what I did, I may have left out some things. If it doesn't work out for you, please make a comment.
