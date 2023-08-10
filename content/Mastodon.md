---
title: Mastodon
tags:
- software
- server
- self-hosted
---

分散型SNS。

個人インスタンス https://social.matsuuratomoya.com を運用している。

### 運用のメモ

運用は省エネにしたいのでDocker。

リバースプロキシはNginxでチュートリアル通りにやってるが、それを[[Cloudflare Tunnel]]で外に公開している。

本当はリバースプロキシは[[Traefik]]にしたい、というかSwarmとかk3sとかでクラスタにしたい（無停止アップデートがしたい）

#### 定期メンテナンス

大体メディアキャッシュが50GBくらいある。

毎日一回実行

```bash
#!/bin/bash
cd /home/tomoya/mastodon
/usr/bin/docker-compose run --rm web bin/tootctl media remove --days 7 --concurrency 2
/usr/bin/docker-compose run --rm web bin/tootctl media remove --remove-headers --days 7 --concurrency 2;
/usr/bin/docker-compose run --rm web bin/tootctl preview_cards remove --days 4;
/usr/bin/docker-compose run --rm web bin/tootctl statuses remove --days 4;
/usr/bin/docker-compose run --rm web bin/tootctl media remove-orphans;
```

一応ElasticSearchも追加してるのでこれも

```bash
/usr/bin/docker-compose run --rm web bin/tootctl search deploy
```

あと[[Fedifetcher]]でリモートのリプライなどを追加している

```bash
/usr/bin/docker run -v fedifetcher-artifacts:/app/artifacts --rm --name fedifetcher -it ghcr.io/nanos/fedifetcher:latest --access-token=XXXXXXXXXXXXXXX --server=social.matsuuratomoya.com --home-timeline-length=200 --max-followings=90 --reply-interval-in-hours=6 --lock-hours=1
```