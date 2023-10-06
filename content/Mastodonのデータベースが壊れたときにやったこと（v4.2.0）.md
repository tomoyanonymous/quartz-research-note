#mastodon #self-hosted 

- 2023/10/06になんかタイムラインが更新されなくなったことに気づく。
- WebもStreamingも生きてるが、Sidekiqがほぼ全て失敗になっていた
- こういう感じのエラーが出ていたので、なんかDBが壊れてるっぽいことに気づく


```
ActiveRecord::StatementInvalid: PG::IndexCorrupted: ERROR: right sibling's left-link doesn't match: block 3530 links to 3531 instead of expected 1 in index "index_account_stats_on_last_status_at_and_account_id"
```

- pgheroには特にslow queryなど出てなかった
	- むしろクエリのエラーになっていた`index_statuses_public_20200119`はunusedになってたりしてなんか怪しい
- 多分その3日前ぐらいに更新した4.2.0以降に何かが発生したのだが、何が起きたのか謎（`db:migrate`はちゃんとやった）

Mastodon管理者マニュアルの[Database Index Corruption](https://docs.joinmastodon.org/admin/troubleshooting/index-corruption/)という項がまさにそれっぽかったので、`docker-compose run --rm web bin/tootctl maintenance fix-duplicates`を試し、何個か修正されたような気がしたと思ったが、特に根本的な解決にはならず

→後から色々調べると、この`tootctl maintenance fix-duplicates`は3.2.2あたりに起因するエラーを直すためのコマンドだったらしく、今回壊れたインデックスを直してくれてなかったっぽい

DBが壊れてるかどうかの判定を行うamcheckのやり方も、Sidekiqでエラーが出てるインデックスを検索に指定しないと意味がないので注意
### うまくいった直し方

- web,streaming,sidekiqのインスタンスを止める（私は[[Portainer]]で管理してるのでそれで止めた）
- dbインスタンスにターミナルで入る
- `su postgres`でユーザー変更
- `psql`の中で`REINDEX`を行うか、`reindexdb`コマンドを使う。私はpsqlなんもわからん人間だったので後者を使った
- データベース名は`docker-compose.yml`で指定しているもので、私は`mastodon_db`としている

```sh
reindexdb -d mastodon_db
```

これでDBを全部再インデックスしてくれる、、が、途中でエラー吐いて止まってしまい、Sidekiqで止まってる目的のインデックスを修正しないで落ちてしまった

ので手動でインデックス指定して修正。私の場合はこの2つを直したら直った

```sh 
reindexdb -d mastodon_db -i index_statuses_public_20200119
reindexdb -d mastodon_db -i index_account_stats_on_last_status_at_and_account_id
```

Web、Streaming、Sidekiqのコンテナを再起動。Sidekiqの管理画面に行って、まだデータベースが壊れているようなら出なくなるまで同様の手順を繰り返すと多分治る