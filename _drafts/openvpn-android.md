---
layout: post
title:  "OpenVPN library for Android"
date:   2016-10-01 00:22:33 +0800
categories: android openvpn vpn
tags: [ android, openvpn, vpn ]
---
<p>For months I've been working on adding OpenVPN features to one of my Android projects. The project is <a href="https://play.google.com/store/apps/details?id=com.surveybods.android">SurveyBods</a> and my client, ResearchBods needed the OpenVPN functionality to perform their "Usage Study" features.
To implement those features, I used Tunnel, an OpenVPN Android library. In this post I'll show you how to use Tunnel to build a simple app that imports an ovpn file and connects to a VPN service.</p>

<h3>What is OpenVPN?</h3>
<p>OpenVPN is an open-source software application that implements virtual private network (VPN) techniques for creating secure point-to-point or site-to-site connections in routed or bridged configurations and remote access facilities. It uses a custom security protocol that utilizes SSL/TLS for key exchange.</p>

<h3>How can I use OpenVPN on Android?</h3>
<p>There are many apps in the Google Play Store that can let you to connect with OpenVPN. I suggest you checkout the official OpenVPN app
<a href= "https://play.google.com/store/apps/details?id=net.openvpn.openvpn">OpenVPN Connect</a> and Arne Schwabe's
<a href="https://play.google.com/store/apps/details?id=de.blinkt.openvpn">OpenVPN for Android</a>.</p>

<h3>What is Tunnel?</h3>
<p>Tunnel is an open-source Android library project that I started. It is based on Arne Schwabe's
<a href="https://play.google.com/store/apps/details?id=de.blinkt.openvpn">OpenVPN for Android</a>.</p>
