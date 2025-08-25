---
title: "第３章　Hello World それはすべての始まり"
emoji: "♌️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["django", "python"]
published: true # trueを指定する
published_at: 2050-08-20 00:00 # 未来の日時を指定する
---

## 01. ロケット🚀から Hello World 🌎へ（レガシーグリーティングに敬意を込めて）
前章までで、sg_user 周りの設定が完了したね！
長くて、わりと退屈で、意味の分からない作業も多かったでしょう？
ここからは、ブラウザ操作が入る、直感的に変化が確認できるフェーズに入っていくよ！

まず、今回やりたいことの確認からね。
いま現在 、ローカルサーバーを起動させて [http://127.0.0.1:8000/](http://127.0.0.1:8000/) にアクセスすると、ロケットの打上げ画面になっていると思う。

この画面を変えて、ブラウザ上に「Hello World 🌏」を表示させてみるよ！

というわけで、さっそくアプリを作成しようね。
前章では管理ユーザーの機能を実装するために「 sg_user 」を作成したね。

１アプリ１機能⭐️

今回は、作品を展示するためのアプリだから「 sg_pieces 」を作るの。
アプリ開始のコマンドは覚えている？
そう！！ startapp を使うんだったね！
（もしこの時点で、runserver を起動させていた場合は ctrl + C でサーバーを停止させてからコマンドを入力してね）
```bash
python3 manage.py startapp sg_pieces
```

どう？新しいアプリは作成された？
VSCode 上で、こうなっていれば大成功！

![](/images/c3_p1_1_newapp.png)

ではでは。
新しいアプリを作ったらやらないといけないこと、覚えてる？
Django に、「このアプリ作ったから覚えてね」という、**教えてあげる設定**が必要だったね。
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
    # 自分が作ったアプリ（今回はログインできるユーザーを管理するアプリ）
    "sg_user",
    "sg_pieces",
]
```

これでOK！
Django は sg_pieces アプリを認識してくれるよ！


では、アプリの準備ができたので、本確的にアプリの中身をカスタムしていこう！！
まずは、アプリ内に「 templates 」ディレクトリを作って、さらにその中に「 sg_pieces 」ディレクトリを作成する。

ここで秘技をお教えしよう・・・。templates/sg_pieces ディレクトリを、一気に作成する小技だ。

まずは、ターミナルのコマンドラインで、アプリの中に入っておいて。
```bash
cd sg_pieces
```

ここからだ！
```bash
mkdir -p templates/sg_pieces
```

![](/images/c3_p1_2_newdir.png)

２つのディレクトリが一気に出来上がってない？？
-p オプション使うと、複数のディレクトリをまとめて作成してくれるからとっても便利！
ぷに蔵、はじめてこれ使ったとき、感動したこと未だに覚えてる笑


さてと、ディレクトリが無事にできたので、templates/sg_pieces の中に、index.html というファイルを作るよ！
拡張子も大事だからね。ちゃんと .html で作ってね！

index.html を作成したら、このコードを貼り付けて。
（最小サンプルなので、DOCTYPE や head タグは省略）

```
<html>
  <body>
    <h1>Hello World 🌏</h1>
  </body>
</html>
```

> 今回は Django の記事なので、html や css の説明はしないです。
> html と css はホームページを作成するときのコンテンツ構造やデザインを作成するための言語だから、覚えておいて損はない！（というか、マスターまではいかなくても良いけど、少し理解しておかないとつらいかも🧐）
> 触ったことない人は、ぜひ挑戦してみて！！


それでは index.html が出来上がったので、今度はこのファイルをブラウザに表示させるための設定をしていこう！

必要なことは……
1. sg_pieces 内の views.py に、index.html を表示させるためのpythonコードを書く
2. sg_pieces アプリ内に urls.py ファイルを作成して、sg_pieces 内のファイルを表示させるためのルーティングをする
3. 最後に、プロジェクトのurls.py に、どのアプリの urls.py を読み込むかを設定する

Webページに html ファイルを表示させるためには、上記３ステップが必要だ！
さあ、いこう！！

**1. sg_pieces の中にある、views.py に、下記コードを入力して。**

```python
from django.shortcuts import render

def index(request):
    return render(request, 'sg_pieces/index.html')
```

**2. 次に、sg_pieces の中に、urls.py というファイルを作成する。**

![](/images/c3_p1_3_make.png =590x)
*VSCode では、黄色枠の左側クリックでファイル、右側クリックでフォルダーが作成できる。*

作成した urls.py ファイル内に下記コードを入力する。
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

**3. secret_gallery プロジェクト内の urls.py を以下のように書き換える。**
（最初から存在しているファイルだよ）

書き換えは２箇所。グレー背景のコードを組み込むよ。
> - from django.urls import path`, include`
> - `path('', include('sg_pieces.urls')),`

書き換えというか、ちょっとの追加だね！

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('sg_pieces.urls')),
]
```
※ ファイル内の冒頭にたくさんコメント書いてあるけど、不要であれば削除して良いものだよ。
ファイルの用途について説明してくれているコメントなので、機能には一切関係なし！

ここまでいけたら、ローカルサーバー起動！！

```bash
python3 manage.py runserver
```
 [http://127.0.0.1:8000/](http://127.0.0.1:8000/) をブラウザで読み込むと・・・

![](/images/c3_p1_4_helloworld.png =590x)
*これはズームアップしております*

どうかな！？表示されたかな！？

表示されたら、大成功ーーー🥳<br>


ここで、いま何をしたのか整理しておこう！

まず最初に、index.html ファイルを作成した。
このとき、sg_pieces アプリ内に templates/sg_pieces というディレクトリを作成したよね。

これ、なんかちょっと不思議な構造じゃない？
だって sg_pieces アプリ内に templates フォルダを作って、さらにその中に、sg_pieces という**アプリと同名のフォルダー**を作成したんだよ。
なんでこんなことするの？って思うよね。

実は Django では、複数アプリが存在するプロジェクトでも、それぞれのアプリごとにテンプレート（HTMLファイルのことを Django ではこう呼ぶ）を持てるようになっていて、
「 **アプリ/templates/アプリ名/xxx.html** 」みたいな構造にしておくと、自動で見つけてくれる仕組みになっているのよ。


:::details 仕組みについての詳細説明はこちら
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

*`「"APP_DIRS": True」`のとき、Django は「アプリ/templates/」内に、テンプレートを自動で探しに行ってくれる*
<br>
「でも、これなら templates の中に、アプリと同名のフォルダ作る意味なくない？」と思ったあなた！鋭すぎる！！<br>
そうなの。別に、アプリと同名のフォルダって、作ることが必須ではないの！
ではなぜ作るんだろうね？<br>
たとえばさ、テンプレートファイルって、結構似た名前になることがあったりするのよ。
top.html とか detail.html みたいに、そのWebページの機能をファイル名にしてしまうと、似通った名前になりがち。
そして Django では、アプリごとにテンプレートを持てる、ってさっき言ったじゃない？
そうすると、アプリごとに、似たような名前・・・むしろ、全く同じファイル名を作りたくなる可能性が、ゼロとはいえない。<br>
そうなったときに、templates/アプリ名/top.html とかにしておけば、どこのアプリの top.html なのか、Django がすぐに判別できて、表示上やファイル参照における干渉が起こりづらくなるの。<br>

あと、さっき設定した views.py の中で、render(request, 'sg_pieces/index.html') って書いたの、覚えてるかな？
これのルーティングで、なぜ「sg_pieces/」にしないといけなかったかというと、Django が templates までしか探さないから。
templates よりも階層をもぐってテンプレートを探すことはないから、templates 以降の階層については、ちゃんと明記してあげる必要がある。<br>
ちょっと特殊な例かもしれないけど、templates 内でテンプレートをさらにアプリ名と同じファイルでラッピングすることで、事故を減らせるって覚えてね。
この運用は、実践でも使ってるよ！

:::

「よく分からない・・・」ってなっても大丈夫！まだまだ触り始めたばっかりだもん。
理解よりも、手を動かすことを優先してよ。ぷに蔵も、最初なんか何も理解しないでコピペ祭りしてたから笑

今は、**Djangoではテンプレートはこういうフォルダ構成にする**って覚えちゃってね。

表示された！やったーー！！って思ってくれれば、それだけで十分だよ⭐️


さあ！次に進もう！！


## 02. コピペで３分。「こんにちは、〇〇さん」


## 03. つなげ！「Hello World」 → 「こんにちは、〇〇さん」
## 04. 作品を展示するための大・前・提⭐︎
## 05. 命を吹き込め！！データベースはここに始まる。
## 06. makemigrations と migrate はセット商品ですよ（大事なことなので２回言う）
## 07. admin.py にモデル定義を設定してみよう
## 08. ところでデータベースって設定しましたっけ？
## 09. どんな画面にしようかな〜（htmlを整えるよ！）
