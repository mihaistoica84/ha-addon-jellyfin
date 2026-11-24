# Jellyfin Server (Next) Add-on

![Supports amd64 Architecture][amd64-shield]
![Supports aarch64 Architecture][aarch64-shield]

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg

This [Home Assistant](https://www.home-assistant.io/addons/) add-on installs the
[Jellyfin](https://jellyfin.org/) 12.4 server and web client, built from source
with the fixes in [`patches/`](./patches) applied on top of the official release:

| Patch | Fixes |
|---|---|
| `server/0001-reload-user-data-before-saving-playback-state.patch` ([jellyfin#18366](https://github.com/jellyfin/jellyfin/pull/18366)) | Favorites (and ratings/played state) changed on the currently playing item being reverted by the next playback progress report ([jellyfin#14981](https://github.com/jellyfin/jellyfin/issues/14981)) |
| `web/0001-skip-to-next-item-when-playing-item-is-deleted.patch` | Deleting the currently playing item (or its album, season or series) in the web client kept playing the deleted media from the buffer; playback now moves to the next queued item, or stops if there is none |
| `web/0002-ctrl-shift-d-deletes-playing-item.patch` | New keyboard shortcut: **Ctrl+Shift+D** in the web client deletes the currently playing item, after the usual delete confirmation (Enter confirms, Esc cancels) |
| `web/0003-add-to-playlist-in-batches.patch` | Saving a long play queue (about 240+ items) to an existing playlist with the player's Save button failed with "Error adding to playlist" (HTTP 414, URL too long); items are now added in batches of 100 |
| `web/0004-follow-currently-playing-item-in-lists.patch` | Lists (library songs, playlists, albums, the play queue) did not follow playback: the row of the playing track is now highlighted, scrolled into view when it is off-screen and focused, both when a new track starts and when the list is shown again |

ffmpeg and hardware acceleration drivers come unchanged from the official
`jellyfin/jellyfin` image.

Server is exposed on port `8096`, and must be accessed directly.

## Running next to the original Jellyfin Server add-on

This add-on stores its configuration and caches in `/share/jellyfin-next`,
separately from the original add-on's `/share/jellyfin`, so the original
installation is never modified. Both add-ons use port `8096`, so run only
one of them at a time (or change the host port of one of them).

To start with your existing libraries, users and watch history, stop both
add-ons and copy `/share/jellyfin` to `/share/jellyfin-next` before the
first start of this add-on. Jellyfin upgrades the copied database on first
start; the original folder stays usable by the original add-on.

Data in `/share/jellyfin-next` is not deleted when the add-on is uninstalled,
and must be deleted manually. This is intentional.

## Adding a patch

1. Put a `git format-patch` file into `patches/server/` (applicable to the
   [jellyfin](https://github.com/jellyfin/jellyfin) tag in `JELLYFIN_REF`) or
   `patches/web/` (applicable to the [jellyfin-web](https://github.com/jellyfin/jellyfin-web)
   tag in `JELLYFIN_WEB_REF`), see [`Dockerfile`](./Dockerfile). Each folder needs
   at least one patch; patches are applied in file name order.
2. Bump `version` in `config.yaml` and push a matching `v*` tag to publish the image.
