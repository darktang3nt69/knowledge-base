### Setup Folder as shown below:
```
data
├── torrents
│   ├── books
│   ├── movies
│   ├── music
│   └── tv
├── usenet
│   ├── incomplete
│   └── complete
│        ├── books
│        ├── movies
│        ├── music
│        └── tv
└── media
     ├── books
     ├── movies
     ├── music
     └── tv

```

### Breakdown of the Folder Structure[](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/#breakdown-of-the-folder-structure "Permanent link")

#### Torrent clients[](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/#torrent-clients "Permanent link")

qBittorrent, Deluge, ruTorrent

The reason why we use `/data/torrents` for the torrent client is because it only needs access to the torrent files. In the torrent software settings, you’ll need to reconfigure paths and you can sort into sub-folders like `/data/torrents/{tv|movies|music}`.

```
data
└── torrents
     ├── books
     ├── movies
     ├── music
     └── tv

```

`Container Path:` => `/data/torrents/`

`Host Path:` => `/<path_to_data>/data/torrents/`

#### Usenet clients[](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/#usenet-clients "Permanent link")

NZBGet or SABnzbd

The reason why we use `/data/usenet` for the usenet client is because it only needs access to the usenet files. In the usenet software settings, you’ll need to reconfigure paths and you can sort into sub-folders like `/data/usenet/complete/{tv|movies|music}`.

```
data
└── usenet
    ├── incomplete
    └── complete
         ├── books
         ├── movies
         ├── music
         └── tv

```

`Container Path:` => `/data/usenet/`

`Host Path:` => `/<path_to_data>/data/usenet/`

#### The Starr Apps[](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/#the-starr-apps "Permanent link")

Sonarr, Radarr, Readarr and Lidarr

Sonarr, Radarr, Readarr and Lidarr gets access to everything using `/data` because the download folder(s) and media folder will look like and be one file system. Hardlinks will work and moves will be atomic, instead of copy + delete.

```
data
├── torrents
│   ├── books
│   ├── movies
│   ├── music
│   └── tv
├── usenet
│   ├── incomplete
│   └── complete
│        ├── books
│        ├── movies
│        ├── music
│        └── tv
└── media
     ├── books
     ├── movies
     ├── music
     └── tv

```

`Container Path:` => `/data`

`Host Path:` => `/<path_to_data>/data/`

#### Media Server[](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/#media-server "Permanent link")

Plex, Emby, JellyFin and Bazarr

Plex, Emby, JellyFin and Bazarr only needs access to your media library using `/data/media`, which can have any number of sub folders like Movies, Kids Movies, TV, Documentary TV and/or Music as sub folders.

```
data
└── media
     ├── movies
     ├── music
     ├── books
     └── tv

```

`Container Path:` => `/data/media`

`Host Path:` => `/<path_to_data>/data/media/`

---

### Permissions[](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/#permissions "Permanent link")

Recursively chown user and group and Recursively chmod to 775/664

```
sudo chown -R $USER:$USER /data
sudo chmod -R a=,a+rX,u+w,g+w /data
```

### Read next: [[The Arr stack]]