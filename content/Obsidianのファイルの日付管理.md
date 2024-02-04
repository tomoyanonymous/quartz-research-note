#obsidian #note

参考：
[Quartz V4のカスタマイズ](https://namaraii.com/notes/quartz_v4_customize) 

- `quartz.layout.ts`と`ContentMeta.ts`のカスタムである程度制御できるが、*そのファイルをはじめに作った日*はGithub Actionsで公開してるとdeployした日になってしまう
- なので、modified dateの方が古くなってしまうみたいなことが起きる
- ローカルで`npx quartz build`する分には問題ない
	- 自前サーバーでホストしてる分には回避できるかも
	


