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
  at the pool directly avoids that. If your pool has a different name, adjust
  the path.
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
  if you have an Intel or AMD GPU), then Apply. New installs no longer have the
  row.
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
