# duckdns-systemd
[Duck DNS](https://www.duckdns.org/) updater script with a dedicated systemd unit and logging configuration

### Why this script over the default cron entry?
- This script only updates the IP if it has actually changed, instead of connecting to www.duckdns.org every X minutes.
- This script logs to a file, which is useful for diagnostics.
- The [suggested command](https://www.duckdns.org/install.jsp#linux-cron) uses `curl -k`, which does not verify the server certificate, making the connection vulnerable to MiTM, see [here](https://curl.haxx.se/docs/manpage.html#-k). Probably they did this because some embedded devices like routers may not have a trust store to check the certificate against.

### Installation

Install `dig` to query the DNS for the current IP addres, and `curl` to use the API
```
sudo apt install dnsutils curl
```

Clone the repository

```
git clone https://github.com/orazioedoardo/duckdns-systemd.git
```

Copy the script to some location, for example ~/
```
cp duckdns-systemd/duckdns ~/
```

Setup the systemd unit
```
mkdir -p ~/.config/systemd/user
cp duckdns-systemd/conf/duckdns.service ~/.config/systemd/user/
nano ~/.config/systemd/user/duckdns.service
```

Change the line `ExecStart=...` to point to the FULL PATH to the script, for example `/home/user/duckdns` in this case

Setup logging and log rotation
```
sudo cp duckdns-systemd/conf/duckdns /etc/logrotate.d/
sudo chown root:root /etc/logrotate.d/duckdns
sudo cp duckdns-systemd/conf/20-duckdns.conf /etc/rsyslog.d/
sudo chown root:root /etc/rsyslog.d/20-duckdns.conf
sudo systemctl restart rsyslog
```

Now create the configuration file for the script
```
mkdir ~/.duckdns
nano ~/.duckdns/config
```
Enter your token, the domain (without .duckdns.org) and the number of seconds between checks, example:
```
TOKEN=310eff65-bd83-4c52-b655-9d1f8d1036dc
DOMAIN=mydomain
INTERVAL=300
```

Start and enable the service
```
sudo loginctl enable-linger $(whoami)
systemctl --user daemon-reload
systemctl --user start duckdns
systemctl --user enable duckdns
```

Check that the service is indeed working
```
$ systemctl --user status duckdns
● duckdns.service - Duck DNS Updater
   Loaded: loaded (/home/user/.config/systemd/user/duckdns.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2019-09-03 16:55:16 CEST; 8s ago
 Main PID: 711 (duckdns)
   CGroup: /user.slice/user-1000.slice/user@1000.service/duckdns.service
           ├─711 /bin/bash /home/user/duckdns
           └─722 sleep 300

set 03 16:55:16 debian systemd[499]: Started Duck DNS Updater.
set 03 16:55:16 debian duckdns[716]: Initialization succeeded, domain: mydomain.duckdns.org, IP: 0.0.0.0
```

You can also check out the log at `/var/log/duckdns.log`
