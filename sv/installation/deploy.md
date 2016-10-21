---
lang: sv
layout: installation
meta_title: Hur man installerar Ghost på sin server - Ghost Docs
meta_description: Allt du behöver för att få igång bloggningsplattformen Ghost på din lokala- eller fjärrmiljö.  
heading: Installera Ghost &amp; Komma igång
subheading: De första stegen för att sätta upp din nya blogg för första gången.
permalink: /sv/installation/deploy/
chapter: installation
section: deploy
prev_section: linux
next_section: upgrading
---
## Få Ghost Live <a id="deploy"></a>

Så du är redo att komma igång med Ghost? Förträffligt!

Det första beslutet du behöver ta är om du vill installera och konfigurera Ghost själv, eller om du föredrar att använda en installerare.

### Installerare

Det finns några val när det kommer till enkla installerare för tillfället:

*   Distribuera till molnet med [Bitnami](http://wiki.bitnami.com/Applications/BitNami_Ghost).
*   Starta Ghost med [Rackspace deployments](http://developer.rackspace.com/blog/launch-ghost-with-rackspace-deployments.html).
*   Kom igång med en [DigitalOcean Droplet](https://www.digitalocean.com/community/articles/how-to-use-the-digitalocean-ghost-application).

### Manuel Setup



Du kommer behöva ett hostingpaket som redan har eller tillåter dig att installera [Node.js](http://nodejs.org).

    Det betyder att någonting som ett moln ([Amazon EC2](http://aws.amazon.com/ec2/), [DigitalOcean](http://www.digitalocean.com), [Rackspace Cloud](http://www.rackspace.com/cloud/)), VPS ([Webfaction](https://www.webfaction.com/), [Dreamhost](http://www.dreamhost.com/servers/vps/)) eller annat paket som har tillgång till SSH (terminal) & tillåter dig att installera Node.js. There are plenty around and they can be very cheap. Det finns massorm som dessutom kan vara ganska billiga.

Någonting som inte fungerar för tillfället är cPanel-liknande hosting lösningar, då dessa oftast är menade för att hosta PHP. Vissa erbjuder dock Ruby och kan komma att erbjuda Node.js i framtiden, då de mer eller mindre liknar varandra.


<p>Oturligt nog är många Node-specifika molnlösningar såsom **Nodejitsu** & **Heroku** **INTE** kompatibla Ghost. 
De kommer att fungera till en början, men kommer att radera alla filer vilket resulterar i att alla uppladdade bilder och databasen kommer försvinna.
Följande länker innehåller intruktioner om hur du kommer igång med:
*   [Dreamhost](http://www.howtoinstallghost.com/how-to-install-ghost-on-dreamhost/) - från [howtoinstallghost.com](http://howtoinstallghost.com)
*   [DigitalOcean](http://ghosted.co/install-ghost-digitalocean/) - från [Corbett Barr](http://ghosted.co)
*   [Webfaction](http://www.howtoinstallghost.com/how-to-install-ghost-on-webfaction-hosting/) - från [howtoinstallghost.com](http://howtoinstallghost.com)
*   [Rackspace](http://ghost.pellegrom.me/installing-ghost-on-ubuntu/) (Ubuntu 13.04 + linux service) - från [Gilbert Pellegrom](http://ghost.pellegrom.me/)
*   [Ubuntu + nginx + forever](http://0v.org/installing-ghost-on-ubuntu-nginx-and-mysql/) - från [Gregg Housh](http://0v.org/)
*   ...se [installationsforumet](https://en.ghost.org/forum/installation) för fler guider ...

## Få Ghost att köras för evigt

Den tidiagre beskrivna metoden för att start Ghost är `npm start`. Det är ett bra sätt att köra lokal utveckling och tester, men om du startar Ghost via kommandotolken kommer det att stoppas när du stänger terminalen eller loggar ut från SSH. För att förhindra Ghost från att stoppas måste du köra Ghost som en service. Det finns två sätt att göra det på.

### Forever ([https://npmjs.org/package/forever](https://npmjs.org/package/forever)) <a id="forever"></a>

Du kan använda `forever` för att köra Ghost som en bakgrundsprocess. `forever` kommer också ta hand om din Ghost-installation och starta om Node-processen om den krashar.

*   För att installera `forever` skriv `npm install forever -g`
*   För att starta Ghost med `forever` från samma directory som din Ghost installation skriv `NODE_ENV=production forever start index.js`
*   För att stoppa Ghost skriv `forever stop index.js`
*   För att se om Ghost körs för tillfället skriv `forever list`

### Supervisor ([http://supervisord.org/](http://supervisord.org/)) <a id="supervisor"></a>

Populära Linux distributioner&mdash;såsom Fedora, Debian, och Ubuntu&mdash;underhåller ett paket för Supervisor: Ett processkontrollsystem som tillåter dig att köra Ghost vid systemets start utan att använda init scripts. Olikt ett init script, är Supervisor portabelt mellan Linux distributioner och versioner.

*   [Installera Supervisor](http://supervisord.org/installing.html) som krävs för din Linux distribution. Oftast är detta:
    *   Debian/Ubuntu: `apt-get install supervisor`
    *   Fedora: `yum install supervisor`
    *   De flesta andra distributioner: `easy_install supervisor`
*   Försäkra dig om att Supervisor körs, genom att köra `service supervisor start`
*   Skapa ett startup skript för din Ghost installaion. Oftast kommer detta hamna i `/etc/supervisor/conf.d/ghost.conf` Till exempel:

    ```
    [program:ghost]
    command = node /path/to/ghost/index.js
    directory = /path/to/ghost
    user = ghost
    autostart = true
    autorestart = true
    stdout_logfile = /var/log/supervisor/ghost.log
    stderr_logfile = /var/log/supervisor/ghost_err.log
    environment = NODE_ENV="production"
    ```

*   Starta Ghost genom Supervisor: `supervisorctl start ghost`
*   För att stoppa Ghost: `supervisorctl stop ghost`

Du kan se [dokumentationen för Supervisor](http://supervisord.org) för mer information.

### Init Script <a id="init-script"></a>

Linux systems use init scripts to run on system boot. These scripts exist in /etc/init.d. To make Ghost run forever and even survive a reboot you could set up an init script to accomplish that task. The following example will work on Ubuntu and was tested on **Ubuntu 12.04**.

*   Create the file /etc/init.d/ghost with the following command:

    ```
    $ sudo curl https://raw.githubusercontent.com/TryGhost/Ghost-Config/master/init.d/ghost \
      -o /etc/init.d/ghost
    ```

*   Open the file with `nano /etc/init.d/ghost` and check the following:
*   Change the `GHOST_ROOT` variable to the path where you installed Ghost
*   Check if the `DAEMON` variable is the same as the output of `which node`
*   The Init script runs with it's own Ghost user and group on your system, let's create them with the following:

    ```
    $ sudo useradd -r ghost -U
    ```

*   Let's also make sure the Ghost user can access the installation:

    ```
    $ sudo chown -R ghost:ghost /path/to/ghost
    ```

*   Change the execution permission for the init script by typing

    ```
    $ sudo chmod 755 /etc/init.d/ghost
    ```

*   Now you can control Ghost with the following commands:

    ```
    $ sudo service ghost start
    $ sudo service ghost stop
    $ sudo service ghost restart
    $ sudo service ghost status
    ```

*   To start Ghost on system start the newly created init script has to be registered for start up.
    Type the following two commands in command line:

    ```
    $ sudo update-rc.d ghost defaults
    $ sudo update-rc.d ghost enable
    ```

*   Let's make sure your user can change files, config.js for example in the Ghost directory, by assigning you to the ghost group:
    ```
    $ sudo adduser USERNAME ghost
    ```

*   If you now restart your server Ghost should already be running for you.


## Setting up Ghost with a domain name <a id="nginx-domain"></a>

If you have setup up Ghost to run forever you can also setup a web server as a proxy to serve your blog with your domain.
In this example we assume you are using **Ubuntu 12.04** and use **nginx** as a web server.
It also assumes that Ghost is running in the background with one of the above mentioned ways.

*   Install nginx

    ```
    $ sudo apt-get install nginx
    ```
    <span class="note">This will install nginx and setup all necessary directories and basic configurations.</span>

*   Configure your site

    *   Create a new file in `/etc/nginx/sites-available/ghost.conf`
    *   Open the file with a text editor (e.g. `sudo nano /etc/nginx/sites-available/ghost.conf`)
        and paste the following

        ```
        server {
            listen 80;
            server_name example.com;

            location / {
                proxy_set_header   X-Real-IP $remote_addr;
                proxy_set_header   Host      $http_host;
                proxy_pass         http://127.0.0.1:2368;
            }
        }

        ```

    *   Change `server_name` to your domain
    *   Symlink your configuration in `sites-enabled`:

    ```
    $ sudo ln -s /etc/nginx/sites-available/ghost.conf /etc/nginx/sites-enabled/ghost.conf
    ```

    *   Restart nginx

    ```
    $ sudo service nginx restart
    ```

## Setting up Ghost with SSL <a id="ssl"></a>

After setting up a custom domain it is a good idea to secure the admin interface or maybe your whole blog using HTTPS. It is advisable to protect the admin interface with HTTPS because username and password are going to be transmitted in plaintext if you do not enable encryption.

The following example will show you how to set up SSL. We assume, that you have followed this guide so far and use nginx as your proxy server. A setup with another proxy server should look similar.

First you need to obtain a SSL certificate from a provider you trust. Your provider will guide you through the process of generating your private key and a certificate signing request (CSR). After you have received the certificate file you have to copy the CRT file from your certificate provider and the KEY file which is generated during issuing the CSR to the server.

- `mkdir /etc/nginx/ssl`
- `cp server.crt /etc/nginx/ssl/server.crt`
- `cp server.key /etc/nginx/ssl/server.key`

After these two files are in place you need to update your nginx configuration.

*   Open the nginx configuration file with a text editor (e.g. `sudo nano /etc/nginx/sites-available/ghost.conf`)
*   Add the settings indicated with a plus to your configuration file:

    ```
     server {
         listen 80;
    +    listen 443 ssl;
         server_name example.com;
    +    ssl_certificate        /etc/nginx/ssl/server.crt;
    +    ssl_certificate_key    /etc/nginx/ssl/server.key;
         ...
         location / {
    +       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    +       proxy_set_header Host $http_host;
    +       proxy_set_header X-Forwarded-Proto $scheme;
            proxy_pass http://127.0.0.1:2368;
            ...
         }
     }
    ```

    *   Restart nginx

    ```
    $ sudo service nginx restart
    ```

After these steps you should be able to reach the admin area of your blog using a secure HTTPS connection. If you want to force all your traffic to use SSL it is possible to change the protocol of the url setting in your config.js file to https (e.g.: `url: 'https://my-ghost-blog.com'`). This will force the use of SSL for frontend and admin. All requests sent over HTTP will be redirected to HTTPS. If you include images in your post that are retrieved from domains that are using HTTP an 'insecure content' warning will appear. Scripts and fonts from HTTP domains will stop working.

In most cases you'll want to force SSL for the administration interface and serve the frontend using HTTP and HTTPS. To force SSL for the admin area the option `forceAdminSSL: true` was introduced.

If you need further information on how to set up SSL for your proxy server the official SSL documention of [nginx](http://nginx.org/en/docs/http/configuring_https_servers.html) and [apache](http://httpd.apache.org/docs/current/ssl/ssl_howto.html) are a perfect place to start.
