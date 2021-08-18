---
title: "Lets Build Windows Boxes"
date: 2021-08-18T19:23:35+02:00
author: Friedrich Wilken
description: how to create a new post
tags: [ vagrant, vm, provisioning ]
categories: [ vagrant, vm, packer ]
series: [win-packer]
toc: false 
draft: false
---

F' it, I build my own windows server image (, to save time provisioning VMs).

First I forked [Stefan Scherers AWESOME packer-windows repo](https://github.com/StefanScherer/packer-windows) and `git clone`d it to my local machine. 

[We](https://github.com/kubernetes-sigs/sig-windows-dev-tools) used Stefan's `windows_2019` and `windows_2019_docker` vagrant boxes in the past, so I copied the `build_windows_2019_docker.sh` and `.json` and renamed them to `windows_2019_core_kubernetes`.

Next, I downloaded the *windows server 2019 essential eval* image from microsoft.

In the `.sh` I changed these three lines (the `iso_url` to the image I just downloaded.
```bash
...
  --only=virtualbox-iso \
  --var autounattend=./answer_files/2019_core/Autounattend.xml \
  --var iso_url=~/Downloads/my.iso \
 windows_2019_core_kubernetes.json
```
I have no clue right now why the orginal file was pointing to `./tmp/2019_core/Autounattend.xml` but since I got an error and could find the file localy, I just changed it.

`--only=vmware-iso` caused trouble on my system because I do not have the needed bins, even though I installed VMWare Workstation. This looks to be a common problem on linux and I will look into it later.

In the `.json` I added this `openssh.ps1`-script that I found in the repo to test how mess around with packer and because we need ssh anyhow.
```json
    ...
    {
      "scripts": [
        "./scripts/vm-guest-tools.ps1",
        "./scripts/debloat-windows.ps1",
        "./scripts/docker/set-winrm-automatic.ps1",
        "./scripts/openssh.ps1"
      ],
      "type": "powershell"
    },
    ...
```
and this from:
```json
"post-processors": [
    {
      "keep_input_artifact": false,
      "output": "windows_2019_docker_{{.Provider}}.box",
```
to
```json
      "output": "windows_2019_k8s_{{.Provider}}.box",
```
I gave it a try:
```bash
./windows_2019_core_kubernetes
```
But I got impatient with all the docker stuff. I first wanted to not touch it to not break anything. But I delted all these lines:
```json
{
      "environment_vars": [
        "docker_images={{user `docker_images`}}",
        "docker_provider={{user `docker_provider`}}",
        "docker_version={{user `docker_version`}}"
      ],
      "scripts": [
        "./scripts/docker/add-docker-group.ps1",
        "./scripts/docker/install-docker.ps1",
        "./scripts/docker/docker-pull.ps1",
        "./scripts/wait-for-tiworker.ps1",
        "./scripts/docker/open-docker-insecure-port.ps1",
        "./scripts/docker/open-docker-swarm-ports.ps1",
        "./scripts/docker/remove-docker-key-json.ps1",
        "./scripts/docker/disable-windows-defender.ps1"
      ],
      "type": "powershell"
    },
```
Again I ran
```bash
./windows_2019_core_kubernetes
```
And got a nice _little_ vagrant box!
```bash
==> Wait completed after 53 minutes 23 seconds

==> Builds finished. The artifacts of successful builds are:
--> virtualbox-iso: 'virtualbox' provider box: windows_2019_k8s_virtualbox.box
```
So, I added this to vagrant
```bash
vagrant box add killme 
```
(I name stuff killme so I later know it's okay to delete it.)

I created a new dir and in there I inited a new Vagrantfile 
```bash
vagrant init killme
```
and started it
```bash
vagrant up
```
and ssh'ed successful in there
```
vagrant ssh default
```

Cool, but this was only the proof that this works. Here is my todo list:
  - no more need to enter a password when ssh'ing into the box
  - automatically start powershell when ssh'ing in ([this might help](https://www.vagrantup.com/docs/vagrantfile/winssh_settings))
  - install choco; I think the repo already contains a script that I can just paste in the `.json`.
  - create a little choco "wish list": neovim, ... 
  - move all the scripts from [sig windows dev tools](https://github.com/kubernetes-sigs/sig-windows-dev-tools) over here