#obsidian #tools

## Quartz

このノートを公開する仕組み

https://github.com/jackyzha0/quartz

~~[[Obsidian]]のものをなるべくそのまま[[Hugo]]で使えるようにしている~~

→v4から完全にJSベースの仕組みに変わった

https://quartz.jzhao.xyz/migrating-from-Quartz-3

## いいところ

- Wikilink`[[link]]`を変換してくれる
- Github Pagesでホスト可能
- 静的サイトなのになんかSSRされたSPAになってる([million](https://million.dev/)というのを使ってるらしい)のでページ読み込みめっちゃ早い。マウスオンでのプレビューとかもできる
- 全文検索もできる

## 難点

-~~文中でハッシュタグが使えない（フロントマッターにタグを指定するしかない~~
	-→v4で可能になった。うれしい。
- 記事未作成だったり、titleが設定されてないリンクはGraph View上で日本語がうまく表示されない（ファイル名だと日本語がエスケープされてんのかな）
	- →パーセントエンコーディングをデコードするようにしたら治った。
		- [プルリク立てた](https://github.com/jackyzha0/quartz/pull/366)
		- これはv4でも治ってなかったので別途手元で修正している最中
	- フロントマッターにtitle要素を指定する必要はない。よってフロントマッター基本不要になった
- ~~手元でサーバー立ててプレビューするのがちょっと辛い（hugo-obsidianコマンドが手元で使える必要があるので、Goをインストールか[[Docker]]イメージを利用する感じになる）~~
	 -v4で解決

### v4以降の難点

- SPAでの読み込みが若干怪しく、youtubeの埋め込みがある記事は一回リロードしないときちんと読まれない時がある？
- グラフのオプション指定がなぜかデフォルトパラメーターで上書きされるバグがあったので修正している。
	- https://github.com/jackyzha0/quartz/pull/384
- GitHub Pagesのデプロイ用アクションはリポジトリから消えてたので別途作成。
	- [PR立てた](https://github.com/tomoyanonymous/quartz-research-note/commit/5f58eaba035038faafcdbbf3c51de410c021c579)
	- ビルドも早いし快調。
- 作成されてないWikilinkのリンクが有効なリンクとして表示されてる。グラフには反映されてないところ見るとレンダリング側でのミスなのかな
- サイト外に飛ぶリンクにはObsidianみたく矢印マークつけてほしい。
- Graphにタグが反映されない。これはインラインでタグ使うようにしてフロントマッターから消去したらできるのか？
	- そんなこともないみたい
- タグをインラインで先頭に書いておくスタイルだと、タグ一覧の下にまた手動で書いたタグ一覧が並んでちょっと座りが悪い
	- →`quartz.layout.ts`の部分をこんな感じでコメントアウト

```ts
beforeBody: [Component.ArticleTitle(), Component.ContentMeta(),/*Component.TagList()*/],
```

このレイアウトの仕組みよくできてんなー