+++
title = "Running LLMs on Fedora 42 Kinoite with an AMD GPU using Ollama"
date = 2025-07-13T13:19:29+03:00
tags = [ "Fedora 42 Kinoite", "Guide" ]
draft = false
+++

This is a short guide to getting Ollama working with an AMD GPU on a Fedora 42 Kinoite system. In practice, the methods discussed should work on any Fedora 42 immutable variant, including but not limited to Fedora 42 Silverblue.

When attempting to install ROCm on Fedora 42 Kinoite (I'll be referring to this as F42K from now on), I encountered several issues:
1. The [installation guide for ROCm](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/quick-start.html) does not include a Fedora version.
2. Attempting to install the RHEL version leads to an error message stating it does not support the current distribution.
3. Fedora packages [does have](https://packages.fedoraproject.org/pkgs/rocm/rocm/) a rocm meta package, however, it is currently only supported on Rawhide or EPEL 10.1 (which isn't recommended on Fedora systems).
4. Rebasing from F42K to Fedora Kinoite Rawhide is not feasible due to some layered packages having dependency issues due to missing packages in the Rawhide package repository. 

Therefore, we need a workaround for installing ROCm: containers! 

# Method 1: Using Ollama with Alpaca

For users of Fedora Immutable, this method will come most naturally; we're going to use [Toolbx](https://containertoolbx.org/) (which is preinstalled on Fedora Immutable variants) to get ROCm installed.

Installing ROCm in a Toolbx container can be done as follows:

1. Create your toolbox with `toolbox create <name>`
2. Enter your toolbox with `toolbox enter <name>`
3. Install the RHEL version of the AMDGPU installer within the toolbox with `sudo dnf install https://repo.radeon.com/amdgpu-install/6.4.1/rhel/9.6/amdgpu-install-6.4.60401-1.el9.noarch.rpm` (or follow AMD's official instructions [here](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/install-methods/amdgpu-installer/amdgpu-installer-rhel.html))
4. Run `amdgpu-install --no-dkms --usecase=rocm -y`

Now, installing Ollama can be done as follows:

1. Either download the latest Ollama version from [here](https://ollama.com/download/ollama-linux-amd64.tgz) or use `curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz`
2. Extract the archive somewhere.
3. Add the `bin` directory inside into your PATH.

Running Ollama can now be done with `ollama serve` within your toolbox. It should properly detect and use AMD GPUs.

For the frontend, I like to use [Alpaca](https://flathub.org/apps/com.jeffser.Alpaca) with this method, as the user interface is pleasant to use. Within Alpaca, you just need to go to the "Manage Instances" page and add Ollama as an instance. Select the one that is just "Ollama", not "Ollama (Managed)". The default settings should work just fine.

However, there are some downsides to using Alpaca, which is that thinking models generally don't seem to actually think (only Phi 4 Reasoning seemed to actually think from my testing). Additionally, web search seems to be broken as it does so with solely DuckDuckGo, which constantly gives me a rate limit error when I attempt to use it.

Well, what if I want a slightly more versatile user interface and a more seamless setup?

# Method 2: Using Ollama and Open WebUI (with Podman Compose)

There exists another very popular user interface for running LLMs known as Open WebUI. However, unlike Alpaca, it requires a server to be run, usually in the form of a Podman/Docker container. For this method, I'll be using Podman as it's preinstalled with F42K, but using Docker with the same approach should be doable.

It should also be noted that if we use Podman directly, we can utilize the `ollama:rocm` image, allowing us to skip the steps needed to install ROCm ourselves.

To make things even simpler to set up, I'll be using Podman Compose, which allows the creation of containers using a YAML file, making things more deterministic.

My `compose.yml` file is as follows:

```yml
version: "3.8"

services:
  ollama:
    image: docker.io/ollama/ollama:rocm
    container_name: ollama
    restart: unless-stopped
    volumes:
      - ollama:/root/.ollama:Z
    networks:
      - common
    devices:
      - /dev/kfd:/dev/kfd
      - /dev/dri:/dev/dri
    security_opt:
      - label=type:container_runtime_t
            
  openwebui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: openwebui
    restart: unless-stopped
    ports:
      - 3000:8080
    depends_on:
      - ollama
    volumes:
      - openwebui_data:/app/backend/data:Z
    networks:
      - common
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434

networks:
  common:
volumes:
  ollama:
  openwebui_data:
```

(Special thanks to [this comment](https://discussion.fedoraproject.org/t/fedora-silverblue-ollama-with-rocm-failed-to-check-permission-on-dev-kfd-open-dev-kfd-invalid-argument/131680/10) for solving the issue regarding device access.)

To use it:

1. Put the `compose.yml` file somewhere.
2. In the same directory, run `podman compose up -d`
3. If prompted to select a repository to download the images from, always select the docker one.
4. Now, if you go to `localhost:3000`, you should be able to access Open WebUI.
5. Create an admin account with an email address and password.
6. To add a model, simply go to the model selection area of the chat (should be on the top left), and in the "Search a model" field, type a model name from [Ollama's models](https://ollama.com/search). An option showing "Pull \<model> from Ollama.com" should appear, and if you gave a valid model name, it should download it upon clicking.

That should be all that's needed to get up and running with a model.

Something to note is that you can allow other devices you own to access Open WebUI by using [Tailscale](https://tailscale.com/). However, one issue with this is that you are required to spin up the compose file every time on your computer before being able to access it from your other devices.

What if we want to start the containers at boot? Doing so with Podman Compose is quite difficult, but there does exist an alternative that integrates more closely with systemd, enabling startup at boot: Podman Quadlets.

# Method 3: Starting Ollama and Open WebUI at Boot (with Podman Quadlets)

Podman Quadlets is like Podman Compose but managed by systemd. This allows our containers to be linked to systemd services, allowing them to be run at login time/boot (We'll get back to this later).

First, create the following files within the `~/.config/containers/systemd/` directory.

1. `ollama-webui.pod`

```
[Pod]
PodName=ollama-webui
PublishPort=3000:8080

[Install]
WantedBy=multi-user.target default.target
```

2. `ollama-webui-ollama.container`

```
[Container]
AddDevice=/dev/kfd:/dev/kfd
AddDevice=/dev/dri:/dev/dri
ContainerName=ollama
Image=docker.io/ollama/ollama:rocm
Pod=ollama-webui.pod
Volume=ollama.volume:/root/.ollama:Z
SecurityLabelType=container_runtime_t

[Service]
Restart=always
```

3. `ollama-webui-openwebui.container`

```
[Unit]
Requires=ollama-webui-ollama.service
After=ollama-webui-ollama.service

[Container]
ContainerName=openwebui
Environment=OLLAMA_BASE_URL=http://ollama:11434
Image=ghcr.io/open-webui/open-webui:main
Pod=ollama-webui.pod
Volume=openwebui.volume:/app/backend/data:Z

[Service]
Restart=always
```

4. `ollama.volume`

```
[Volume]
```

5. `openwebui.volume`

```
[Volume]
```

Your `~/.config/containers/systemd/` directory should now look like the following:

```
~/.config/containers/systemd
├── ollama.volume
├── ollama-webui-ollama.container
├── ollama-webui-openwebui.container
├── ollama-webui.pod
└── openwebui.volume
```

Next, run:
1. `systemctl --user daemon-reload`
2. `systemctl --user start ollama-webui-pod.service`

Open WebUI should now be accessible from `localhost:3000`. However, there is still one slight problem. The containers will only spin up at login time, not boot. To fix this, we need to run:

```
loginctl enable-linger <your user>
```

Now, Ollama and Open WebUI should launch at boot time, and combined with Tailscale, you may now use Open WebUI across devices in your Tailnet right after powering on your system.

**Edit**: Do note that your computer (that's running Ollama + Open WebUI) needs to be able to have network access at boot, which may not be trivial especially when using WiFi. In my case, using an ethernet connection seems to work just fine.
