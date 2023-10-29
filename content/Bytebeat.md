#programming #music #sound 



[Algorithmic symphonies from one line of code -- how and why?(2011)](http://countercomplex.blogspot.com/2011/10/algorithmic-symphonies-from-one-line-of.html)


https://youtu.be/tCRPUv8V22o

Bytebeatは2011年に[[viznut]]がYoutube上の動画で公開し、自身のブログの解説などで広がっていった、短いプログラムでオーディオを生成する技法。

[Algorithmic symphonies from one line of code -- how and why?(2011)](http://countercomplex.blogspot.com/2011/10/algorithmic-symphonies-from-one-line-of.html)

その後、Webブラウザ上でも同様のコードを実行できる環境がいくつか誕生

[HTML5 Bytebeat](https://greggman.com/downloads/examples/html5bytebeat/html5bytebeat.html)

[Bytebeat Composer](https://sarpnt.github.io/bytebeat-composer)

---

Bytebeatは元々次のようなC言語のプログラムで作られてた。

```c
main(t){for(;;t++)putchar(((t<<1)^((t<<1)+(t>>7)&t>>12))|t>>(4-(1^7&(t>>19)))|t>>7);}
```

このC言語のコードは極限まで圧縮されているのでもうちょっと丁寧に書くとこうなります。

```c
int main(int t){
    for(;;t++){
    putchar(((t<<1)^((t<<1)+(t>>7)&t>>12))|t>>(4-(1^7&(t>>19)))|t>>7);
    }
}
```

これをLinuxの、昔なら`/dev/dsp`、今なら`aplay`のようなパイプで直接音声波形を流し込めるものを使って音を鳴らしていた。

macOSでやろうとするなら、`ffmpeg`に付属する`ffplay`で次のようなコードで書ける

```sh
program | ffplay -f u8 -i pipe:0 -ar 44k -ac 1
```

どうせならC言語使わずにデータを生成したいが、シェルスクリプトで直接バイナリを扱うのは死ぬほどだるい（`printf`コマンドや`bc`であれこれすれば不可能でもないが、結局ファイルを一度経由しないと厳しい）

ので、Node.jsでやるとこういう感じでできる

```js
const sample_rate = 8000;
const seconds = 1;
const length = sample_rate * seconds;
//メインの曲の生成部
const bytebeat = t =>
    (((t >> 10 ^ t >> 11) % 5) * t >> 16) * ((t >> 14 & 3 ^ t >> 15 & 1) + 1) * t % 99 + ((3 + (t >> 14 & 3) - (t >> 16 & 1)) / 3 * t % 99 & 64);
let t = 0;

const mainProcess = () =>{
    const data = Uint8Array.from({ length: length },
        (v, _t) => {
            const res = bytebeat(t);
            t += 1;
            return res
        }
    );
    process.stdout.write(data);
};
//場合によってはインターバルを少し短くしないとデータ不足で落ちることあり
setInterval(mainProcess,seconds / 1000.0);
```

これがうまくいくのはJavascriptの整数変換処理が32bitになったりするためなのだが、詳しいことは[授業資料](https://teach.matsuuratomoya.com/docs/2023/mediaart-programming2/4/)に書いた。割愛すると3238年間を超えなければ連続再生しても大丈夫ということになるっぽい。