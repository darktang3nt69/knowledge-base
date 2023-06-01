This basic example is based on the use of docker images

Keep in mind the path are setup so it works with hardlinks and instant moves.

More info [HERE](https://trash-guides.info/Hardlinks/Hardlinks-and-Instant-Moves/)

### Additional Config
1. Set container dns to 1.1.1.1 and 8.8.8.8
2. 


### Bad path suggestion[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#bad-path-suggestion "Permanent link")

The default path setup suggested by some docker developers that encourages people to use mounts like `/movies`, `/tv`, `/books` or `/downloads` is very suboptimal and it makes them look like two or three file systems, even if they aren’t (_Because of how Docker’s volumes work_). It is the easiest way to get started. While easy to use, it has a major drawback. Mainly losing the ability to hardlink or instant move, resulting in a slower and more I/O intensive copy + delete is used.

But you're able to change this, by not using the pre-defined/recommended paths like:

- `/downloads` => `/data/downloads`, `/data/usenet`, `/data/torrents`
- `/movies` => `/data/media/movies`
- `/tv` => `/data/media/tv`

---

Note

Settings that aren't covered means you can change them to your own liking or just leave them on default.

---

## Downloads[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#downloads "Permanent link")

`Tools` => `Options` => `Downloads` (Or click on the cogwheel to access the options)

### When adding a torrent[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#when-adding-a-torrent "Permanent link")

[![!When adding a torrent](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-downloads-when-adding-a-torrent.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-downloads-when-adding-a-torrent.png)

1. For consistency with other torrents I recommend leaving this on `Original`.
    
    **Suggested: `Original`**
    
2. Delete the .torrent file after it has been added to qBittorrent.
    
    **Suggested: `Personal preference`**
    
3. Pre-allocated disk space for the added torrents, this limits fragmentation and also makes sure if you use a cache drive or a feeder disk that the space is available.
    
    **Suggested: `Enabled`**
    
    Warning
    
    Do not set Pre-allocated disk space if you are using ZFS as your filesystem as ZFS [does not support fallocate](https://github.com/openzfs/zfs/issues/326)
    

### Saving Management[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#saving-management "Permanent link")

[![Saving Management](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-downloads-saving-management.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-downloads-saving-management.png)

1. Make sure this is set to `Automatic`. Your downloads will not go into the category folder otherwise.
    
    **Suggested: `Automatic`**
    
2. This helps you to manage your file location based on categories.
    
    **Suggested: `Enabled`**
    
3. Same as `Step 2`
    
    **Suggested: `Enabled`**
    
4. Your download root path (Download folder/location).
    
    **Read the `ATTENTION` block below**
    
5. If you enable this, your incomplete downloads will be placed in this directory until completed. This could be useful if you want your downloads to use a separate SSD/Feeder disk[1](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#fn:1), but this also results in extra unnecessary moves or in worse cases a slower and more I/O intensive copy + delete.
    
    **Suggested: `Personal preference`**
    

#### ATTENTION[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#attention "Permanent link")

ATTENTION

1. You set your download location in your download client
2. Your download client **ONLY** downloads to your download folder/location.
3. And you tell Radarr where you want your clean media library
4. Radarr imports from your download location (copy/move/hardlink) to your media folder/library
5. Plex, Emby, JellyFin or Kodi should **ONLY** have access to your media folder/library

![‼](https://cdn.jsdelivr.net/gh/jdecked/twemoji@14.1.2/assets/svg/203c.svg ":bangbang:") ****Your Download and Media Library should be **NEVER** the same locations**** ![‼](https://cdn.jsdelivr.net/gh/jdecked/twemoji@14.1.2/assets/svg/203c.svg ":bangbang:")

---

## Connection[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#connection "Permanent link")

### Listening Port[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#listening-port "Permanent link")

[![!Listening Port](https://trash-guides.info/Downloaders/qBittorrent/images/qbt-options-connection.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qbt-options-connection.png)

1. Set this to TCP for the best performance
    
    **Suggested: `TCP`**
    
2. Your port used for incoming connections, this is the port you opened in your router/firewall or port forwarded at your VPN provider to make sure you're connectable.
    
    **Suggested: `The port you opened in your router/firewall or port forwarded at your VPN provider`**
    
3. This should be disabled in your router for several security reasons.
    
    **Suggested: `Disabled`**
    
4. Make sure this is disabled so you don't mess up the forwarded port.
    
    **Suggested: `Disabled`**
    

### Connections Limits[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#connections-limits "Permanent link")

[![!Connections Limits](https://trash-guides.info/Downloaders/qBittorrent/images/qbt-options-connection-connections-limits.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qbt-options-connection-connections-limits.png)

The best settings for this depends on many factors so I won't be covering this.

**Suggested: `personal preference based on your setup and connection.`**

### Proxy Server[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#proxy-server "Permanent link")

[![!Proxy Server](https://trash-guides.info/Downloaders/qBittorrent/images/qbt-options-connection-proxy-server.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qbt-options-connection-proxy-server.png)

This is where you would add for example your SOCKS5 settings from your VPN provider.

**Suggested: `I personally don't recommend this unsecure option being it's un-encrypted and only spoofs your IP.`**

---

## Speed[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#speed "Permanent link")

### Global Rate Limits[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#global-rate-limits "Permanent link")

[![!Global Rate Limits](https://trash-guides.info/Downloaders/qBittorrent/images/qbt-options-speed-global-rate-limits.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qbt-options-speed-global-rate-limits.png)

Here you can set your global rate limits, meaning your maximum download/upload speed used by qBittorrent. (For all torrents)

The best settings depends on many factors.

- Your ISP speed.
- Your hardware used.
- Bandwidth needed by other services in your home network.
    
    **Suggested: `For a home connection that you use with others it's best practice to set the upload/download rate to about 70-80% of your maximum upload/download speed.`**
    

### Alternative Rate Limits[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#alternative-rate-limits "Permanent link")

[![!Alternative Rate Limits](https://trash-guides.info/Downloaders/qBittorrent/images/qbt-options-speed-alternative-rate-limits.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qbt-options-speed-alternative-rate-limits.png)

When enabled, it basically does the same as above, but with the option to setup a schedule.

Examples:

- Limit your upload/download rate during daytime when you make most use of it, and unlimited it during nighttime when no one is using the connection.
- If you have an internet connection that's limited during specific hours (unlimited bandwidth during the night, but limited during the day)
    
    **Suggested: `Personal preference`**
    

### Rate Limits Settings[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#rate-limits-settings "Permanent link")

[![!Rate Limits Settings](https://trash-guides.info/Downloaders/qBittorrent/images/qbt-options-speed-rate-limits-settings.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qbt-options-speed-rate-limits-settings.png)

Not going to cover the technical part of what it does, but the following settings are recommended for best speeds (in most cases).

1. Prevents you from being flooded if the uTP protocol is used for any reason.
    
    **Suggested: `Enabled`**
    
2. Apply rate limit to transport overhead
    
    **Suggested: `Disabled`**
    
3. Apply rate limit to peers on LAN
    
    **Suggested: `Enabled`**
    

---

## Bittorrent[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#bittorrent "Permanent link")

### Privacy[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#privacy "Permanent link")

[![!Privacy](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-bittorrent-privacy.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-bittorrent-privacy.png)

1. These settings are mainly used for public trackers (and should be enabled for them) and not for private trackers, decent private trackers use a private flag where they ignore these settings.
    
    **Suggested: `Personal preference`**
    
2. Recommended setting `Allow encryption` rather than enforcing it allows more peers to connect and is recommended on underpowered systems as it will allow for lower overhead.
    
    **Suggested: `Allow encryption`**
    
3. Anonymous mode hides clients (qBittorrent) fingerprint from the peer-ID, sets the ‘User-Agent’ to Null and it doesn’t share your IP-address directly with trackers (though peers will still see your IP address). If using private trackers, it's recommended to `disable` this. I also got reports from people who are using this that they had worse speeds.
    
    **Suggested: `Disabled`**
    

### Torrent Queueing[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#torrent-queueing "Permanent link")

[![!Torrent Queueing](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-bittorrent-torrent-queueing.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-bittorrent-torrent-queueing.png)

These options allow you to control the number of active torrents being downloaded and uploaded.

**Suggested: `personal preference based on your setup and connection.`**

### Seeding Limits[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#seeding-limits "Permanent link")

[![!Seeding Limits](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-bittorrent-seeding-limits.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-bittorrent-seeding-limits.png)

1. Your maximum seeding ratio preference. (When both ratio and seeding time are enabled it will trigger the action on whatever happens first.)
    
    **Suggested: `Disabled`**
    
2. Your maximum seeding time preference (When both ratio and seeding time are enabled it will trigger the action on whatever happens first.)
    
    **Suggested: `Disabled`**
    
3. What to do when ratio or seeding time is reached.
    
    **Suggested: `Paused and Disabled`**
    

Tip

Personally, I recommend using the seeding goals in your Starr Apps indexer settings (enable advanced), or use [qBit Manage](https://trash-guides.info/Downloaders/qBittorrent/3rd-party-tools/#qbit-manage)

### Automatically add these trackers to new downloads[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#automatically-add-these-trackers-to-new-downloads "Permanent link")

[![!Automatically add these trackers to new downloads](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-bittorrent-automatically-add-these-trackers.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-bittorrent-automatically-add-these-trackers.png)

**Recommendation: `Disabled`**

Warning

![‼](https://cdn.jsdelivr.net/gh/jdecked/twemoji@14.1.2/assets/svg/203c.svg ":bangbang:") **NEVER USE THIS OPTION ON (Semi-)PRIVATE TRACKERS** ![‼](https://cdn.jsdelivr.net/gh/jdecked/twemoji@14.1.2/assets/svg/203c.svg ":bangbang:")

---

## Web UI[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#web-ui "Permanent link")

### Authentication[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#authentication "Permanent link")

[![!Authentication](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-webui-authentication.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-webui-authentication.png)

1. When enabled there will be no authentication required for clients on localhost.
2. When enabled there will be no authentication required for clients in the `step.3` whitelist.
3. Add all IP subnets that you want to bypass authentication.

### Security[](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#security "Permanent link")

[![!Security](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-webui-security.png)](https://trash-guides.info/Downloaders/qBittorrent/images/qb-options-webui-security.png)

1. In some cases when this is enabled it could result in issues.
    
    **Suggested: `Disabled`**
    

---

Questions or Suggestions?

If you have questions or suggestions click the chat badge to join the Discord Support Channel where you can ask your questions directly and get live support.

[![Discord chat](https://img.shields.io/discord/492590071455940612?style=for-the-badge&color=4051B5&logo=discord)](https://trash-guides.info/discord)

---

1. If you use unRaid then you don't need this since you can make use of the default cache drive option. [↩](https://trash-guides.info/Downloaders/qBittorrent/Basic-Setup/#fnref:1 "Jump back to footnote 1 in the text")