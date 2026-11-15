---
title: "Extra packages and the CleanWRT package feed"
description: "Install more packages on the Arcadyan AW1000 with apk, from the official OpenWrt repositories and the CleanWRT feed (qBittorrent, the Syncthing switch)."
---

# Extra packages

## From the official repositories

The official OpenWrt 25.12.5 repositories are enabled on every router:

```sh
apk update
apk add <package>
```

Examples that are not part of the image: `mwan3` and `luci-app-mwan3` (multi-WAN),
`transmission-daemon` (another download client), `aria2`, `luci-app-acme` (certificates).

Packages you add yourself stay unpinned and are yours to manage.

## The CleanWRT feed (`cw-opt`)

Every router also lists this project's own signed `apk` feed, for what the official repositories
do not carry and the image leaves out to stay small:

| Package | What it is |
|---|---|
| `qbittorrent`, `luci-app-qbittorrent` | the download client and its LuCI page, with the Qt6 libraries it needs |
| `luci-app-syncthing` | the on/off switch for Syncthing in LuCI (`syncthing` itself is in the official repositories) |

```sh
apk update
apk add qbittorrent luci-app-qbittorrent
```

The feed lives at `https://shanelkapramuditha.github.io/cleanwrt-aw1000/feed/` and is listed in
`/etc/apk/repositories.d/cw-opt.list`. Its index is signed with the project's key, which every
image trusts (`/etc/apk/keys/public-key.pem`), so `apk` refuses a feed signed with anything else.
The feed also carries the community packages the image is built from (QModem, 3ginfo-lite and
the rest), at the versions this project patched and tested.

### How the feed is made (for the project)

The `feed` workflow (*Actions → feed → Run workflow*) builds the image together with the feed
set, signs and verifies the index (`tools/cw-publish-feed.sh`: the signature against the shipped
key, one version per package, and an install check against the official repositories), and
publishes it to `feed/` on the site. Run it:

- once before the first image that lists the feed: the build workflow refuses to build a draft
  while the feed does not answer, because a dead feed entry breaks `apk update` on every router;
- again when qBittorrent, the Syncthing switch or the OpenWrt base changes.

A normal release does not need it, so the image build stays short.
