# tofa on Unraid

[tofa](https://tofa.tv) is a media server for your movies and shows. One
container runs everything: the server, its own database, automatic backups, and
its own video engine for hardware transcoding. Point it at your media, claim the
server with your tofa account, and watch on the web, Apple TV, iPhone, iPad,
Android TV, and Windows.

tofa is in beta. You need a tofa account to claim a server; you can sign up at
[tofa.tv](https://tofa.tv).

## Install

Install **tofa** from the Apps tab. Two settings are worth understanding before
you click Apply, and both are filled in for you by default:

- **Config (`/data`) defaults to `/mnt/cache/appdata/tofa`.** tofa runs its own
  database in there, and databases do a lot of small writes. Paths under
  `/mnt/user` go through Unraid's share layer, which adds overhead to every one
  of those writes, even when the share lives on your cache pool. Pointing tofa
  at the pool directly avoids that.

  **Check the pool name before you click Apply.** That default assumes your
  pool is called `cache`. If yours is called something else, change the path to
  match it, for example `/mnt/main_cache/appdata/tofa`. Unraid does not fail on
  a path that does not exist. It creates the folder on its RAM-backed root
  instead, the container starts, everything looks normal, and the whole
  database is gone the next time the server reboots. Your pool names are on the
  Main tab.
- **Media (`/media`) defaults to `/mnt/user/media`.** Point this at wherever
  your library lives. Set the mode to Read Only if tofa should never delete
  files from disk.

`PUID` and `PGID` default to `99` and `100`, Unraid's `nobody` user and `users`
group, which own appdata and shares out of the box. tofa fixes ownership of its
own data folder on every start. It never changes ownership of your media.

The container uses host networking. That is what lets devices on your network
connect to tofa directly, and lets tofa ask your router for a port for remote
access. tofa listens on port 33333, which avoids the default ports of other
media servers, so it can run alongside them.

## Claim your server

Open a browser on any device on your network:

```
http://YOUR-UNRAID-IP:33333
```

Sign in with your tofa account and claim the server. The first start takes a
minute or two while tofa sets up its database. Your server picks up its secure
certificate on its own moments after the claim, so there is no need to restart
the container.

Then add a library in the web interface and point it at your media inside the
container: `/media`, or a subfolder like `/media/Movies`. tofa scans, matches,
and fills in artwork on its own.

## Hardware transcoding

- **Intel or AMD, including iGPUs**: in the container settings, turn on Advanced
  View, click **Add another Path, Port, Variable, Label or Device**, choose
  **Device**, and enter `/dev/dri` as the value. Then use Settings, then
  Transcoding, then Test hardware in the tofa web interface.
- **NVIDIA**: install the *Nvidia Driver* plugin from Community Applications,
  set Extra Parameters to `--init --runtime=nvidia`, and add two variables:
  `NVIDIA_VISIBLE_DEVICES` = `all` and `NVIDIA_DRIVER_CAPABILITIES` =
  `compute,video,utility`.

## Updates

tofa shows an update banner in the web interface when a new build is out. To
apply it, use **force update** on the container in the Docker tab (turn on
Advanced View to see it). Your data and settings live in your appdata path and
survive updates.

## If something is off

- **Apply fails with `docker: bad format for path`.** An early version of this
  template shipped an empty Device entry for `/dev/dri`, and Unraid passes an
  empty device straight to docker, which refuses it. Edit the container, turn on
  Advanced View, and remove the empty Device row (or fill it in with `/dev/dri`
  if you have an Intel or AMD GPU), then Apply. The row is gone from the
  template here, but Community Applications serves its own cached copy and can
  keep offering the old one for a day or two, so check for the row before you
  click Apply. Unraid also saves your own copy of the template once you have
  applied it, so an install that already failed keeps the row until you remove
  it by hand.
- **tofa says your data is not being saved.** The folder mapped to `/data` is
  in memory rather than on a disk, so everything tofa stores goes away on the
  next reboot. The usual cause is a pool name that is not `cache`: the default
  path `/mnt/cache/appdata/tofa` never existed, and Unraid created it on its
  RAM-backed root. Run `ls /mnt` on the host to see your real pool names. Copy
  your existing data across before you change anything, and stop tofa first so
  the database is not copied mid-write: stop the container from the Docker tab,
  then run `docker cp tofa:/data/. /mnt/main_cache/appdata/tofa/` and
  `chown -R nobody:users /mnt/main_cache/appdata/tofa`. Then edit the container,
  point Config at the real pool path, and start it again. Do this before the
  next reboot, which is what takes the RAM copy with it. Community Applications
  serves its own cached copy of this template, so a fresh install can still
  offer the older description for a day or two after we change it.
- **The container stops right after starting.** Check its log from the Docker
  tab. If you see a permissions message, confirm `PUID` and `PGID` are set to
  99 and 100 and that your appdata path is a real disk path like
  `/mnt/cache/appdata/tofa`.
- **My media isn't showing up.** Check that the `/media` mapping points at the
  right share and that the `nobody` user can read those files. Files moved onto
  the array by other tools occasionally end up with odd ownership; Unraid's
  *New Permissions* tool on the Tools tab resets a share to `nobody:users`.
- **Playback stutters on big files.** Set up hardware transcoding above, and
  keep tofa's appdata on a pool device rather than `/mnt/user`.
- **Port 33333 is already in use.** Something else on the host claimed it. Stop
  that service, or switch the container off host networking and map a different
  port. Off host networking tofa cannot detect your server's address by itself,
  so also set your Unraid server's LAN IP in the `ADVERTISED_LAN_IP` variable,
  or under Settings, then Remote Access in the app. That keeps devices at home
  connecting directly.

## Getting help

Once you have a tofa account, the support portal is at
<https://accounts.tofa.tv/support>, and it is also linked from Settings inside
the app. Diagnostics from your server attach to a ticket in one click, which is
usually the fastest route to an answer.

## About this repository

`templates/tofa.xml` is the Community Applications template for the tofa media
server. Issues with the template itself can be raised here.
