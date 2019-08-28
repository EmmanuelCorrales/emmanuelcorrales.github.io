---
layout: post
title: "How I install and configure Docker for my Mac via Homebrew"
date: 2019-01-29 10:30:00 +0800
categories: docker
tags: [ docker ]
---

This is the best way I can come up with to install Docker on my Mac. I never
liked manually downloading the "*.dmg*" file and then moving it to the
*Applications* directory. I will be sharing in this post how I installed it via
Homebrew while also configuring bash completion for Docker.

### Steps to install and configure Docker:
1. [Install Homebrew and Cask](#install_homebrew_and_cask)
2. [Install Docker](#install_docker)
3. [Setup Bash auto completion](#setup_bash_auto_completion)

## <a name="install_homebrew_and_cask" />Install Homebrew and Cask
Homebrew is a package manager for Mac and has always been my preferred way to
install my tools because I can integrate it with my setup scripts. To install it
execute
```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
then install *Homebrew Cask* which is an extension of *Homebrew*. It makes the
installation of large binaries and graphical applications simpler.
```bash
brew tap caskroom/versions
```

## <a name="install_docker" />Install Docker
To install Docker via Homebrew run
```bash
brew cask install docker
```
then after installation run
```bash
docker --version
```
to confirm its installation.

## <a name="setup_bash_auto_completion" />Setup Bash auto completion

Install bash completion via Homebrew.
```bash
brew install bash-completion
```
Create symlinks for Docker's completion on Bash.
```bash
etc=/Applications/Docker.app/Contents/Resources/etc
ln -s $etc/docker.bash-completion $(brew --prefix)/etc/bash_completion.d/docker
ln -s $etc/docker-machine.bash-completion $(brew --prefix)/etc/bash_completion.d/docker-machine
ln -s $etc/docker-compose.bash-completion $(brew --prefix)/etc/bash_completion.d/docker-compose
```
Add this line to your **~/.bash_profile**.
```bash
[ -f /usr/local/etc/bash_completion ] && . /usr/local/etc/bash_completion
```
Apply the changes by running
```bash
source ~/.bash_profile
```
Thats it! Now you have installed Docker while also enabling completion for Bash.
If you know a better way to install it please don't hesitate to comment below.
