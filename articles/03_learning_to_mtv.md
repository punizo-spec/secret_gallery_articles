---
title: "第３章　Hello World それはすべての始まり"
emoji: "♌️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["django", "python"]
published: true # trueを指定する
published_at: 2025-09-02 08:12
---

## 01. ロケット🚀から Hello World 🌎へ（レガシーグリーティングに敬意を込めて）
前章までで、sg_user 周りの設定が完了した。
長くて、わりと退屈で、意味の分からない作業も多かったでしょう？
ここからは、ブラウザ操作が入る、直感的に変化が確認できるフェーズに入っていくよ！

まず、今回やりたいことの確認から。
いま現在 、ローカルサーバーを起動させて [http://127.0.0.1:8000/](http://127.0.0.1:8000/) にアクセスすると、ロケットの打上げ画面になっていると思う。

この画面を変えて、ブラウザ上に「Hello World 🌏」を表示させることが目標！

というわけで、さっそくアプリを作成しよう！
前章では管理ユーザーの機能を実装するために「 sg_user 」を作成したよね。

１アプリ１機能！

今回は、作品を展示するためのアプリだから、新しい「 sg_pieces 」アプリを作るよ。

アプリ開始のコマンドは覚えている？
startapp を使うよ。
（もしこの時点で、runserver を起動させていた場合は ctrl + c でサーバーを停止させてからコマンドを入力してね）

```bash
python3 manage.py startapp sg_pieces
```

新しいアプリは作成された？
VSCode 上で、こうなっていれば成功！

![](/images/c3_p1_1_newapp.png)

ではでは。
新しいアプリを作ったとき、最初にやらないといけないこと、覚えてる？

Django に、「このアプリを作ったから覚えてね」という、**教えてあげる設定**が必要だったね。
ということで、setting.py の INSTALLED_APPS に、いま作ったアプリを追加しよう！

```python
INSTALLED_APPS = [
    # Djangoの標準機能
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    # 自分が作ったアプリ
    "sg_user",
    "sg_pieces",
]
```

これでOK！
Django は sg_pieces アプリを認識してくれるよ！


アプリの準備ができたので、本確的にアプリの中身をカスタムしていこう！！
まずは、アプリ内に「 templates 」ディレクトリを作って、さらにその中に「 sg_pieces 」ディレクトリを作成する。

ここで秘技をお教えしよう・・・。templates/sg_pieces ２つのディレクトリを一気に作成する小技だ。

まずは、ターミナルのコマンドラインで、アプリの中に入る。
```bash
cd sg_pieces
```

ここからだ！打て！！
```bash
mkdir -p templates/sg_pieces
```

![](/images/c3_p1_2_newdir.png)

２つのディレクトリが一気に出来上がってない？？

-p オプション使うと、複数のディレクトリをまとめて作成してくれるから、とっても便利！
ぷに蔵、はじめてこれ使ったとき、感動したこと未だに覚えてる笑


ディレクトリが無事にできたので、templates/sg_pieces の中に、画面に表示させるための index.html ファイルを作るよ〜。
拡張子も大事だからね。ちゃんと .html で作ってね！

index.html を作成したら、このコードを貼り付けて。
（最小サンプルなので、DOCTYPE や head タグは省略）

```html
<!-- sg_pieces/templates/index.html -->
<html>
  <body>
    <h1>Hello World 🌏</h1>
  </body>
</html>
```

> 今回は Django の記事なので、html や css の説明はしないです。
> html と css はホームページを作成するときのコンテンツ構造やデザインを作成するための言語だから、覚えておいて損はない！（というか、マスターまではいかなくても良いけど、少し理解しておかないとつらいかも🧐）
> 触ったことない人は、ぜひ挑戦してみて！


それでは index.html が出来上がったので、今度はこのファイルをブラウザに表示させるための設定をしていこう！

必要なことは……
1. sg_pieces アプリ内の views.py に、index.html を表示させるための python コードを書く
2. sg_pieces アプリ内に urls.py ファイルを作成して、sg_pieces 内のファイルを表示させるためのルーティングをする
3. 最後に、プロジェクトのurls.py に、どのアプリの urls.py を読み込むかを設定する

Webページに html ファイルを表示させるためには、上記３ステップが必要だ！
さあ、いこう！！

**1. sg_pieces の中にある、views.py に、下記コードを入力して。**

```python
# render はテンプレートを指定して、ブラウザに表示させる関数
from django.shortcuts import render

def index(request):
    return render(request, 'sg_pieces/index.html')
```

**2. 次に、sg_pieces の中に、urls.py というファイルを作成する。**

![](/images/c3_p1_3_make.png =590x)
*VSCode では、黄色枠の左側クリックでファイル、右側クリックでフォルダが作成できる。*

作成した urls.py ファイル内に下記コードを入力する。
```python
# Django の path() は、どのURLにアクセスしたら、どの関数（views）を実行するのかの設定時に使用する
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

**3. secret_gallery プロジェクト内の urls.py を以下のように書きかえる。**
（最初から存在しているファイルだよ）

書きかえは２箇所。グレー背景のコードを組み込むよ。
> - from django.urls import path`, include`
> - `path('', include('sg_pieces.urls')),`

書きかえというか、ちょっとの追加だね！

```python
# secret_gallery/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('sg_pieces.urls')),
]
```
※ ファイル内の冒頭にたくさんコメント書いてあるけど、不要であれば削除して良いものだよ。
ファイルの用途について説明してくれているコメントなので、機能には一切関係なし。

ここまでいけたら、ローカルサーバー起動！！

・・・したいところだけど、さっき cd sg_pieces に移動しているから、下記コマンドで１つ階層上がってから。
```bash
cd ../
```

manage.py ファイルがある階層にいかないと、下記コマンドが効かないのでね。
```bash
python3 manage.py runserver
```

そして [http://127.0.0.1:8000/](http://127.0.0.1:8000/) をブラウザで読み込むと・・・

![](/images/c3_p1_4_helloworld.png =590x)
*これはズームアップしております*

どうかな！？表示されたかな！？

表示されたら、大成功ーーー🥳<br>


ここで、いま何をしたのか整理しておこう！

まず最初に、index.html ファイルを作成した。
このとき、sg_pieces アプリ内に templates/sg_pieces というディレクトリを作成したね。
ファイルの作成自体は難しくなかったと思う。

・・・でも・・・これ、なんかちょっと不思議な構造じゃない？
だって sg_pieces アプリ内に templates フォルダを作って、さらにその中に、sg_pieces という**アプリと同名のフォルダ**を作成したんだよ。
なんでこんなことするの？って思うよね。


実は Django では、複数アプリが存在するプロジェクトでも、それぞれのアプリごとにテンプレート（HTMLファイルのことを Django ではこう呼ぶ）が持てるようになっている。
そして「 **アプリ/templates/アプリ名/xxx.html** 」みたいな構造にしておくと、Django が自動でテンプレートファイル（.html）を見つけてくれる仕組みになっているのよ。


::::details 仕組みについての詳細説明はこちら
各アプリ内に `templates` というディレクトリがあれば、Django はその中を探しに自ら行ってくれるけど、前提として、settings.py にその設定がされていないとだめ！
settings.py の TEMPLATES の中を見ると、こんな感じの設定があるから、自分のファイルで探してみて？

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        ...
    },
]
```

*`「"APP_DIRS": True」`のとき、Django は「アプリ/templates/」内に、テンプレートファイルを自動で探しに行ってくれる*
<br>

「**でも、それなら templates の中に、アプリと同名のフォルダ作る意味なくない**」
と思ったあなた！鋭すぎる！！<br>
そうなの。別に、アプリと同名のフォルダって、作成必須のフォルダではないの！
作らなくても、構造がしっかり管理されていれば、ぶっちゃけ動く。<br>
じゃあ、なんで作るんだろうね？<br>
たとえばさ、テンプレートファイルって、結構似た名前になることがあったりするのよ。
top.html とか detail.html みたいに、そのWebページの機能をファイル名にしてしまうと、似通った名前になりがち。<br>
そして、 **Django では、アプリごとにテンプレートを持てる** ってさっき言ったじゃない？
そうすると、アプリごとに、似たような名前のテンプレートファイル・・・むしろ、全く同じファイル名で作りたくなる可能性が、ゼロとはいえないのよ。<br>
そうなったときでも、templates/アプリ名/top.html とかにしておけば、どこのアプリの top.html なのか、Django がすぐに判別できて、表示上やファイル参照における干渉が起こりづらくなるの。<br>
あと、もうひとつ説明させてね。
さっき設定した views.py の中で、render(request, 'sg_pieces/index.html') って書いたの、覚えてるかな？<br>
このルーティングを、なぜ「'index.html'」ではなく「'sg_pieces/index.html'」にしないといけなかったかというと・・・**Django は templates ディレクトリまでしか探さない**から。
templates よりも階層をもぐってテンプレートファイルを探すことをしないから、templates 以降の階層については、ちゃんと明記してあげる必要がある。<br>
ちょっと特殊な例だけど、**templates 内で HTMLテンプレートファイルをさらにアプリ名と同じファイルでラッピングすることで、事故を減らせる**って覚えてね。<br>
この運用は、実践でも使ってるよ！

::::

index.html を templates/sg_pieces に設置した後は、index.html を表示させるためのコードを sg_pieces/views.py に書いたね。
ということは、views.py は HTML テンプレートファイルを表示させるための役割を担っていたんだ！！

そして最後に、urls.py（プロジェクトとアプリ両方）にルーティング設定した。
個々のファイルをつなげる役目を担っているのだよ、urls.py は。

> プロジェクトの urls.py は、各アプリの urls.py というファイルの存在を Django に教えてあげる役割
> 各アプリの urls.py は、アプリ内のファイル同士をつなげる役割
> プロジェクトとアプリの urls.py は、ちょっとだけ役割は違うけど、**ルーティングをする**という機能は同じだね

ここまでの説明を読んでみて、「よく分からない・・・😩」ってなっているかな？
ても大丈夫！
まだまだ触り始めたばっかり。理解することよりも、手を動かすことを優先しよ！！

ぷに蔵も、最初なんか何も理解しないでコピペ祭りしてた！
触りかた覚えてきたのなんて、20くらいプロジェクト作ってからだと思う（学びが遅いｗ）

いまは、**Djangoではテンプレートはこういうフォルダ構成にする**って覚えちゃってね。

表示された！やったーー！！って思ってくれれば、それだけで十分だよ⭐️


さあ！次に進もう！！


## 02. コピペで３分。「こんにちは、〇〇さん」
今回は、前回と似た作業の軽量版。
やりたいことの完成系は、ブラウザに「こんにちは、〇〇さん」て表示させること。
`〇〇さん` は自分の名前でもなんでもいいよ！


作業の手順としては、1~5 の流れでいこう。新しいファイル名は greeting.html にしよう！
> 1. まずは、templates/sg_pieces/index.html をコピペ。
> 2. ファイル名を greeting.html に変更して、「Hello World 🌏」を「こんにちは、〇〇さん」に変更
> 3. views.py にテンプレートファイルを表示させる関数を追加
> 4. urls.py に views.py に書いた新しい関数をルーティング
> 5. ブラウザに表示

一気にいこう！
コードの上部に書いてある <!-- --> や # ~~~ の箇所は対象ファイルを示しているよ。

```html
<!-- sg_pieces/templates/greeting.html -->
<html>
  <body>
    <h1>こんにちは、ぷに蔵さん</h1>
  </body>
</html>
```

```python
# sg_pieces/views.py
from django.shortcuts import render

def index(request):
    return render(request, 'index.html')

def greeting(request):
    return render(request, 'greeting.html')
```

```python
# sg_pieces/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('aisatsu/', views.greeting),
]
```

[http://127.0.0.1:8000/aisatsu/](http://127.0.0.1:8000/aisatsu/) をブラウザで読み込むと・・・

![](/images/c3_p2_5_greeting.png =590x)
*これはズームアップしております*

どうでしょうね？？？できたかな！？

今回、新しいところは urls.py の path('`aisatsu/`', views.greeting), のところだね。
これは、**views.py の def greeting() 関数を、'aisatsu/' にアクセスしたときに実行**という意味。


urls.py にこの path() を加えれば、[http://127.0.0.1:8000/aisatsu/](http://127.0.0.1:8000/aisatsu/)へのリンクが爆誕💥✨💥


::::details ちょこっと解説：URLの末尾の /（スラッシュ）はルーティングのキモ
urls.py に「path('aisatsu/', views.greeting)」と設定したら、
「http://127.0.0.1:8000/aisatsu/ にアクセスしたときは greeting() 関数を呼ぶ」という意味なんだけど、末尾の「/」はなぜ書くのだろう？<br>
試しに、urls.py の'aisatsu/'部分末尾スラッシュを抜いた `path('aisatsu', views.greeting)`に変えて、[http://127.0.0.1:8000/aisatsu/](http://127.0.0.1:8000/aisatsu/) にアクセスしてみて。<br>
Page not found (404)と表示されたね！？<br>
でも[http://127.0.0.1:8000/aisatsu](http://127.0.0.1:8000/aisatsu)でアクセスすれば、ちゃんと表示される。
URL が一致してるから、それはそう。<br>
では、次。
urls.py に末尾スラッシュをつけた状態に戻して、[http://127.0.0.1:8000/aisatsu](http://127.0.0.1:8000/aisatsu)にアクセスしてみて。<br>
今度は「こんにちは、〇〇さん」が表示されたんじゃないかな？<br>
え？でも、今度は、
ルーティング：末尾スラッシュあり
ブラウザURL：末尾スラッシュなし
だから、URL 不一致になるのに、おかしいよね！？大混乱だよ！！<br>
でもこれ、実は Django の仕様。
仕様というか、Django の基本ルールとして
**末尾に / （スラッシュ）がついている URL を正式なルートとする** というものがある。
だから、もしも末尾スラッシュなしアクセスされたら、**勝手に / つきのURLにリダイレクトする**動きをしているの。<br>
「ふーん。便利じゃん」<br>
そう思うよね笑<br>
でもこれ、１ページを表示するたびに、Django が「末尾スラッシュ有無を判定 → / ありの URL にリダイレクト」というステップを踏んでいる。
つまり、URL の設定ひとつで減らせる無駄な作業を Django にやらせている。<br>
趣味のホームページで、ルート数も10そこそこくらいなら、何も問題ないよ。
でもこれが、企業ホームページやECサイトのような、とても大掛かりテンプレートを扱うようなレベルになったときは、かなり余計な負荷を強いることになってしまう。<br>
これは正しいルーティングをするだけで減らせる負荷だから、最初のうちから正しい書き方を意識するクセをつけておこう！！！
::::


## 03. つなげ！「Hello World」 → 「こんにちは、〇〇さん」
それぞれのテンプレートファイルの作成が完了し、いまはそれぞれをブラウザに表示できているだろうか？
> index.html → Hello World 🌏
> greeting.html → こんにちは、〇〇さん

でも、いまのままだとそれぞれのページが独立している。
Webアプリは、つながってこそナンボ！

つなげていこうぜ、彼らをよ・・・（キャラ崩壊）

というわけで、今回は
「Hello World 🌏」をクリックすると「こんにちは、〇〇さん」ページに飛ぶ🚀

そんな“ページ間リンク”を実装してみよう！

使うものは、以下のファイル
- index.html
- アプリ内の urls.py

それでは、index.html の加工からいこうか。
HTML ではおなじみの <a> タグ（リンク）を使って、文字にリンクを埋め込むよ！
```html
<!-- sg_pieces/templates/index.html -->
<html>
  <body>
   <h1><a href='aisatsu/'>Hello World 🌏</a></h1>
  </body>
</html>
```

この時点で、index.html のページを再読み込みしてみて！
リンクが完成していて、「こんにちは、〇〇さん」ページに飛んでいける！！

これで完成っ🎁


・・・・・でも良いんだけど、Django はここからなんだ！！


ここから、アプリ内の urls.py をちょっとだけ加工するよ。
```python
# sg_pieces/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('aisatsu/', views.greeting, name='greeting'),  # ここに追加したよ
]
```
name='greeting'　← これは一体なんだろう？？<br>

これは「リンクの名前」です。
Django では、**urls.py のルートそれぞれに名前（name）」をつけておくことができるの**。

なぜそんなことをするのか、って？

それはね・・・テンプレートファイルに URL を書くときでも、URL の直書きをしなくてよくなるから！！

たとえば、 いま href="aisatsu/" って書いたじゃない。
でもこれ、もしも URL 構造が変わったら全部書き直しになっちゃうでしょ？
**端的に言って、地獄☠️**


でもここに、リンクの「名前」が設定されていて、それを不変なものとして位置付けていたらどうだろう？
別にリンク URL が変わろうが、「名前」が変わらない限り、テンプレートファイルの書き直しは発生しない！

**え？天国から地獄だね😇**

この「名前」の使い方はとっても簡単。
index.html のhref を"{% url 'greeting' %}"に書きかえるだけ！
```html
<!-- sg_pieces/templates/index.html -->
<html>
  <body>
   <h1><a href="{% url 'greeting' %}">Hello World 🌏</a></h1>
  </body>
</html>
```

これだけで良いんだ！！！

これで、後から「**URL も'greeting/'に変更しちゃおうかな〜**」と思い立ったとしても、name='' が変わらなければ、テンプレートの修正は一切ナシ！！

この {% url 'ルート名' %} 構文は、HTMLテンプレートファイルで使える Django の独自機能。
とても便利だから、覚えておこう！！


## 04. 作品を展示するための大・前・提は models.py にあり！
これまでは、おそらくチュートリアル。
この記事を読んで一緒にやってくれたあなた！お疲れさま！！

ここから、アプリを作っていくことになるよ。


第３章の途中から・・・中途半端ｗ


まぁ、Django の基本的な動きが分かったところで、さっそく取り掛かろうかな。


いざ、開幕！！**秘密のプライベートギャラリー**


まずは、ギャラリーに展示するためには、展示する作品の箱を作らないといけない。

箱＝データベースの設定

第２章あたりで、
**models.py に「テーブルの設定詳細（モデル定義）」を書くと、Django がその定義どおりにデータベースにテーブルを作ってくれる**
って言ったの覚えてるかな？

sg_user の作成で models.py は一度使ったけど・・・でも正直、管理ユーザーの作成は特殊な設定の部類なんだよね（ぷに蔵的な感覚では）。
だから、今回が初めてくらいの気持ちで、アプリにおける models.py の使い方に入っていこ！

まずは、ギャラリーに必要な情報は何にしようか？
定義としては、こんな感じかなぁ🧐
- タイトル（その名の通り作品タイトル）
- 作品登録日（DB に登録した日）
- メモ（自由記述。空欄OK）

ひとまず、このくらいで。
もっと増やしたい人は、項目増やしてもいいよ！

上記の内容で、models.py に書いていこう！
```python
# sg_pieces/models.py
from django.db import models

class GalleryPiece(models.Model):
    name = models.CharField(max_length=100)  
    created_at = models.DateTimeField(null=True, blank=True)
    memo = models.TextField()

    class Meta:
        verbose_name = "ギャラリー作品"
        verbose_name_plural = "ギャラリー作品一覧"
        db_table = "gallery_piece"  
```

> **設定したフィールドごとに、どんな設定をしたのか確認してみよう。**
> `name` : 文字列の入力。未入力はエラーになる。
> `created_ad` : 日時の選択入力フィールド。空欄, null OK
> `memo` : 長文テキスト入力用フィールド

なんとなく、モデル定義の書き方は分かるかな？
モデルで使うフィールドは、[Django 公式で紹介している](https://docs.djangoproject.com/ja/5.2/ref/models/fields/)けど、いまは文字列や日付など、一般的なフィールド型を使ってモデル定義を書いていくよ！

モデルのクラス内で、異質なエリアがあったよね。

```python
class Meta:
    verbose_name = "ギャラリー作品"
    verbose_name_plural = "ギャラリー作品一覧"
    db_table = "gallery_piece"  
```
このエリア。

class Meta は、モデルの振る舞いや見た目のカスタマイズをしたいときに使うもの。
モデル自体は、DB テーブルのフィールドに関する定義を書いているでしょ。
フィールドのデータ型（CharField, DateField, DateTimeField ...etc）とか、デフォルト値とか、空白可否とか。
class Meta は、それらフィールドが入る箱（テーブル）を、「管理画面上の表示名」や「DB 上のテーブル名」を指定する箇所。
Djangoはこの class Meta という名前のついたエリアには特別な意味を持たせていて、**“モデルの設定を書くための特別エリア”**として自動で認識してくれるのよ。

ちなみに、管理画面上の表示などは、直接 DB に影響を与えない（＝管理画面だけに影響）。マイグレート不要。
> - `verbose_name` : 管理画面表示用
> - `verbose_name_plural` : 管理画面表示用

だけど、DB 上のテーブル名の表示は、DB に影響尾を与える（＝DB スキーマに影響）。変更した場合は、その都度マイグレート必須！
> - `db_table` : DB に反映したときのテーブル名を指定。未設定の場合は、アプリ名＋クラス名のテーブルが出来上がる。今回なら sg_pieces_gallerypiece テーブル


これは、いま一生懸命覚えなくて大丈夫なところ。
将来的に自分でプロジェクトを作るとき、「ここをこうしたいなぁ・・・」と思ったときに調べたら、「class Meta で設定すればいいんだ！」って場面が出てくるかも？！


## 05. DB に命を吹き込め！！makemigrations と migrate はセット商品です（大事なので２回言う）
モデル定義の設定が完了した！

・・・とうことは、今度はそれを DB に反映させるんだ！
sg_user で管理ユーザーのモデル定義のときに一度やった作業だが、覚えているかい？
覚えていなくても大丈夫！
大事なことなので、２回言うスタイルでいく！！（ただ前章のネタを再掲しただけとも言う）


まずは、コマンドラインに以下を打ち込め！
（コマンド入力はローカルサーバーを ctrl + c で停止させてから）
```bash
python3 manage.py makemigrations
```
- `makemigrations` : モデル定義（＝テーブル構造）を DB に反映させるため、保存用ファイルに変換する
「こういう形で保存しますね」という準備作業

```bash
python3 manage.py migrate
```
- `migrate` : makemigrations で保存用変換ファイルが作成されたので「DB に反映させます」という実行コマンド

> 🟢 **ファイル作成して → 実行** 🟠

> 🟢 **makemigrations → migrate** 🟠

ここで、タイトル回収！！（このノリも２回目…暑苦しいｗ）

> 🟢 **makemigrations と migrate はセット商品です**🟠

:::message
**エラーが出たりしていないかな？**
モデル定義の記入ミスはしていない？
仮想環境には入っている？（コマンドラインの先頭に(venv)が付いてないとエラー不可避！）
コマンドのタイポはないかな？
:::

エラーが出ずに、コマンドラインがこんな表示になっていれば、DB に反映完了！
![](/images/c3_p5_6_migrate.png)

でもこれだと、DB に反映された実感がないよね・・・。

それなら、DB の中身を見てみよう！！
拡張機能をインストールすれば、VSCode 上で見られるんだ！！
まずは、拡張機能のアイコンクリックから。

1. **左の縦長にアイコンが並ぶ「アクティビティバー」から、拡張機能アイコンをクリック**

2. **「sqlite」と検索**
そうすると「 SQLite Viewer 」が出てくるから、これをインストールしよう。
![](/images/c3_p5_8_sqliteviewer.png =590x)


3. **SQLite Viewer を入れ終わった後に、プロジェクトルート階層（manage.py がある階層のこと）にある「 db.sqlite3 」というファイルをクリック**
![](/images/c3_p5_9_sqliteview.png)
*この記事だと、小さくて見づらいけど、自分の画面だと分かるんじゃないかな？*

設定した、name, created_at, memo が反映されているかな？


おやや？でもなんか、「id 」という、設定していないフィールドがあるぞ？

これは「主キー」と呼ばれるもので、このデータの**一意の識別子になるもの**。
つまり、他のデータと混同させない（＝重複させない）ための id 管理ということだね。

フィールド定義の中に、自分で主キーを設定することも出来るんだけど、今回はそれをしなかったのよね。
だから「 id 」いうフィールドが自動で追加されて、データの一意性を保ってくれているという状態になったの。
「あー、データの管理番号が勝手に付いてくれたんだね」くらいに思っておいてもらえれば、感覚としては正しいかな🌻


## 06. admin.py にモデル定義を設定してみよう
それでは、テーブルが作成されたので、ぜひ成果を管理画面で確認してみようじゃないか！
設定するファイルは sg_pieces アプリ内の admin.py だよ。
前の sg_user/admin.py と、少し書き方が変わっている点を確認しながら書いていこう！

今回の admin.py は、こう！
```python
# sg_pieces/admin.py
from django.contrib import admin
from .models import GalleryPiece

@admin.register(GalleryPiece)
class GalleryPieceAdmin(admin.ModelAdmin):
    pass
```
**４ステップでいこう！**
> 1. Django の管理画面機能（admin モジュール）を読み込む
> 2. 管理画面に表示させたい GalleryPieceモデル定義を読み込む
> 3. @admin.register(GalleryPiece) で「このクラスを GalleryPiece に結びつけます」という合図を Django に送る
> 4. GalleryPiece モデルを表示させるためのクラスを設定する


設定が完了したら、ローカルサーバー起動。
```bash
python3 manage.py runserver
```

管理画面にログインして、内容を確認。
[http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/)

青色枠部分が追加されていたら、設定成功！
![](/images/c3_p6_10_admin.png)

ここの画面から、ギャラリー作品の登録ができるぞ！
いまはまだ文字列のみの登録だけど、ここに画像を表示させるエリアがあったら、一気に作品展示っぽくなるよね！？（伏線…ﾌﾌﾌ……）

## 07. ところでデータベースって設定しましたっけ？
無事にデータベースにモデル定義が反映され、管理画面でも確認できるところまでいけたね！

でもさ、これだけ「定義、設定、反映確認、定義、設定、反映確認……」ってやっていたけど、なんか忘れてない？

データベースの設定って、どこかでやったっけ？？？


そうなんです。
やってないんですよ！！！

ということで、settings.py を見てみよう！（プロジェクト内にあるよ）
該当箇所はここ。
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

実はデフォルトで設定されていたんだね、という結論。

でもさ、正直言うと、SQLite って簡易的な感じで使えるけど、実運用には向いていない。
実際の現場でデータベースを使用する場合なら、MySQL や PostgreSQL あたりを使うことが多い（もちろん要件定義次第ではあるけど、とてもメジャー）。

:::message
**データベース → ❌　RDBMS → ⭕️**
正確には、「 SQLite、MySQL、PostgreSQL 」などの名称は、 **RDBMS** （`リレーショナルデータベース管理システム（Relational Database Management System）`）というデータベース管理・操作するためのソフトウェアの名称のこと。**データベースとは、データを入れる枠組みのことでしかない**からね。いまは説明の便宜上、「データベース」と呼んでいるだけ）<br>

データベース → データを入れる箱そのもの
RDBMS → その箱を操作するためのソフト（SQLite や MySQL など）
Django は、settings.py の DATABASES の ENGINE の設定を見て「どのRDBMSを使うか」を選んでいるよ
:::

🧐 サーバーで Webアプリケーションを運用するときには、データベースの切り替えが必要になってきそうだね🌻

## 📕 Django の『 MTV 』という考え方

ここまでの作業で、Django の仕組みを一通り体験してもらったよ！！
無事にここに辿り着いてくれた人はどのくらい居るんだろう・・・
説明が細かすぎたよね笑

ぷに蔵も書いていて、思ったよ。
「長ぇし、細けぇな……」と。

ここで第３章の締めとして、MTVアーキテクチャという、Django の概念に触れてみようと思う。
現時点で、管理画面表示までいっている人にとっては、何も難しいことはない。

> M（Model）: models.py → データベースのテーブル構造を設計
> T（Template）: templates/*.html → 見た目を作成
> V（View）: views.py → Model と Template をつなぐ橋渡し

これを見て、**「あ、全部触ったことあるファイルだな」**と思った人、大正解👍🥳🎊

そうなのよ。いままで設定してもらったり、作成してもらってきたファイルたちの繋がりのことを**MTVアーキテクチャ**と呼んでいる。

ただそれだけ！

この MTV が3つそろって、初めてDjangoが「Webアプリ」として動くという仕組み。

覚えなくても良いんだけど、一応 Django さんの設計思想みたいだから、頭の片隅に置いておいてもらえると、いつか役に立つかも〜〜（……立つか？）。

:::message
 🫛 豆知識  
世の中には「MVCアーキテクチャ」という考え方があるよ（こちら様が、思想の本家）。  

- **M ： Model** → Django では models.py  
- **V ： View** → 見た目（Django では templates/*.html ）  
- **C ： Controller** → 機能全般（Django では views.py ＋ URLconf（urls.py））  

Django も MVC の考え方をベースにしているんだけど、MVC の場合の Controller を Django では **views.py と urls.py に分けて実装** しているのね。
だから Django では「 MTV 」と呼ぶのよ。
:::