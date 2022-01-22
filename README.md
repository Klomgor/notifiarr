<img src="https://docs.notifiarr.com/img/repo-logo.png">

This is the unified client for [Notifiarr.com](https://notifiarr.com).
The client enables content requests from Media Bot in your Discord Server.
It also provides reports for Plex usage and system health.

## Installation

### Linux

Linux repository hosting provided by
[![packagecloud](https://docs.golift.io/integrations/packagecloud-full.png "PackageCloud.io")](http://packagecloud.io)

This works on any system with apt or yum. If your system does not use APT or YUM, then download a package from the [Releases](https://github.com/Notifiarr/notifiarr/releases) page.
Install the Go Lift package repo and Notifiarr with this command:
```
curl -s https://golift.io/repo.sh | sudo bash -s - notifiarr
```

After install, edit the config and start the service:

```
sudo nano /etc/notifiarr/notifiarr.conf
sudo systemctl restart notifiarr
```

#### Arch Linux

This one is special; hope you know what you're doing.
Build a package with `makepkg` using the `aur` source.
Here: https://github.com/golift/aur

### FreeBSD

-   Download a package from the [Releases](https://github.com/Notifiarr/notifiarr/releases) page.
-   Install it, edit config, start it.

Example of the above in shell form:

```shell
wget -qO- https://raw.githubusercontent.com/Notifiarr/notifiarr/main/scripts/install.sh | sudo bash

vi /usr/local/etc/notifiarr/notifiarr.conf
service notifiarr start
```

On Linux and FreeBSD, Notifiarr runs as `user:group` `notifiarr:notifiarr`.

### macOS Install

#### Homebrew

-   Edit config file at `/usr/local/etc/notifiarr/notifiarr.conf`
-   Start it.
-   Like this:

```shell
brew install golift/mugs/notifiarr
vi /usr/local/etc/notifiarr/notifiarr.conf
brew services start notifiarr
```

#### macOS App

-   You can use the unsigned app on the Releases page.
-   You must right click the app and select `Open` so macOS allows it.
-   When you open it for the first time it will create a config file and log file:
    -   `~/.notifiarr/notifiarr.conf`
    -   `~/.notifiarr/notifiarr.log`
-   Edit the config file and reload or restart the app.

### Windows

-   Extract a `.exe.zip` file from [the Releases page](https://github.com/Notifiarr/notifiarr/releases).
-   Run the `notifiarr.amd64.exe` binary. This starts the app in the system tray.
-   When you open it for the first time it will create a config file and log file:
    -   `C:\ProgramData\notifiarr\notifiarr.conf`
    -   `<your home folder>\.notifiarr\notifiarr.log`
-   Edit the new config file suit your environment then reload or restart the app.

### Docker

This project builds automatically in [Docker Cloud](https://hub.docker.com/r/golift/notifiarr)
and creates [ready-to-use multi-architecture images](https://hub.docker.com/r/golift/notifiarr/tags).
The `latest` tag is always a tagged release in GitHub. The `main` tag corresponds
to the `main` branch in GitHub and may be broken.

#### Docker Config File

-   Copy the [example config file](https://github.com/Notifiarr/notifiarr/blob/main/examples/notifiarr.conf.example) from this repo.
-   Then grab the image from docker hub and run it using an overlay for the config file.
-   You must set `privileged` to use `smartctl` (`monitor_drives`) and/or `MegaCli` (`monitor_raid`).
-   Map the `/var/run/utmp` volume if you want to count users.
-   Mount any volumes you want to report storage space for. Where does not matter, "where" is the "name".
-   You MUST set a static hostname. Each client is identified by hostname.

```shell
docker pull golift/notifiarr
docker run --hostname=$(hostname) -d \
-v /your/config/notifiarr.conf:/config/notifiarr.conf \
-v /var/run/utmp:/var/run/utmp \
golift/notifiarr
docker logs <container id from docker run>
```

#### Docker Environment Variables

See below for more information about which environment variables are available.
You must set `--privileged` when `monitor drives` is enabled on the website.

```shell
docker pull golift/notifiarr
docker run --hostname $(hostname) -d --privileged \
  -v /var/run/utmp:/var/run/utmp \
  -e "DN_API_KEY=abcdef-12345-bcfead-43312-bbbaaa-123" \
  -e "DN_SONARR_0_URL=http://localhost:8989" \
  -e "DN_SONARR_0_API_KEY=kjsdkasjdaksdj" \
  golift/notifiarr
docker logs <container id from docker run>
```

## Configuration Information

-   Instead of, or in addition to a config file, you may configure a docker
    container with environment variables.
-   Any variable not provided takes the default.
-   Must provide an API key from notifiarr.com.
    -   **The Notifiarr application uses the API key for bi-directional authorization.**
-   Must provide URL and API key for Sonarr or Radarr or Readarr or any combination.
-   You may provide multiple sonarr, radarr or readarr instances using
    `DN_SONARR_1_URL`, `DN_SONARR_2_URL`, etc.

|Config Name|Variable Name|Default / Note|
|---|---|---|
|api_key|`DN_API_KEY`|**Required** / API Key from Notifiarr.com|
|auto_update|`DN_AUTO_UPDATE`|`off` / Set to `daily` to turn on automatic updates (windows only)|
|bind_addr|`DN_BIND_ADDR`|`0.0.0.0:5454` / The IP and port to listen on|
|quiet|`DN_QUIET`|`false` / Turns off output. Set a log_file if this is true|
|urlbase|`DN_URLBASE`|default: `/` Change the web root with this setting|
|upstreams|`DN_UPSTREAMS_0`|List of upstream networks that can set X-Forwarded-For|
|ssl_key_file|`DN_SSL_KEY_FILE`|Providing SSL files turns on the SSL listener|
|ssl_cert_file|`DN_SSL_CERT_FILE`|Providing SSL files turns on the SSL listener|
|log_file|`DN_LOG_FILE`|None by default. Optionally provide a file path to save app logs|
|http_log|`DN_HTTP_LOG`|None by default. Provide a file path to save HTTP request logs|
|log_file_mb|`DN_LOG_FILE_MB`|`100` / Max size of log files in megabytes|
|log_files|`DN_LOG_FILES`|`10` / Log files to keep after rotating. `0` disables rotation|
|file_mode|`DN_FILE_MODE`|`"0600"` / Unix octal filemode for new log files|
|timeout|`DN_TIMEOUT`|`60s` / Global API Timeouts (all apps default)|

All applications below (starr, downloaders, tautulli, plex) have a `timeout` setting.
If the configuration for an application is missing the timeout, the global timeout (above) is used.

### Secret Settings

Recommend not messing with these unless instructed to do so.

|Config Name|Variable Name|Default / Note|
|---|---|---|
|extra_keys|`DN_EXTRA_KEYS_0`|`[]` (empty list) / Add keys to allow API requests from places besides notifiarr.com|
|mode|`DN_MODE`|`production` / Change application mode: `development` or `production`|
|debug|`DN_DEBUG`|`false` / Adds payloads and other stuff to the log output; very verbose/noisy|
|debug_log|`DN_DEBUG_LOG`|`""` / Set a file system path to write debug logs to a dedicated file|
|max_body|`DN_MAX_BODY`|Unlimited, `0` / Maximum debug-log body size (integer) for payloads to and from notifiarr.com|
||`TMPDIR`|`%TMP%` on Windows. Varies depending on system; must be writable if using Backup Corruption Check|

_Note: You may disable the GUI (menu item) on Windows by setting the env variable `USEGUI` to `false`._

### System Snapshot

This application can take a snapshot of your system at an interval and send
you a notification. Snapshot means system health like cpu, memory, disk, raid, users, etc.
Other data available in the snapshot: mysql health, `iotop`, `iostat` and `top` data.
Some of this may only be available on Linux, but other platforms have similar abilities.

If you monitor drive health you must have smartmontools (`smartctl`) installed.
If you use smartctl on Linux, you must enable sudo. Add the sudoers entry below to
`/etc/sudoers` and fix the path to `smartctl` if yours differs. If you monitor
raid and use MegaCli (LSI card), add the appropriate sudoers entry for that too.

To monitor application disk I/O you may install `iotop` and add the sudoers entry
for it, shown below. This feature is enabled on the website.

#### Snapshot Sudoers

The following sudoers entries are used by various snapshot features. Add them if you use the respective feature.
You can usually just put the following content into `/etc/sudoers` or `/etc/sudoers.d/00-notifiarr`.

```
# Allows drive health monitoring on macOS, Linux/Docker and FreeBSD.
notifiarr ALL=(root) NOPASSWD:/usr/sbin/smartctl *

# Allows disk utilization monitoring on Linux (non-Docker).
notifiarr ALL=(root) NOPASSWD:/usr/sbin/iotop *

# Allows monitoring megaraid volumes on macOS, Linux/Docker and FreeBSD.
# Rarely needed, and you'll know if you need this.
notifiarr ALL=(root) NOPASSWD:/usr/sbin/MegaCli64 -LDInfo -Lall -aALL
```

#### Snapshot Packages

  - **Windows**:  `smartmontools` - get it here https://sourceforge.net/projects/smartmontools/
  - **Linux**:    Debian/Ubuntu: `apt install smartmontools`, RedHat/CentOS: `yum install smartmontools`
  - **Docker**:    It's already in the container. Lucky you! Just run it in `--privileged` mode.
  - **Synology**: `opkg install smartmontools`, but first get Entware:
    - Entware (synology):  https://github.com/Entware/Entware-ng/wiki/Install-on-Synology-NAS
    - Entware Package List:  https://github.com/Entware/Entware-ng/wiki/Install-on-Synology-NAS

#### Snapshot Configuration

There is no client configuration for snapshots (except MySQL, below).
Snapshot configuration is found on the [website](https://notifiarr.com).

#### MySQL Snapshots

You may add mysql credentials to your notifiarr configuration to snapshot mysql
service health. This feature snapshots `SHOW PROCESSLIST` and `SHOW STATUS` data.

Access to a database is not required. Example Grant:

```
GRANT PROCESS ON *.* to 'notifiarr'@'localhost'
```

|Config Name|Variable Name|Note|
|---|---|---|
snapshot.mysql.name|`DN_SNAPSHOT_MYSQL_NAME`|Setting a name enables service checks of MySQL|
snapshot.mysql.host|`DN_SNAPSHOT_MYSQL_HOST`|Something like: `localhost:3306`|
snapshot.mysql.user|`DN_SNAPSHOT_MYSQL_USER`|Username in the GRANT statement|
snapshot.mysql.pass|`DN_SNAPSHOT_MYSQL_PASS`|Password for the user in the GRANT statement|

### Lidarr

|Config Name|Variable Name|Note|
|---|---|---|
lidarr.name|`DN_LIDARR_0_NAME`|No Default. Setting a name enables service checks|
lidarr.url|`DN_LIDARR_0_URL`|No Default. Something like: `http://lidarr:8686`|
lidarr.api_key|`DN_LIDARR_0_API_KEY`|No Default. Provide URL and API key if you use Readarr|
lidarr.username|`DN_LIDARR_0_USERNAME`|Provide username if using backup corruption check and auth is enabled|
lidarr.password|`DN_LIDARR_0_PASSWORD`|Provide password if using backup corruption check and auth is enabled|
lidarr.http_user|`DN_LIDARR_0_HTTP_USER`|Provide username if Lidarr uses basic auth (uncommon) and BCC enabled|
lidarr.http_pass|`DN_LIDARR_0_HTTP_PASS`|Provide password if Lidarr uses basic auth (uncommon) and BCC enabled|
lidarr.max_body|`DN_LIDARR_0_MAX_BODY`|`0` (off) / How much of the response body is logged when debug is on|

* __BCC = Backup Corruption Check__

### Prowlarr

|Config Name|Variable Name|Note|
|---|---|---|
prowlarr.name|`DN_PROWLARR_0_NAME`|No Default. Setting a name enables service checks|
prowlarr.url|`DN_PROWLARR_0_URL`|No Default. Something like: `http://prowlarr:9696`|
prowlarr.api_key|`DN_PROWLARR_0_API_KEY`|No Default. Provide URL and API key if you use Readarr|
prowlarr.username|`DN_PROWLARR_0_USERNAME`|Provide username if using backup corruption check and auth is enabled|
prowlarr.password|`DN_PROWLARR_0_PASSWORD`|Provide password if using backup corruption check and auth is enabled|
prowlarr.http_user|`DN_PROWLARR_0_HTTP_USER`|Provide username if Prowlarr uses basic auth (uncommon) and BCC enabled|
prowlarr.http_pass|`DN_PROWLARR_0_HTTP_PASS`|Provide password if Prowlarr uses basic auth (uncommon) and BCC enabled|
prowlarr.max_body|`DN_PROWLARR_0_MAX_BODY`|`0` (off) / How much of the response body is logged when debug is on|

### Radarr

|Config Name|Variable Name|Note|
|---|---|---|
radarr.name|`DN_RADARR_0_NAME`|No Default. Setting a name enables service checks.|
radarr.url|`DN_RADARR_0_URL`|No Default. Something like: `http://localhost:7878`|
radarr.api_key|`DN_RADARR_0_API_KEY`|No Default. Provide URL and API key if you use Radarr|
radarr.username|`DN_RADARR_0_USERNAME`|Provide username if using backup corruption check and auth is enabled|
radarr.password|`DN_RADARR_0_PASSWORD`|Provide password if using backup corruption check and auth is enabled|
radarr.http_user|`DN_RADARR_0_HTTP_USER`|Provide username if Radarr uses basic auth (uncommon) and BCC enabled|
radarr.http_pass|`DN_RADARR_0_HTTP_PASS`|Provide password if Radarr uses basic auth (uncommon) and BCC enabled|
radarr.max_body|`DN_RADARR_0_MAX_BODY`|`0` (off) / How much of the response body is logged when debug is on.|

### Readarr

|Config Name|Variable Name|Note|
|---|---|---|
readarr.name|`DN_READARR_0_NAME`|No Default. Setting a name enables service checks|
readarr.url|`DN_READARR_0_URL`|No Default. Something like: `http://localhost:8787`|
readarr.api_key|`DN_READARR_0_API_KEY`|No Default. Provide URL and API key if you use Readarr|
readarr.username|`DN_READARR_0_USERNAME`|Provide username if using backup corruption check and auth is enabled|
readarr.password|`DN_READARR_0_PASSWORD`|Provide password if using backup corruption check and auth is enabled|
readarr.http_user|`DN_READARR_0_HTTP_USER`|Provide username if Readarr uses basic auth (uncommon) and BCC enabled|
readarr.http_pass|`DN_READARR_0_HTTP_PASS`|Provide password if Readarr uses basic auth (uncommon) and BCC enabled|
readarr.max_body|`DN_READARR_0_MAX_BODY`|`0` (off) / How much of the response body is logged when debug is on.|

### Sonarr

|Config Name|Variable Name|Note|
|---|---|---|
sonarr.name|`DN_SONARR_0_NAME`|No Default. Setting a name enables service checks|
sonarr.url|`DN_SONARR_0_URL`|No Default. Something like: `http://localhost:8989`|
sonarr.api_key|`DN_SONARR_0_API_KEY`|No Default. Provide URL and API key if you use Sonarr|
sonarr.username|`DN_SONARR_0_USERNAME`|Provide username if using backup corruption check and auth is enabled|
sonarr.password|`DN_SONARR_0_PASSWORD`|Provide password if using backup corruption check and auth is enabled|
sonarr.http_user|`DN_SONARR_0_HTTP_USER`|Provide username if Sonarr uses basic auth (uncommon) and BCC enabled|
sonarr.http_pass|`DN_SONARR_0_HTTP_PASS`|Provide password if Sonarr uses basic auth (uncommon) and BCC enabled|
sonarr.max_body|`DN_SONARR_0_MAX_BODY`|`0` (off) / How much of the response body is logged when debug is on.|

### Downloaders

You can add supported downloaders so they show up on the dashboard integration.
You may easily add service checks to these downloaders by adding a name.
Any number of downloaders of any type may be configured.

#### QbitTorrent

|Config Name|Variable Name|Note|
|---|---|---|
qbit.name|`DN_QBIT_0_NAME`|No Default. Setting a name enables service checks|
qbit.url|`DN_QBIT_0_URL`|No Default. Something like: `http://localhost:8080`|
qbit.user|`DN_QBIT_0_USER`|No Default. Provide URL, user and pass if you use Qbit|
qbit.pass|`DN_QBIT_0_PASS`|No Default. Provide URL, user and pass if you use Qbit|
qbit.http_user|`DN_QBIT_0_HTTP_USER`|Provide this username if Qbit is behind basic auth (uncommon)|
qbit.http_pass|`DN_QBIT_0_HTTP_PASS`|Provide this password if Qbit is behind basic auth (uncommon)|

#### SABnzbd

|Config Name|Variable Name|Note|
|---|---|---|
sabnzbd.name|`DN_SABNZBD_0_NAME`|No Default. Setting a name enables service checks|
sabnzbd.url|`DN_SABNZBD_0_URL`|No Default. Something like: `http://localhost:8080/sabnzbd`|
sabnzbd.api_key|`DN_SABNZBD_0_API_KEY`|No Default. Provide URL and API key if you use SABnzbd|

#### Deluge

|Config Name|Variable Name|Note|
|---|---|---|
deluge.name|`DN_DELUGE_0_NAME`|No Default. Setting a name enables service checks|
deluge.url|`DN_DELUGE_0_URL`|No Default. Something like: `http://localhost:8080`|
deluge.password|`DN_DELUGE_0_PASSWORD`|No Default. Provide URL and password key if you use Deluge|
deluge.http_user|`DN_DELUGE_0_HTTP_USER`|Provide this username if Deluge is behind basic auth (uncommon)|
deluge.http_pass|`DN_DELUGE_0_HTTP_PASS`|Provide this password if Deluge is behind basic auth (uncommon)|

### Plex

This application can also send Plex sessions to Notfiarr so you can receive
notifications when users interact with your server. This has three different features:

- Notify all sessions on a longer interval (30+ minutes).
- Notify on session nearing completion (percent complete).
- Notify on session change (Plex Webhook) ie. pause/resume.

You [must provide Plex Token](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/)
for this to work. You may also need to add a webhook to Plex so it sends notices to this application.

- In Plex Media Server, add this URL to webhooks:
  - `http://localhost:5454/plex?token=plex-token-here`
- Replace `localhost` with the IP or host of the notifiarr application.
- Replace `plex-token-here` with your plex token.
- **The Notifiarr application uses the Plex token to authorize incoming webhooks.**

|Config Name|Variable Name|Note|
|---|---|---|
plex.url|`DN_PLEX_URL`|`http://localhost:32400` / local URL to your plex server|
plex.token|`DN_PLEX_TOKEN`|Required. [Must provide Plex Token](https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/) for this to work.|

### Tautulli

Only 1 Tautulli instance may be configured per client. Providing Tautulli allows Notifiarr
to use the "Friendly Name" for your Plex users and it allows you to easily enable a service check.

|Config Name|Variable Name|Note|
|---|---|---|
tautulli.name|`DN_TAUTULLI_NAME`|No Default. Setting a name enables service checks of Tautulli|
tautulli.url|`DN_TAUTULLI_URL`|No Default. Something like: `http://localhost:8181`|
tautulli.api_key|`DN_TAUTULLI_API_KEY`|No Default. Provide URL and API key if you want name maps from Tautulli|

### Service Checks

The Notifiarr client can also check URLs for health. If you set names on your
Starr apps they will be automatically checked and reports sent to Notifiarr.
If you provide a log file for service checks, those logs will no longer write
to the app log nor to console stdout.

|Config Name|Variable Name|Note|
|---|---|---|
services.log_file|`DN_SERVICES_LOG_FILE`|If a file path is provided, service check logs write there|
services.interval|`DN_SERVICES_INTERVAL`|`10m`, How often to send service states to Notifiarr; minimum: `5m`|
services.parallel|`DN_SERVICES_PARALLE`|`1`, How many services can be checked at once; 1 is plenty|

You can also create ad-hoc service checks for things like Bazarr.

|Config Name|Variable Name|Note|
|---|---|---|
service.name|`DN_SERVICE_0_NAME`|Services must have a unique name|
service.type|`DN_SERVICE_0_TYPE`|Type must be one of `http`, `tcp`, `process`|
service.check|`DN_SERVICE_0_CHECK`|The `URL`, or `host/ip:port` to check|
service.expect|`DN_SERVICE_0_EXPECT`|`200`, For HTTP, the return code to expect|
service.timeout|`DN_SERVICE_0_TIMEOUT`|`15s`, How long to wait for service response|
service.interval|`DN_SERVICE_0_INTERVAL`|`5m`, How often to check the service|

#### Process Service Checks

When `type` is set to `process`, the `expect` parameter becomes a special variable.
You may set it to `restart` to send a notification when the process restarts.
You may set it to `running` to alert if the process is found running (negative check).
You may set it to `count:min:max`. ie `count:1:2` means alert if process count is below 1 or above 2.
You may combine these with commas. ie `restart,count:1:3`.

By default `check` is the value to find in the process list. It uses a simple string match.
Unless you wrap the value in slashes, then it becomes a regex.
ie. use this `expect = "/^/usr/bin/smtpd$/"` to match an exact string.

Run `notifiarr --ps` to view the process list from Notifiarr's point of view.

## Reverse Proxy

You'll need to expose this application to the Internet, so Notifiarr.com
can make connections to it. While you can certainly poke a hole your firewall
and send the traffic directly to this app, it is recommended that you put it
behind a reverse proxy. It's pretty easy.

You'll want to tune the `upstreams` and `urlbase` settings for your environment.
If your reverse proxy IP is `192.168.3.45` then set `upstreams = ["192.168.3.45/32"]`.
The `urlbase` can be left at `/`, but change it if you serve this app from a
subfolder. We'll assume you want to serve the app from `/notifiarr/` and
it's running on `192.168.3.33` - here's a sample nginx config to do that:

```
location /notifiarr/ {
  proxy_set_header X-Forwarded-For $remote_addr;
  proxy_pass http://192.168.3.33:5454$request_uri;
}
```

Make sure the `location` path matches the `urlbase` and ends with a `/`.
That's all there is to it.

Here are two more example Nginx configs:

- [TRaSH-'s Swag](https://gist.github.com/TRaSH-/037235b0440b38c8964a2cbb64179cf3) - A drop-in for Swag users.
- [Captain's Custom](https://github.com/Go-Lift-TV/organizr-nginx/blob/master/golift/notifiarr.conf) - Fits into Captain's Go Lift setup. Not for everyone.

## Troubleshooting

- Find help on [GoLift Discord](https://golift.io/discord).
- And/or on [Notifiarr Discord](https://discord.gg/AURf8Yz).

Log files:

You can set a log file in the config. You should do that. Otherwise, find your logs here:

-   Linux: `/var/log/messages` or `/var/log/syslog` (w/ default syslog)
-   FreeBSD: `/var/log/syslog` (w/ default syslog)
-   Homebrew: `/usr/local/var/log/notifiarr.log`
-   macOS: `~/.notifiarr/notifiarr.log`
-   Windows: `<home folder>/.notifiarr/notifiarr.log`

If transfers are in a Warning or Error state they will not be extracted.

Still having problems?
[Let us know!](https://github.com/Notifiarr/notifiarr/issues/new)

## Integrations

The following fine folks are providing their services, completely free! These service
integrations are used for things like storage, building, compiling, distribution and
documentation support. This project succeeds because of them. Thank you!

<p style="text-align: center;">
<a title="PackageCloud" alt="PackageCloud" href="https://packagecloud.io"><img src="https://docs.golift.io/integrations/packagecloud.png"/></a>
<a title="GitHub" alt="GitHub" href="https://GitHub.com"><img src="https://docs.golift.io/integrations/octocat.png"/></a>
<a title="Docker Cloud" alt="Docker" href="https://cloud.docker.com"><img src="https://docs.golift.io/integrations/docker.png"/></a>
<a title="Travis-CI" alt="Travis-CI" href="https://Travis-CI.com"><img src="https://docs.golift.io/integrations/travis-ci.png"/></a>
<a title="Homebrew" alt="Homebrew" href="https://brew.sh"><img src="https://docs.golift.io/integrations/homebrew.png"/></a>
<a title="Go Lift" alt="Go Lift" href="https://golift.io"><img src="https://docs.golift.io/integrations/golift.png"/></a>
<a title="Better Uptime" alt="Go Lift" href="https://betteruptime.com"><img src="https://docs.golift.io/integrations/betteruptime.png"/></a>
</p>

## Contributing

Yes, please.

## License

[MIT](https://github.com/Notifiarr/notifiarr/blob/main/LICENSE) - Copyright (c) 2020-2022 Go Lift
