---
layout: post
title: Gitbook Cli Compiles Chinese Into Blanks Issue
---
We have been using gitlab+gitbook+apache to manage some internal documents for several months. It works perfect. But yesterday, one of our team member asked me for a pdf version of an api document which was currently generated as static html via gitbook-cli. According to this [link](https://github.com/GitbookIO/gitbook-pdf), it's quite easy to achieve that. Acctually I was able to build the pdf in one minute. But when I opened the pdf generate, all the Chinese characters have gone, they were all replaced with blanks.

Apparently it's an encoding issue. I checked this issue on the Internet and got confirmed with a lot of similar issues. As I was using a centos 7.3 server as my compiling system, it does not contain essencial fonts(especially Chinese fonts) which is a need to compile a gitbook pdf with Chinese characters. I tried serveral fonts files, but it didn't work. So I just copied all the fonts on my macbook ```/Library/Fonts/*``` into ```/usr/share/fonts/``` on centos and issue fixed ;)