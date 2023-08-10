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