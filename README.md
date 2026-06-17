# Cesium Terrain Server

[![tests](https://github.com/brickhouse-tech/cesium-terrain-server/actions/workflows/tests.yml/badge.svg)](https://github.com/brickhouse-tech/cesium-terrain-server/actions/workflows/tests.yml)
[![GitHub stars](https://img.shields.io/github/stars/brickhouse-tech/cesium-terrain-server.svg?style=social)](https://github.com/brickhouse-tech/cesium-terrain-server)
[![Go Report Card](https://goreportcard.com/badge/github.com/brickhouse-tech/cesium-terrain-server)](https://goreportcard.com/report/github.com/brickhouse-tech/cesium-terrain-server)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0)

**A lightweight Go server for hosting [CesiumJS](http://cesiumjs.org/) terrain tilesets.** Serves filesystem-based terrain tiles for use with [`CesiumTerrainProvider`](http://cesiumjs.org/Cesium/Build/Documentation/CesiumTerrainProvider.html), with optional Memcached caching and Docker support.

Built for use with tilesets created by [Cesium Terrain Builder](https://github.com/geo-data/cesium-terrain-builder).

## Install

```bash
go get github.com/brickhouse-tech/cesium-terrain-server/cmd/cesium-terrain-server
```

Or use Docker:

```bash
docker pull brickhouse-tech/cesium-terrain-server
```

## Quick Start

Given a terrain tileset at `/data/tilesets/terrain/srtm/`:

```bash
cesium-terrain-server -dir /data/tilesets/terrain -port 8080
```

Tiles are then available at `http://localhost:8080/tilesets/srtm/` — point your `CesiumTerrainProvider` there.

## Usage

```
cesium-terrain-server [options]

  -dir="."                  Root directory containing tileset directories
  -port=8000                Server port
  -base-terrain-url="/tilesets"  URL prefix for all tilesets
  -cache-limit=1.00MB       Max resource size for caching (supports kB, MB, GB, TB)
  -memcached=""              Memcached address for tile caching (e.g. localhost:11211)
  -web-dir=""                Directory for serving static files
  -log-level=notice          Log level: crit, err, notice, debug
  -no-request-log=false      Suppress client request logging
```

### Adding Tilesets

Just drop tileset directories under your root dir. For example:

```
/data/tilesets/terrain/
├── srtm/          → http://localhost:8080/tilesets/srtm/
├── lidar/         → http://localhost:8080/tilesets/lidar/
└── custom/        → http://localhost:8080/tilesets/custom/
```

### `layer.json`

`CesiumTerrainProvider` requires a `layer.json` file. If one exists in your tileset root, it's served as-is. Otherwise, the server returns a sensible default.

### Root Tiles

Cesium requires both `0/0/0.terrain` and `0/1/0.terrain`. If one is missing (common when your dataset doesn't cross the prime meridian), the server returns a blank tile automatically.

### Memcached Caching

Use with a reverse proxy (e.g. Nginx) for production caching:

```bash
cesium-terrain-server -dir /data/tilesets/terrain -memcached memcache.me.org:11211
```

<details>
<summary>Example Nginx configuration</summary>

```nginx
server {
    listen 80;
    server_name localhost;
    root /var/www/app;
    index index.html;

    location /tilesets/ {
        set            $memcached_key "tiles$request_uri";
        memcached_pass memcached:11211;
        error_page     404 502 504 = @fallback;
        add_header Access-Control-Allow-Origin "*";

        location ~* \.terrain$ {
            add_header Content-Encoding gzip;
        }
    }

    location @fallback {
        proxy_pass     http://tiles:8000;
        proxy_set_header X-Memcache-Key $memcached_key;
    }
}
```

</details>

## Development

```bash
# Build from source
make

# Build Docker image
make docker-local
```

Requires Go with `GOPATH`, `GOROOT`, and `GOBIN` set. See the Makefile for details.

## Contributing

Please report bugs via the [GitHub issue tracker](https://github.com/brickhouse-tech/cesium-terrain-server/issues). Pull requests welcome!

## Sponsor

If you find this project useful, consider [sponsoring @brickhouse-tech](https://github.com/sponsors/brickhouse-tech) to support ongoing maintenance and development. ❤️

## License

[Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0)

## Credits

Originally created by Homme Zwaagstra at [GeoData](https://github.com/geo-data).
