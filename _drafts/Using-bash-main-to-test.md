---
#layout: post
title: Bash to the rescue - moving misplaces test classes with 1 line
date: 2018-07-31
categories: ant
author: EldarEccor
---

Scripting is not only you best friend and enabler when in comes to automation, but can make your daily life that much easier as well. As a Windows user myself (at least for workstations) I have been repeatedly surprised what can achieved with some simple lines of shell script. 

These scripts are usually so basic in nature, that it doesn't even matter which shell interprets them. I like to try and keep it that way as we have to support Solaris, SLES, CentOS, RedHat etc... and i would like to keep Pandora's box closed on that front at least.

The problem I faced today started out pretty easy. Some colleagues had checked in all their tests under *src/main/java* for years. This against java conventions that dictate *src/test/java* for all test classes and resources, but never seemed to be a problem. The CI build was always green and everybody was happy :)   

https://stackoverflow.com/questions/27472540/difference-between-and-in-bash

for FILE in `find . -name *TestCase.java`;do mkdir -p $(dirname "${FILE/main/test}"); mv $FILE "${FILE/main/test}"; done