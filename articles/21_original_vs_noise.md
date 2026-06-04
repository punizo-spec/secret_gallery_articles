---
title: "画像に載せた「ノイズ」は、生成AI対策として有効なのか？"
emoji: "♈️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["python"]
published: true # trueを指定する
published_at: 2026-06-08 00:00
---


## 1. はじめに
生成AIの学習対策について調べているとき、画像に「ノイズ」を乗せることが有効だ、という記事をお見かけした。
<br>
ぷに蔵が読んだ記事の中では、「Glazeが有効」と紹介されているけど、同じ記事内で
### **Glazeでノイズを乗せると時間がかかるので、ペイントソフトでノイズをかけることも有効**
というような説明がされていることもあった。
<br>
<br>

これに少し疑問を持ってしまった・・・。
<br>
<br>

そもそも、ペイントソフトなどでかける「ノイズ」は、画像にザラつきや粒状感を加えるテクスチャ加工に近いよね。
<br>
つまり、小難しく言うなら、画像の画素値にランダムな揺らぎを加える、ピクセル空間のランダム摂動だと思う。
<br>

一方で、Glazeが行っている「ノイズ」は、**モデルの特徴表現やスタイル認識を特定方向にずらすための「敵対的摂動」。**
<br>
<br>

この二つ、同じ「ノイズ」という言葉だけでは説明できない技術の違いがある。（良い悪いの話ではなくて、「違うものだよ」と認識していただきたいだけ）
<br>
<br>

ちょっと前に仕事で latent 層を扱う機会があった。
<br>
<br>

生成AIではない分野だったので、今回扱う「画像生成」とは少し違うけれど、latent 層の考え方自体は共通している。
<br>
なので、latentを軸に、「画像へのノイズ」について考えてみようと思う。
<br>
<br>


まず、前提として、latentとはなんだ？という話。<br>
でもこれは、難しく考えずに、今回は「 **AIが対象物を理解するために作った内部表現（という名の特徴メモ）** 」だと思ってくれれば良い。
<br>
<br>


人間が絵を見るときは、「これは髪の毛」「これは目」「ここが顔の輪郭」といったように、絵と意味を関連づけて見ると思うし、描くと思う。
<br>
一方で、AIモデルは画像をそのまま人間のように理解しているわけではない。
<br>
画像を数値のかたまりとして受け取り、そこから線、色、形、配置、質感のような特徴を圧縮した内部表現（これがlatent）に変換して扱っている。
<br>
<br>

ということは **もし単純なザラつきノイズが本当に生成AI対策として有効なら、この内部メモである latent も大きく変わるはず** という仮定が成立するのでは？と考えた。
<br>
逆に、見た目にはノイズが増えていても内部メモがほとんど変わらないなら、少なくとも「ペイントソフトでノイズを乗せるだけ」で十分な対策になるとは言いにくいと示唆できる。
<br>
<br>

というわけで、今回は、簡単な実験で確認してみようと思う。
<br>
<br>



## 2. 実験にあたって前準備

今回は、小さな AutoEncoder を使って検証します。
<br>

このAutoEncoder は簡単に言うと、
> - 画像を encoder で latent に圧縮し、
> - decoder で元画像に戻すモデル
というもの。
<br>

これは、「検証用のナニカ」程度に思っておいてください。
<br>
<br>

今回のモデルでは、画像は 224px にリサイズして、学習回数は epoch=300 に固定。<br>
epoch は、「学習データを何周AIに見せるか」という意味。<br>
たくさん見せれば良いとか、少ないと悪いとかではなくて、どのくらい見せることで、モデルが「これ以上学習しても意味なくなるか」を見極める回数のこと。<br>
本来なら、何度か色々回してみて、そのモデルに最適な epoch 数を決めたりしますが（loss が横ばいになる周回を見極めながら決定するの）。<br>
今回は 300 でいく。<br>

```python
class SimpleAutoEncoder(nn.Module):
    def __init__(self):
        super().__init__()

        self.encoder = nn.Sequential(
            nn.Conv2d(3, 16, 3, stride=2, padding=1),
            nn.ReLU(),
            nn.Conv2d(16, 32, 3, stride=2, padding=1),
            nn.ReLU(),
        )

        self.decoder = nn.Sequential(
            nn.ConvTranspose2d(32, 16, 4, stride=2, padding=1),
            nn.ReLU(),
            nn.ConvTranspose2d(16, 3, 4, stride=2, padding=1),
            nn.Sigmoid(),
        )

    def encode(self, x):
        return self.encoder(x)

    def forward(self, x):
        latent = self.encoder(x)
        reconstructed = self.decoder(latent)
        return reconstructed
    
```
活性化関数に ReLU 使いますが、LeakyReLU でも良いです。<br>
今回は「単純ノイズで latent が大きく動くか」を見るのが目的なので、ここにこだわりはナシ。
<br>
<br>

### 🟣 latent の「似てる度」を見る

今回は、「画像そのものを目視比較」ではなくて、encoder が作った latent 同士の比較が目的。<br>
そのために、latent を取り出す関数と、latent 同士の近さを見る関数を用意したよ。<br>
<br>
次に出る疑問が、「モデルの評価指標はなにでするの？」ですよね。<br>
今回は、下記の関数でいきます。<br>

```python
model = SimpleAutoEncoder().to(DEVICE)
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
loss_fn = nn.MSELoss()

def get_latent(model, img):
    model.eval()
    with torch.no_grad():
        latent = model.encode(img)
    return latent.cpu().numpy().flatten()


def calc_latent_r2(model, img_a, img_b):
    z_a = get_latent(model, img_a)
    z_b = get_latent(model, img_b)
    return r2_score(z_a, z_b)
```

<br>
これで、元画像とノイズ加工後画像の「似てる度」スコア化の準備が整った。<br>

**r2_score** というのが、今回の実験における「似てる度スコア」ということです。

厳密には「類似度」そのものではないのだけれど、「基準のばらつきを、比較対象がどれくらい説明できるか」みたいなことを見たいので、簡易チェックには使えるかな、と。<br>
そもそも、latent の比較に絶対的な指標があるわけじゃなくて、その時々の目的によって使い分けるらしいのですよ。<br>
（この間お仕事で使ったときには、全く別の評価指標を使用したし）
<br>
<br>

今回の目的は
**「元画像の latent とノイズ付与画像の latent がどのくらい近い値を取るか」＝「どのくらいズレているか」**
なので、r2 が妥当かな？と思って採用しました。
<br>
<br>

うまく r2 を説明できないから、chatGPTに「分かりやすく説明して」と投げて、返ってきた答えが下記。

```
r² は本来「似てる度スコア」そのものではありません。
sklearn の `r2_score` は下記で表されます。
R² = 1 - Σ(y_true - y_pred)² / Σ(y_true - 平均(y_true))²

今回でいうと、
y_true = 元画像の latent
y_pred = ノイズ画像の latent
です。

すごくざっくり言うと、今回のケースでは
「元画像の latent に対して、ノイズ画像の latent がどれくらい近い値を保っているか」
を見るための値です。

もう少し正確に言うなら、
「元画像 latent のばらつきに対して、ノイズ画像 latent との差分がどれくらい小さいか」
を見ています。

1.0 に近いほど、元画像とノイズ画像の latent はかなり近い。
0 に近いほど、元画像の平均値だけを見ているのとあまり変わらない。
マイナスになると、元画像の平均値を使った方がまだマシ、という状態です。
```

<br>
・・・と言っていました。
<br>
<br>
<br>


**分かりやすく・・・とは・・・？笑**
<br>
<br>


まぁ、つまり、結果の目安は、こんな感じと思ってくれて良いですね。

> - 1.0000 に近い：AIの内部表現としてかなり近い
> - 0 に近づく：似ているとは言いにくくなる
> - マイナス：かなり違う内部表現になっている可能性が高い


<br>
それでは、次からは実際に検証ターン。
<br>
<br>
<br>

## 3. モデルの検証

まずは、この比較方法がちゃんと動くか確認してみましょう。

同じ画像を、さっき作った def calc_latent_r2() を使用して latent を比較しましょう。<br>
こんな感じ。

```
base = load_image("./ankho.png", IMG_SIZE)
latent_r2_base = calc_latent_r2(model, base, base)
```

<br>
同じ画像なので、r2 が 1.0000 に近くなるのが理想ですね。<br>
結果はこちら。

```
base vs base latent r² = 1.0000
```

うん。モデルと比較関数は成立してそう。<br>

- 同じ画像を同じものとして見る
- latent の取り出しと比較が正常に動いている

という前提はクリアできてるのではないかな。
<br>
<br>

ということで、準備オーケー。
<br>
<br>

## 4. 学習と画像比較の実験

まずは、画像を準備します。<br>
今回は自分で描いた単純な絵の「ちょうちんあんこう」を使います。<br>
紙に描いたのスキャンして、AIに色つけてもらいました。<br>
<br>
![](/images/ankho.png)
<br>

これを元絵として、次に、ノイズレベルを段階的に付与したノイズ画像を準備します。<br>
ペイントソフト持ってないので、pythonで作ります。雑ですみません。<br>
これを、strength : 10, 20, 40, 60, 80 で回して5枚のノイズ画像完成です。<br>

```python
def add_texture_noise(
    input_path,
    output_path,
    strength,
):
    image = Image.open(input_path).convert("RGB")
    arr = np.array(image).astype(np.int16)

    noise = np.random.normal(loc=0, scale=strength, size=arr.shape)

    noisy = arr + noise
    noisy = np.clip(noisy, 0, 255).astype(np.uint8)

    Image.fromarray(noisy).save(output_path)
```

これでノイズレベル　10, 20, 40, 60, 80 の「ちょうちんあんこう」が出来上がりました。<br>
こんな感じです。<br>
![](/images/aankho_noise_10.png) ![](/images/ankho_noise_20.png) ![](/images/ankho_noise_40.png) ![](/images/ankho_noise_60.png) ![](/images/ankho_noise_80.png)

<br>
<br>
やりたいことは、「各ノイズレベルの画像に対して、学習済みモデルが元画像とどれくらい近い latent を出すかを見る」こと。<br>
AIのモデルを使用するためには、学習用の画像が必要です。<br>
手元に、全くタッチが違う画像を5枚ほど用意しました。<br>
<br>
学習には、自作イラスト1枚（元画像と同じタッチ）、写真1枚、イラスト3枚を使用します。<br>
（今回は再現性を厳密に担保する実験ではなく、「単純なノイズで latent が大きく変わるのか」を見るための実験なので、学習画像そのものの掲載は割愛！）<br>
友人のイラストレーターから借りたものも入れたので、著作権もありますので、ご了承ください。<br>
<br>
<br>


それでは参りましょう。
<br>
<br>

学習部分は通常の AutoEncoder と同じです。<br>
入力画像を encoder → decoder に通し、復元画像と元画像の差が小さくなるように学習。<br>
詳細コードは省きますが、optimizer は Adam と loss 測定には平均二乗誤差を使用です。<br>
```
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
loss_fn = nn.MSELoss()
```

<br>
これで回します。<br>

```python
base = load_image("./check_images/ankho.png", IMG_SIZE)
for s in [10, 20, 40, 60, 80]:
    noise = load_image(f"./check_images/ankho_noise_{s}.png", IMG_SIZE)

    latent_r2_base = calc_latent_r2(model, base, base)
    latent_r2_noise = calc_latent_r2(model, base, noise)
    recon_r2_base = calc_reconstruction_r2(model, base)
```

結果。<br>

```
base vs noise_10 latent r² = 0.9998
base vs noise_20 latent r² = 0.9996
base vs noise_40 latent r² = 0.9981
base vs noise_60 latent r² = 0.9938
base vs noise_80 latent r² = 0.9813
base reconstruction r² = 0.8842
```

<br>
<br>
こ、これは・・・・・
<br>
<br>

noise_80 まで強くしても latent r2 は 0.9813 で、元画像の latent とかなり近い値ですね。<br>
一方、reconstruction r² は 0.8842 でした。<br>
<br>
<br>

一回だけだとつまらないので、学習画像を入れ替えてもう一度やってみましょ。<br>
学習画像は元画像の「ちょうちんあんこう」含めて6枚です。<br>
条件を変えるので、model と optimizer も作り直しますね。同じ model を使い回すと、前回の学習結果が残ってしまうので。<br>
<br>
<br>

そんな条件下においての実験結果は・・・
<br>

```
base vs noise_10 latent r² = 0.9999
base vs noise_20 latent r² = 0.9997
base vs noise_40 latent r² = 0.9985
base vs noise_60 latent r² = 0.9952
base vs noise_80 latent r² = 0.9873
base reconstruction r² = 0.9487
```

<br>
おおお・・・。
<br>

reconstruction r² = 0.9487 に変動しましたね！！高っ！
<br>

これは、小型 AutoEncoder の復元性能が学習画像の構成に影響されたからかな？<br>
（復元についての仮説などは、今回は目的としないので深掘りやめておきます）<br>
<br>

ただ、元画像とノイズ画像の latent r² は、どの条件でも 0.98 以上を保っているんですよね。<br>
見た目にはかなりザラついているのに、モデル内部の圧縮情報としては、かなり元画像に近いまま。<br>
<br>

ということで、今回の小型 AutoEncoder 実験で示唆できることは、<br>
**『単純なランダムノイズが latent 表現を大きく崩す傾向が見られない』**
<br>
ということかと思います。
<br>
<br>

やっぱり単純なテクスチャ系のノイズと Glaze の敵対的摂動は別物だな、という印象。
<br>
<br>
<br>

## 5. さいごに：ノイズより前に考えたいこと
ここまで実験モードだったので、さいごに少し、生成AIの学習について。<br>
<br>

そもそも、漫画家さん、小説家さん、画家さん、声優さん、絵師さん、他にもたくさんのクリエイターの方々の、いわゆるゼロから作品を創り出す方たちって、「学習素材にされたくない」ということが大きいですよね？<br>
だから本来の目的は、「ノイズを入れて学習しづらくする」ではなくて、「**そもそも無断で学習素材として扱われにくくする**」ことだと思うんです。
<br>
<br>

・・・という考えに思い至ると、仕事柄、「そもそも学習素材にさせない仕組みを作れば良いのでは？」とか考えてしまいます。悪い癖。<br>
まぁ、そう難しくなくできると思うんですよね。<br>
要は、「学習しても旨みのないもの」にすれば良いだけだから。<br>
研究所や論文レベルの学習素材の阻害をしよう！ではなくて、大量に雑なスクレイピングから作品を守ろう！が目的のはずですから。<br>
<br>

ただ、ここで言っておかなければならないことは、「AIそのものが悪い」ということではないということです。<br>
そうではなくて、
**技術には「使いどころ」がある**という話です。<br>
ぷに蔵の仕事とAIは相性が良すぎて、アイディアの壁打ちにも最適だし、日々依存しています。<br>
<br>

だけど、創作物や声のように、作り手の身体性・人格・文脈と深く結びついたものを、作り手の意思と関係なく素材化することは、<br>
どうしても「使いどころが違う」と思ってしまうんです。<br>
<br>
<br>

先日、声優さんたちが、法務省の「<a href="https://www.moj.go.jp/MINJI/minji07_00404.html">肖像、声等の無断利用による民事責任の在り方に関する検討会第２回（令和８年５月２８日）</a>」で、要望書を提出していましたよね。<br>
（実はちょっと前に、声の波形処理で生成AIの演技抽出の阻害ができないか考えたことありました。結構難しいですね）<br>
実際に、「声」に対しての境界線は、すごく難しいと思います。<br>
<br>

ただ、個人的には、これらの要望が現場の声優さんたちから提出されたことは、「おいおい」状態で・・・。<br>
本来なら、当事者がここまで訴えるよりも前に、制度側を整備しておく話でしょ？と思っています。<br>
<br>

この話を掘り下げていくと、かなり長くなるので止めますけど、日本は制度化が遅いうえに、法律にすらせず「お願い」状態で現状維持とかするので、もっと法整備を早めてほしいです。<br>
<br>
<br>

生成AIを止めろとは思いません。<br>
<br>
<br>
<br>
ただ、誰かが「素材」と呼んでいるその作品たち一つ一つには、必ずゼロから創造した誰かがいるという事実だけは、絶対に忘れたくないですね。<br>
<br>
<br>

ということで、「画像に載せた「ノイズ」は、生成AI対策として有効なのか？」の検証でした。
<br>
