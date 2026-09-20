# [Odoo](https://www.odoo.com "Odoo's Homepage") 20 Install Script — x86 / x86_64 / ARM
# Arch detection + patched wkhtmltopdf + optional Paper Muncher

This script is based on the install script from André Schenkels (https://github.com/aschenkels-ictstudio/openerp-install-scripts)
but goes a bit further and has been improved. This script will also give you the ability to define an http_port in the .conf file that is generated under /etc/
This script can be safely used in a multi-odoo code base server because the default Odoo port is changed BEFORE the Odoo is started.

## Which script do I use?

| Script | Target |
|---|---|
| `odoo_install.sh` | Ubuntu 24.04 LTS and newer |
| `odoo_install_debian.sh` | Debian 13 (trixie) and newer |

Both scripts now share the same logic and options; only the defaults (distribution
codename, certbot install method) differ.

## Requirements for Odoo 20

Odoo 20 raised its minimum requirements. The script checks both and aborts with a clear
message instead of producing a broken install:

| | Odoo 19 | Odoo 20 |
|---|---|---|
| Python | 3.10+ | **3.12+** |
| PostgreSQL | 13+ | **16+** |
| `http_interface` default | `0.0.0.0` | **`127.0.0.1`** |

Because of the Python requirement, **Ubuntu 22.04 (Python 3.10) and Debian 12
(Python 3.11) can no longer run Odoo 20**. Use Ubuntu 24.04 LTS or Debian 13 or newer.

## What changed compared to the 19.0 branch

- **`http_interface` now defaults to `127.0.0.1` in Odoo 20** (it was `0.0.0.0` up to
  Odoo 19). Without setting it, a fresh Odoo 20 is only reachable from the machine
  itself. The script writes it explicitly: `0.0.0.0` on a plain install, `127.0.0.1`
  when Nginx is installed in front of Odoo. Override with `HTTP_INTERFACE`.
- `longpolling_port` was removed from Odoo; the config file now writes `gevent_port`.
- The Nginx template proxies `/websocket` to the gevent port with the `Upgrade` /
  `Connection` headers, which is what the Odoo bus actually uses (the old
  `/longpolling` location never worked).
- Dependencies are installed from the distribution packages by default, parsed out of
  Odoo's own `debian/control`, which is the method Odoo documents. Set
  `DEPENDENCIES_MODE="pip"` to use `requirements.txt` instead.
- `pytz` and `xlwt` were dropped from Odoo's requirements and `h11` was added (it is used
  by the new Paper Muncher PDF engine).
- The Enterprise extra libraries were updated to what the Odoo 20 Enterprise manifests
  actually declare: `paramiko`, `pyjwt`, `xmlsec`, `phonenumbers`. `dbfread`, `ebaysdk`
  and `firebase_admin` are no longer needed, and the `less` npm packages were dropped
  (Odoo has not shipped `.less` files for years).
- wkhtmltopdf is now installed from the official `0.12.6.1-3` build that is patched
  against Qt, so report headers and footers render correctly. The distribution package
  is only used as a fallback (and the script warns about it).
- A systemd unit is installed instead of the legacy `/etc/init.d` script. The init
  script is still used automatically when systemd is not available (for example inside
  a container).
- The addons path no longer drops `custom/addons` when Enterprise is installed.
- `odoo_install_debian.sh` was brought up to date; it was still pinned at Odoo 14.0 with
  wkhtmltopdf 0.12.5 Buster download links.

## Installing Nginx
If you set the parameter ```INSTALL_NGINX``` to ```True``` you should also configure workers. Without workers you will probably get connection loss issues. Look at [the deployment guide from Odoo](https://www.odoo.com/documentation/20.0/administration/on_premise/deploy.html) on how to configure workers.

## Installation procedure

##### 1. Download the script:
```
sudo wget https://raw.githubusercontent.com/Yenthe666/InstallScript/20.0/odoo_install.sh
```
##### 2. Modify the parameters as you wish.
There are a few things you can configure, this is the most used list:<br/>
```OE_USER``` will be the username for the system user.<br/>
```GENERATE_RANDOM_PASSWORD``` if this is set to ```True``` the script will generate a random password, if set to ```False```we'll set the password that is configured in ```OE_SUPERADMIN```. By default the value is ```True``` and the script will generate a random and secure password.<br/>
```INSTALL_WKHTMLTOPDF``` set to ```False``` if you do not want to install Wkhtmltopdf, if you want to install it you should set it to ```True```.<br/>
```OE_PORT``` is the port where Odoo should run on, for example 8069.<br/>
```HTTP_INTERFACE``` is the interface Odoo binds to. Leave empty to let the script pick (```0.0.0.0``` without Nginx, ```127.0.0.1``` with Nginx). Odoo 20 changed this default, see above.<br/>
```GEVENT_PORT``` is the port used by the websocket (live chat, bus, notifications), 8072 by default.<br/>
```OE_VERSION``` is the Odoo version to install, for example ```20.0``` for Odoo V20.<br/>
```IS_ENTERPRISE``` will install the Enterprise version on top of ```20.0``` if you set it to ```True```, set it to ```False``` if you want the community version of Odoo 20.<br/>
```INSTALL_POSTGRESQL_PGDG``` installs PostgreSQL from the official postgresql.org repository instead of the distribution package.<br/>
```POSTGRESQL_VERSION``` is the PostgreSQL major version to install, 17 by default. Odoo 20 needs 16 or above.<br/>
```DEPENDENCIES_MODE``` is ```apt``` (distribution packages, recommended) or ```pip``` (```requirements.txt```).<br/>
```INSTALL_PAPER_MUNCHER``` set to ```True``` to also install [Paper Muncher](https://odoo.github.io/paper-muncher/), the new in-house Odoo PDF engine. It is amd64 only for now, and opt-in: wkhtmltopdf stays the default engine.<br/>
```USE_SYSTEMD``` set to ```False``` to keep the legacy ```/etc/init.d``` service.<br/>
```OE_SUPERADMIN``` is the master password for this Odoo installation.<br/>
```INSTALL_NGINX``` is set to ```False``` by default. Set this to ```True``` if you want to install Nginx.<br/>
```WEBSITE_NAME``` Set the website name here for nginx configuration<br/>
```ENABLE_SSL``` Set this to ```True``` to install [certbot](https://github.com/certbot/certbot) and configure nginx with https using a free Let's Encrypted certificate<br/>
```ADMIN_EMAIL``` Email is needed to register for Let's Encrypt registration. Replace the default placeholder with an email of your organisation.<br/>
```INSTALL_NGINX``` and ```ENABLE_SSL``` must be set to ```True``` and the placeholder in ```ADMIN_EMAIL``` must be replaced with a valid email address for certbot installation<br/>
  _By enabling SSL though Let's Encrypt you agree to the following [policies](https://www.eff.org/code/privacy/policy)_ <br/>

#### 3. Make the script executable
```
sudo chmod +x odoo_install.sh
```
##### 4. Execute the script:
```
sudo ./odoo_install.sh
```

## Using Paper Muncher instead of wkhtmltopdf

Odoo 20 ships a pluggable PDF engine. `base_report_wkhtmltox` is auto-installed and
wkhtmltopdf stays the default. To switch to Paper Muncher after installing the binary
(`INSTALL_PAPER_MUNCHER="True"`):

1. Install the `base_report_paper_muncher` module in your database.
2. Set the `report.pdf_engine_default` system parameter to `paper-muncher`
   (Settings ‣ Technical ‣ System Parameters).

## Where should I host Odoo?
There are plenty of great services that offer good hosting. The script has been tested with a few major players such as [Google Cloud](https://cloud.google.com/), [Hetzner](https://www.hetzner.com/), [Amazon AWS](https://aws.amazon.com/) and [DigitalOcean](https://www.digitalocean.com/products/droplets/).
If you'd like you can use my [DigitalOcean referral link](https://m.do.co/c/d605cc420682) which gives you a 200$ voucher for free for the first 60 days.

## Minimal server requirements
While technically you can run an Odoo instance on 1GB (1024MB) of RAM it is absolutely not advised. A Linux instance typically uses 300MB-500MB and the rest has to be split among Odoo, postgreSQL and others. If you install an Odoo you should make sure to use at least 2GB of RAM. This script might fail with less resources too.
There are known issues on DigitalOcean for example where the installation crashes on 1GB RAM machines. See https://github.com/Yenthe666/InstallScript/issues/243
