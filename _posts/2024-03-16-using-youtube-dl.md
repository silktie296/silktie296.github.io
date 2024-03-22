---
layout: post
title: Downloading Youtube videos directly through youtube-dl
date: 2024-03-16 15:59 -0400
categories: [Tools]
tags: [Tools]
image:
    path: /assets/img/youtube-dl-batch.png
    alt: Huntress CTF Certificate, placed in the Top 2.6% with 100% of challenges solved.
---
<link href="../../assets/css/terminal_styles.css" rel="stylesheet">

How to download YouTube videos directly in the command-line and streamline workflow into Plex with a Batch file.

---

## Introduction and Purpose

I was looking for a way to directly download YouTube videos without using sketchy converter websites, and I came across a tool on GitHub that would allow me to do this directly through the Windows command-line,  [`youtube-dl`](https://github.com/ytdl-org/youtube-dl). 

This blog post will explain how to download this tool, go through its usage and documentation, and explore how we can set up a batch script to streamline this process to download these files directly to a Plex library and have it update to be watched immediately.

---

## How to Download

`yt-downloader` works on Linux, Mac, and Windows, with solid installation instructions and usage [documentation on GitHub](https://github.com/ytdl-org/youtube-dl#installation).

> To download the utility onto Windows, you can either download the .exe file to any location on your PATH or using the pip command.

I ran into issues with the latest version on the youtube-dl repository, but was able to get it working using the nightly version.

I used the pip installation method since I had it already installed, but I will provide explanations for both methods on a fresh Windows 11 machine.

<div class="language-console">
    <div class="highlight">
        <pre class="highlight"><code><strong>pip</strong> install <span class="t-seagreen">--upgrade --force-reinstall</span> <span style="color: #BB4444">"git+https://github.com/ytdl-org/ytdl-nightly.git"</span></code></pre>
    </div>
</div>

---

### Method 1: Downloading the .exe and adding it to PATH

Download the latest version of `youtube-dl.exe` from the [ytdl-nightly repository](https://github.com/ytdl-org/youtube-dl#installation).

I suggest creating a folder in your C:\Program Files\Youtube-DL and moving the executable file there.

Add the folder's directory to your PATH environmental variable:
- Copy the directory path for youtube-dl.exe (C:\Program Files\Youtube-DL)
- Open Settings and navigate to System > About > Advanced system settings > Environment Variables
- Select *PATH* > Edit > add youtube-dl's directory to the list.

Open a new command prompt, and you should be able to use the `youtube-dl` command. 

If you get a message saying the operation "cannot proceed because msvcr100.dll was not found", this can be fixed by installing "Microsoft Visual C++ 2010 Redistributable". This can be fixed by installing both the [x86](https://download.microsoft.com/download/1/6/5/165255E7-1014-4D0A-B094-B6A430A6BFFC/vcredist_x86.exe) and [x64](https://download.microsoft.com/download/1/6/5/165255E7-1014-4D0A-B094-B6A430A6BFFC/vcredist_x64.exe) versions from the [`Microsoft Docs`](https://learn.microsoft.com/en-US/cpp/windows/latest-supported-vc-redist?view=msvc-170).

---

### Method 2: Downloading with Python, pip, and git

pip is the standard package manager used by Python to easily import modules to use when writing python scripts. 

This also means that pip comes installed with Python, and we can get it by [downloading the latest version of Python](https://www.python.org/downloads/) (3.12.2 as of writing) from the official website. 

- When running the Python installer, check both:
  - "Use admin privileges when installing py.exe"
  - "Add python.exe to PATH"

After that you should be able to use the `py` command in the command prompt. <br/>It should open give an output like this:

<div class="language-console">
    <div class="code-header">
        <span data-label-text="Python"><i class="fas fa-code fa-fw small"></i></span>
        <span></span>
    </div>
    <div class="highlight">
        <pre class="highlight"><code><span class="t-blue">C:\Users\Austin></span> py<br/>Python <span style="color: #B8D7A3">3.12.2</span> (tags/v3.12.2:6abddd9, Feb 6 2024, 21:26:36) [MSC v.1937 64 bit (AMD64)] on win32<br/>Type <span style="color: #D69D85">&quot;help&quot;</span>, <span style="color: #D69D85">&quot;copyright&quot;</span>, <span style="color: #D69D85">&quot;credits&quot;</span> or <span style="color: #D69D85">&quot;license&quot;</span> for more information.<br/>>>></code></pre>
    </div>
</div>

If the Windows Store opens instead when using the command, you can tell Windows it installed doing the following:
- Open Settings: 'System > Settings > Apps > Apps & features'
- Select 'App execution aliases'
- Uncheck the entries for: 'App Installer - python.exe' and 'App Installer - python3.exe'

The pip command to download Python modules should now work using the command `py -m pip install <module>` 

Download [Git for Windows](https://git-scm.com/download/win) from its official website, and add Git to your path:
- Find the download directory for Git (C:\Program Files\Git\bin)
- Open Settings and navigate to System > About > Advanced system settings > Environment Variables
- Select *PATH* > Edit > add Git's directory to the list.

We can now download the nightly version of youtube-dl using the following:

<div class="language-console">
    <div class="highlight">
        <pre class="highlight"><code><span class="t-lightblue">py</span> <span class="t-seagreen">-m</span> <strong>pip</strong> install <span class="t-seagreen">--upgrade --force-reinstall </span><span style="color: #BB4444">"git+https://github.com/ytdl-org/ytdl-nightly.git"</span></code></pre>
    </div>
</div>

---

## Usage

After install, you should now be able to download videos from YouTube using the `youtube-dl <YT-URL>` command.

You can view the [documentation](https://github.com/ytdl-org/youtube-dl#description) on Github, or using the help command: `youtube-dl -h`

---

### Creating a Batch Script to utilize with Plex

If you are hosting a Plex Media Server, you can access media files anywhere you can download Plex (which includes on mobile and most Smart TVs). 

This is handy if you have a backlog of YouTube videos that you'd like to watch on TV without advertisements.

For this setup, I have created a Plex account, installed [`Plex Media Server`](https://www.plex.tv/media-server-downloads/), and organized my Media folders as shown below.

<div class="language-console">
    <div class="highlight">
        <pre class="highlight"><code>📂 <a href="https://support.plex.tv/articles/naming-and-organizing-your-tv-show-files/">Plex Media Server Folder Structure</a><br/>
/Media 
   /Movies
      movie content
   /TV Shows
      television content
   /YouTube
      youtube content</code></pre>
    </div>
</div>

I was able to come up with the following batch script:

<div class="language-console">
    <div class="code-header">
        <span data-label-text="Batch"><i class="fas fa-code fa-fw small"></i></span>
        <span></span>
    </div>
    <div class="highlight">
        <pre class="highlight"><code>@<span class="t-blue">echo off</span><br/><span class="t-green">:: This batch file downloads a YouTube video from a URL and saves to a chosen Plex library.</span><br/><span class="t-green">:: Then forces that Plex library to scan for updates.</span><br/><span class="t-blue">set</span> <span class="t-lightblue">PLEX_TOKEN</span>=(see: <a href="https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/">Finding an authentication token</a>)<br/><span class="t-blue">set</span> /p <span class="t-lightblue">URL</span>=Enter YouTube URL: <br/><br/><span class="t-blue">set</span> <span class="t-lightblue">ID</span>=<span class="t-seagreen">0</span><br/><span class="t-blue">echo</span> Enter the library that you would like this media to be saved to: <br/><br/>:<span class="t-blue">loop</span><br/><br/><span class="t-orange">"C:\Program Files\Plex\Plex Media Server\Plex Media Scanner.exe"</span> --list<br/><br/><span class="t-blue">set</span> /p <span class="t-lightblue">ID</span>=<span class="t-orange">"Library ID#: "</span><br/><br/>if <span class="t-lightblue">%ID%</span>==<span class="t-seagreen">1</span> <span class="t-gold">(</span>set DIR=Movies&&goto endloop<span class="t-gold">)</span><br/>if <span class="t-lightblue">%ID%</span>==<span class="t-seagreen">2</span>  <span class="t-gold">(</span>set DIR=TV Shows&&goto endloop<span class="t-gold">)</span><br/>if <span class="t-lightblue">%ID%</span>==<span class="t-seagreen">3</span>  <span class="t-gold">(</span>set DIR=YouTube&&goto endloop<span class="t-gold">)</span><br/><br/><span class="t-blue">echo</span>. && <span class="t-blue">echo</span> Invalid Choice<br/>goto loop<br/><br/><br/>:<span class="t-blue">endloop</span><br/><br/><span class="t-blue">set</span> DL_DIR=C:/Users/silkt/Videos/Plex/Media/<span class="t-lightblue">%DIR%</span>/<br/><span class="t-blue">echo</span>. && <span class="t-blue">echo</span> Media will be saved to <span class="t-lightblue">%DL_DIR%</span><br/><br/>youtube-dl <span class="t-lightblue">%URL%</span> --output <span class="t-orange">"</span><span class="t-lightblue">%DL_DIR%</span><span class="t-gold">%%</span><span class="t-orange">(title)s -</span> <span class="t-gold">%%</span><span class="t-orange">(uploader)s.%%(ext)s"</span><br/><br/><span class="t-blue">echo</span> Updating Plex Media Server <span class="t-gold">(</span><span class="t-lightblue">%DL_DIR%</span><span class="t-gold">)</span> folder<br/>curl "<u class="t-orange">http://localhost:32400/library/sections/<span class="t-lightblue">%ID%</span>/refresh?X-Plex-Token=<span class="t-lightblue">%PLEX_TOKEN%</span>"</u></code></pre>
    </div>
</div>

---

