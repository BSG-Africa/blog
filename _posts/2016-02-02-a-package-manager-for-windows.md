---
title: A package manager for Windows
author: richardyoung
---
One of my biggest gripes with developing software on Windows, is how hard it is to install software. In a world where scripting *everything* means saving hours of time, Windows simply cannot stack up to its Unix-based competitors. <!--more-->

The introduction of PowerShell brought some powerful scripting tools to developers, but it's utterly useless when most Windows software must first be installed by downloading binaries (from possibly dubious sources), clicking through a wizard, waiting for the install to complete, and finally clicking finish. Updating your software is just as difficult. Most people do not truly understand how much of an inconvenience it is until they need to install their entire tech stack on 20 different servers.

<center>
  <img title="Wizard" src="{{ site.baseurl }}/images/wizard.jpg"/>
</center>

But now there is [Chocolatey](https://chocolatey.org/). Chocolatey NuGet is a Machine Package Manager, somewhat like apt-get, but built with Windows in mind. You can install thousands of packages with one line, silently, and from verified sources.

~~~bash
    C:\> choco install jre8
~~~

Even installing Chocolatey itself is as easy as executing one line of PowerShell. The future of Windows package management looks bright now that Microsoft has caught onto the idea. [OneGet](https://github.com/oneget/oneget) (aka PackageManagement) ships with Windows 10 and WMF 5.0 and allows you to download packages from the Chocolatey repository, as well as manage the MSI-installed programs you already have. In the future this will likely include many more sources like the Windows app store, Python packages and Ruby gems.
