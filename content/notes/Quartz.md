---
title: Quartz
tag: 
- obsidian
---

## Quartz

このノートを公開する仕組み

https://github.com/jackyzha0/quartz

ObsidianのものをなるべくそのままHugoで使えるようにしている

## いいところ

- Hugoのテンプレートの黒魔術を使ってWikilink`[[link]]`を相互リンクに変換してくれる
- なんかSPAになってる([million](https://million.dev/)というのを使ってるらしい)のでめっちゃ早い
- 全文検索もできる

## 難点

- インラインでハッシュタグが使えない（フロントマッターにタグを指定するしかない）
- 手元でサーバー立ててプレビューするのがちょっと辛い（hugo-obsidianコマンドが手元で使える必要があるので、GoをインストールかDockerイメージを利用する感じになる）