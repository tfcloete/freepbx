# FreePBX on Docker

[<img src="https://img.shields.io/docker/pulls/tfcloete/freepbx"/>](https://hub.docker.com/r/tfcloete/freepbx)
[<img src="https://img.shields.io/docker/automated/tfcloete/freepbx"/>](https://hub.docker.com/r/tfcloete/freepbx)

FreePBX container image for running a complete Asterisk server - as forked from [flaviostutz/freepbx](https://github.com/flaviostutz/freepbx).
This development, including the content of this readme, was done by Flavio Stutz. I added the Asterisk queue module to the dokerfile - and made a few configuration changes required for our deployment to the docker-hub.yml

With this container you can create a telephony system in your office or house with integration among various office branches and integration to external VOIP providers with features such as call recording and IVR (interactive voice response) Menus.

If "Apply" is taking too long, disable "Module signature check" (if you know what you're doing).

Thanks to https://github.com/tiredofit/docker-freepbx for various insights on the new Asterisk 15 installation.

This image is used in production deployments.

## Image includes

* Asterisk 16
* FreePBX 15
* Modules: IVR, Time Conditions, Backup, Recording
* Automatic backup script


## Usage

* Create docker-compose.yml

```yml
version: '3.3'
services:
  freepbx:
    image: tfcloete/freepbx
    container_name: asterisk
    hostname: borderpbx
    extra_hosts:
      - "borderpbx:127.0.0.1"
    network_mode: "host"
    ports:
      - 8080:80
      - 5060:5060/udp
      - 5160:5160/udp
      - 3306:3306
      - 18000-18100:18000-18100/udp
    restart: always
    environment:
      - ADMIN_PASSWORD=admin123
    volumes:
      - backup:/backup
      - recordings:/var/spool/asterisk/monitor

volumes:
  backup:
  recordings:
```

* Create the docker container - but DON'T start it yet! ```docker-compose up --no-start```

* Restore the Asterisk configuration from backup. ```docker cp new.tar.gz <CONTAINER_ID>:backup/```

* Run ```docker-compose up -d```

* Open admin panel at http://localhost:8080/admin/

## Sample host preparation

* Install Ubuntu 18.04

* Install Docker + Docker Compose

* Run this container

## ENVs

* **ADMIN_PASSWORD** - GUI password for user 'admin'. required
* **RTP_START** - port range from for RTP. defaults to 18000
* **RTP_FINISH** - port range to for RTP. defaults to 18100
* **SIP_NAT_IP** - SIP NAT Public IP for calls. defaults to ip got from "curl ifconfig.me"
* **USE_CHAN_SIP** - if true, disables pjsip and enables legacy chan_sip engine. defaults to false, meaning it will use pjsip engine by default
* **ENABLE_AUTO_RESTORE** - if true, when a new container instance is run, it will try to restore an existing backup from /backup/new.tar.gz. This backup is created each one hour automatically. This is useful when creating a new container instance (all MYSQL and other data is lost), so that your configurations are kept. defaults to true
* **ENABLE_DELETE_OLD_RECORDINGS** - Delete all recordings older than 60 days if enabled. defaults to true
* **DISABLE_SIGNATURE_CHECK** - Disables module signature checks so that configuration reloads are way faster. Disable if you know what module signing protection means. defaults to false
* **CERTIFICATE_DOMAIN** - certificate domain name when generating site certs with let's encrypt. this is used to locate certificated by name in /etc/asterisk/keys/ and configure Apache to use it automatically. defaults to ''

## Volumes

* **/backup** - keeps new.tar.gz and old.tar.gz automatic backups. Default backup job stores backup there too.
* **/var/spool/asterisk/monitor** - call recording storage location

* **/etc/asterisk/keys** - Let's Encrypt and self signed certificates pub/private keys generated in pbxadmin

