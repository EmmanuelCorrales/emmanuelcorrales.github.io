---
layout: post
title: "Installing Oracle JDK for MacOS via Homebrew"
date: 2019-02-05 08:30:00 +0800
categories: jdk java
tags: [ jdk java ]
---

MacOS has OpenJDK installed by default however I prefer to use Oracle's version
of JDK because its the official version. I don't want to install it the same way
Oracle instructs it on their docs as I find it very tedious. I'm a guy who loves
automating stuff so I prefer to install it via Homebrew. I frequently do a clean
install on my Mac every time there is a new version of OSX so I have to install
JDK again and again. I'd rather just run a single installation script instead of
heading over to Oracle's website and following their instructions.

### Steps to install and configure the Oracle JDK:
1. [Install Homebrew and Cask](#install_homebrew_and_cask)
2. [Install Oracle JDK](#install_jdk)
3. [Setup the JAVA_HOME environment variable](#setup_the_java_home_environment_variable)
4. [Verifying installation](#verifying_installation)

## <a name="install_homebrew_and_cask" />Homebrew and Cask
*Homebrew* is a package manager for Mac and has always been my preferred way to
install my command line tools because I can integrate it with my setup scripts.
To install it I'll run
```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
then I'll install *Homebrew Cask* which is an extension of *Homebrew*. It makes
the installation of large binaries and graphical applications simpler.
```bash
brew tap caskroom/versions
```
## <a name="install_jdk" />JDK Installation
Before I install the JDK, I'll check first which version it will install by
default. I'm very picky about the version because most of the time I just use
Java for Android development. I also prefer the older and more stable version of
JDK so I run
```bash
brew cask info java
```
which will output
```bash
java: 11.0.2,9
https://jdk.java.net/
Not installed
From: https://github.com/Homebrew/homebrew-cask/blob/master/Casks/java.rb
==> Name
OpenJDK
==> Artifacts
```
This means that the latest version is JDK 11. I can install it now by running
```bash
brew cask install java
```
but I prefer to install JDK 8 over 11 so instead I'll run

```bash
brew cask install java8
```
## <a name="setup_the_java_home_environment_variable" />Setup Java_HOME environment variable
Once installed, I will set the **JAVA_HOME** environment variable by editing my
*.bash_profile*
```bash
vim ~/.bash_profile
```
and inserting this line
```bash
export JAVA_HOME=$(/usr/libexec/java_home)
```
and applying these changes by running
```bash
source ~/.bash_profile
```

## <a name="verifying_installation" />Verifying Installation

Now to confirm if the installation was sucessful I'll run this command.
```bash
java -version
```
If the installation is successful, its output would be similiar to this
```bash
java version "1.8.0_202"
Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
```
This tells me that I have installed the Oracle version of the JDK. However if
the output is like this

```bash
java version "1.8.0_202"
OpenJDK Runtime Environment (build 1.8.0_202-b08)
OpenJDK 64-Bit Server VM (build 25.202-b08, mixed mode)
```
then I may have failed to install the JDK properly or the changes may not have
been applied yet because I can see that OpenJDK is still being used. I'll try to
fix this by restarting my Mac then running "*java -version*" again.

## Automating installation with a script
Below is a simple script to automate the installation of the latest Oracle JDK.
{% gist 86bf0f9a189114ec8cf1219288e7d280 jdk-oracle-macos-setup.bash %}

That's it! Now you can automate your JDK installation on you Mac by running the
script.
