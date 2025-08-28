---
title: "第４章　CRUDアプリはすぐそこに"
emoji: "♍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["django", "python"]
published: false # trueを指定する
# published_at: 2050-08-20 00:00 # 未来の日時を指定する
---

## 01. 概念的に View を考えてみる - Don't Think! FEEL!! -
前章までで、なんとなくのファイルのつながりは見えてきたんじゃないだろうか？


ここからは、本格的に CRUD アプリに触れていこうと思うが・・・
その前に、**CRUD** という言葉は知っている？
データベース管理に必要な４つの機能「 Create / Read / Update / Delete 」それぞれの頭文字を取った言葉。

- **C : Create（新規作成）**
- **R : Read（読み取り＝一覧・詳細）**
- **U : Update（更新）**
- **D : Delete（削除）**

世の中のWebアプリのほとんどは、この CRUD の組み合わせでできているといっても過言ではない！（……と思うよ。たぶん。）


そして Django には、このCRUD をものすごくラクに実装できるクラスが、あらかじめ準備されている！

それが**クラスベースビュー**（CBV）という仕組み。

:::details CBV と FBV の小話

前章までで扱ったファイルの中に views.py があったよね。<br>
あのとき、「**aisatsu/ にアクセスがあったら（リクエスト）」、「greeting.html を表示させる（レスポンス）」という関数を書いた**こと、覚えてる？<br>
自分で関数を定義して書くスタイルのことを、**FBV（Function-Based View）** と呼ぶの。（Django語）
それとは別に、Django があらかじめ準備した View クラスを使用してリクエスト・レスポンスを書くスタイルを **CBV（Class-Based View）** と呼ぶ。（同じく Django語）<br><br>

- **FBV（Function-Based View） : 関数として View を書くスタイルのこと**
- **CBV（Class-Based View） : クラスとして View を書くスタイルのこと**
<br>

やっていることは、どちらも「リクエストを受け取ってレスポンスを返す」なんだけどね。
「コードの書き方」と「便利さ」が違うかな。

> **FBV → 自分で全部の処理を書く。自由度が高いけど手間はかかる**
> **CBV → Django が用意したクラスを継承して CRUD 作成完了。CRUD 処理の最強効率！**

<br>
♦️ 個人的には、CRUD 処理は FBV でも CBV でも好きな書き方をして良いと思うのよ。
でも、CBV を使ってサクッと開発を進められる箇所は、最強クラスの恩恵を受けながら開発することをおすすめするよ。
だって、FBV じゃないと DB 操作できない場面が数多く存在するんだもん！！（絶叫）
どちゃくそ面倒な集計とか、APIから受け取った結果をこねくり回す処理とか、絶句するほど複雑な分析とか……
こういうのはさすがに自動化されたクラスでは対応できないので、がっつり自分で書くしかない😔
::::


## 02. 作品展示。はじめの一歩は CreateView

それでは最初に、CRUD の中の Create の部分を操作してみよう。
使うものは CreateView だ！


順番としては、まず、以下を実行していこう！
> 1. index.html を書きかえて、「作品登録」ページへのリンクを作成する
> 2. 作品登録ページを作成する（greeting.html ファイルは削除）
> 3. views.py に、HTMLテンプレートファイルを表示させるためのクラスを作成する　← New!!
> 4. ルーティングする


1. index.html を書きかえて、「作品登録」ページへのリンクを作成する
まずは index.html を、以下のように書きかえるよ！
```html
<!-- sg_pieces/templates/sg_pieces/index.html -->
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>PRIVATE GALLERY</title>
  </head>
  <body>
      <h1>秘密のプライベートギャラリー</h1>
      <ul>
        <li><a href="{% url 'piece_create' %}">作品を登録する</a></li>
      </ul>
    </div>
  </body>
</html>
```

今回は、html 構文を少し整えたので、本番に寄せた書き方にしてある。（といっても、最低限）
これで、ローカルサーバーを起動した状態で、ブラウザを読み込んでみよう。

こんな画面になっていればOKだよ！
![](/images/c4_p2_1_index.png =580x)


2. 作品登録ページを作成する
index.html と同じ場所に create.html というテンプレートファイルを作成して、下記の内容にしていこう！
- このタイミングで、greeting.html ファイルを削除してしまおう！もう使わないからね。
```html
<!-- sg_pieces/templates/sg_pieces/create.html -->
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>PRIVATE GALLERY</title>
  </head>
  <body>
    <h1>秘密のプライベートギャラリー</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
    <button type="submit">登録</button>
    </form>
  </body>
</html>
```

新しいものが出てきている！
> {% csrf_token %}
> {{ form.as_p }}

でも、ここはひとまず先に進むので、とりあえずコピペか、とりあえず写経！


3. views.py に、HTMLテンプレートファイルを表示させるためのクラスを作成する
```python
# sg_pieces/views.py
from django.shortcuts import render
from django.urls import reverse_lazy
from django.views.generic import CreateView
from .models import GalleryPiece

def index(request):
    return render(request, "sg_pieces/index.html")

class GalleryPieceCreateView(CreateView):
    model = GalleryPiece
    template_name = "sg_pieces/create.html"
    fields = "__all__"
    success_url = reverse_lazy("index")
```

**ここで CreateView が出てきたね！！**

4. ルーティングで index.html と create.html をつなげる
```python
# sg_pieces/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('create/', views.GalleryPieceCreateView.as_view(), name='piece_create'),
]
```

ここまでの設定が終了した状態でブラウザを読み込み直すと、index.html のリンクから、'piece_create' に飛べるようになっている！
この画面に飛べたら大成功だよ！
![](/images/c4_p2_2_create.png =580x)

:::message
「エラーが出てリンクに飛べない」とか、「そもそも index.html の読み込みでエラーになる！」ときは、下記を確認してみよう！
**NoReverseMatch** → urls.py の設定は大丈夫？
**TemplateSyntaxError** → index.html の {% url 'piece_create' %} の書き方チェックしてみて。
**TemplateDoesNotExist** → views.py の template_name のスペルミスはない？
:::

無事に index.html と crate.html 画面が表示された？
ここで一度、新しく出てきたものを見てみよう！

まずはこの２つ。
> - **{% csrf_token %}**
POST フォームを Django で扱う場合の頻出テンプレートタグの「改ざん防止トークン」。これがないと「CSRF（クロスサイトリクエストフォージェリ）対策エラー」でフォーム送信が弾かれるよ！！
※GETフォームでは不要。POST のときだけが必須ね。
> - **{{ form.as_p }}**
「モデルから自動生成したフォーム」を簡易レンダリングで表示してくれる。各フィールドを <p> でラップしているよ。
※ 「レンダリング」とは、View で定義したモデル情報をテンプレートファイルに渡して表示すること。

さ。ここで、CreateView について、少し詳しく見ていこう！
```python
class GalleryPieceCreateView(CreateView):
    model = GalleryPiece
    template_name = "sg_pieces/create.html"
    fields = "__all__"
    success_url = reverse_lazy("index")
```

クラス名は、基本的に自由に付けてOK。
`※ だけど、前にぷに蔵は、公式ドキュメントにも載っていない命名規則で弾かれまくってエラーが出まくった経験ある。どうしても解消できないエラーがあったときは、クラス名を変えると解決することもある・・かも？（でも多分、珍しい例だと思うが。ちなみに、具体例を出したかったが、忘れた😝）`

まずは、CreateView を継承したクラスを作成するときの最小構成で作ってみたよ！

> 最小構成は、以下の４つ
> - model → 使用するモデルを指定。今回は GalleryPiece にデータを登録する想定
> - template_name → View を表示させるテンプレートファイルを指定
> - fields（または form_class） → どのフィールドをフォームの入力対象にするのか
> - success_url → reverse_lazy("リンクの名前") で、登録が成功したときに遷移するページを設定できる

これで、フォームを使用してデータの登録にトライしてみる？？？

・・・・むむ？
ちょっと待って待って！
「 Created at 」って、モデル定義のときに DateTimeField を指定したよね？
でもこれだと、文字列入力になっているね？ DateTimeField なら、カレンダーとか時間選択とかできるもんじゃない？ふつう。

・・・と思った疑問、正しい感覚！！

これを解決するために、新しいファイルを作成する必要があるの。
では、sg_pieces/views.py と同じ場所に forms.py というファイルを作成して！！

中身は、以下のように書くよ！
```python
# sg_pieces/forms.py
from django import forms
from .models import GalleryPiece

class GalleryPieceForm(forms.ModelForm):
    class Meta:
        model = GalleryPiece
        fields = "__all__"
        widgets = {
            "created_at": forms.DateTimeInput(attrs={"type": "datetime-local"})
        }
```

そして、forms.py の導入に伴って、views.py も修正しないといけないの。
```python
# sg_pieces/views.py の該当箇所だけ抜粋
class GalleryPieceCreateView(CreateView):
    model = GalleryPiece
    template_name = "sg_pieces/create.html"
    form_class = GalleryPieceForm
    success_url = reverse_lazy("index")
```

変更点、わかるかな？
views.py にあった `fields = "__all__"` が forms.py の GalleryPieceForm / class Meta 内に移動して、views.py の fields 指定が、form_class の指定に変更になったよ！

views.py と forms.py の設定が完了したら、作品登録ページをリロードしてみよう！

![](/images/c4_p2_3_datetime.png)


これで、データの登録ができるようになったね！
ここで早速データの新規登録を・・・と言いたいところだが、その前に、settings.py の設定をひとつだけしておこう！

```python
# secret_gallery/settings.py
LANGUAGE_CODE = 'en-us'

TIME_ZONE = 'UTC'
```

settings.py に、上記設定になっている箇所があるので探し出して欲しいの。
そしてこれを、以下のように変更して！

```python
# secret_gallery/settings.py
LANGUAGE_CODE = 'ja'

TIME_ZONE = 'Asia/Tokyo'
```

:::message
- LANGUAGE_CODE = 'ja'
'ja'を設定すると、管理画面の表示や、フォームのエラーメッセージなどが日本語表示されるようになるよ。

- TIME_ZONE = 'Asia/Tokyo'
'Asia/Tokyo'を設定すると、データベースに保存されたデータをテンプレートに表示させるとき、Asia/Tokyo 時間への補正を Django が自動でやってくれるよ。
※ データの保存は UTC で行われるので、最初のうちは混乱するかも！
:::

いよいよ本当に、CreateView の設定が終了だよ！
ではここで、入力フォームを使ってデータを登録してみよう！！

![](/images/c4_p2_4_createdata.png =630x)
*入力内容は超絶テキトー*

登録が成功すると reverse_lazy("index") されるよ〜。
登録された内容は管理画面で確認ができるので、[http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/)に行ってみる💨

![](/images/c4_p2_5_admindata.png)
*なんだかとってもよく分からない表示・・・*

登録されたデータが「 GalleryPiece Object(1) 」になっていて、何が登録されているのか分からないね。これは困る。

ここの一覧に何が登録されているのか一目で分かるのが、理想じゃない？

それを叶えるには、sg_pieces/admin.py の設定に１行追加するだけ！！

```python
# sg_pieces/admin.py
from django.contrib import admin
from .models import GalleryPiece

@admin.register(GalleryPiece)
class GalleryPieceAdmin(admin.ModelAdmin):
    list_display = ("id", "name", "created_at", "memo")
```

list_display ！！これだけ！

list_display を使うと、一覧画面に好きなフィールドを列として表示できるようになるよ！
この状態で、もう一度管理画面をリロードして確認してみよう！

![](/images/c4_p2_6_adminlist.png)
*わかりやすい表示になった！*

list_disp;ay で指定したフィールドが全部表示されて、一覧になったね！

これで CreateView が完成だ！！


## 03. base.htmlで共通項すべてまるっと全部おまとめ致します！
突然ですが、index.html と create.html のコードを見比べていただきたい。

```html
 <!-- sg_pieces/templates/sg_pieces/index.html -->
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>PRIVATE GALLERY</title>
  </head>
  <body>
      <h1>秘密のプライベートギャラリー</h1>
      <ul>
        <li><a href="{% url 'piece_create' %}">作品を登録する</a></li>
      </ul>
    </div>
  </body>
</html>
```

```html
<!-- sg_pieces/templates/sg_pieces/create.html -->
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>PRIVATE GALLERY</title>
  </head>
  <body>
    <h1>秘密のプライベートギャラリー</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
    <button type="submit">登録</button>
    </form>
  </body>
</html>
```

・・・・・・・どうだろうか？

なんか・・・・・・・似てない？

というか、一部以外、同じじゃない？？？

```html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>PRIVATE GALLERY</title>
  </head>
  <body>

  ...
  
  </body>
</html>  
```

完全一致すぎる！！
これ、何回も書く必要あると思う？

ないんだよ！！！

同じことは何回も書かない！それがオシャレなコーディング！（そうか？）


まぁ、それは冗談にしても、違うファイルに同じことを書いたままにしておくと、コードを修正するときが大変。
１箇所の修正だけなのに、何個も何個も同じ箇所を修正して回らないといけなくなるから。
だから、共通項でまとめられるファイルは、まとめてしまうのがスマートなのよ。

というわけで、早速いきます！

sg_pieces/templates/sg_pieces 内に base.html というファイルを作成して、下記を書いて！
```html
<!-- sg_pieces/templates/sg_pieces/base.html -->
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}PRIVATE GALLERY{% endblock %}</title>
  </head>
  <body>
      {% block content %}
      <!-- ページごとの中身がここに入る -->
      {% endblock %}
  </body>
</html>
```

次は、**index.html と create.html の書きかえ**をするよ！
```html
<!-- sg_pieces/templates/sg_pieces/index.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー TOP{% endblock %}
{% block content %}

      <h1>秘密のプライベートギャラリー</h1>
      <ul>
        <li><a href="{% url 'piece_create' %}">作品を登録する</a></li>
      </ul>

{% endblock %}
```


```html
<!-- sg_pieces/templates/sg_pieces/create.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー 登録{% endblock %}
{% block content %}

    <h1>秘密のプライベートギャラリー</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
    <button type="submit">登録</button>
    </form>

{% endblock %}
```

どうかな。
base.html を作り出したことで、コードがすごくスッキリしたよね！

そして、**ここで抑えたいポイントは「 block 」と「 extends 」だよ。**

> 1. {% block %} と {% endblock %}
まずは、親テンプレートに {% block %} … {% endblock %} エリアを設置する。
「ここに子テンプレートの内容を差し込みたい」という意思表示エリアね。
>
> 2. {% extends %}
次に、{% block %} … {% endblock %} エリアが設置された子テンプレートの一番最初に {% extends %} を書く。
これを書くことで、「このファイルは**親テンプレートを継承します**という宣言になるよ。
今回だと base.html が親テンプレートね。親＝土台、ってイメージかな。
base.html の「親テンプレート」に対して、index.html や create.html の固有の部分は「子テンプレート」と呼ばれるよ。

コードの流れを見ると、 
1. {% extends %} で土台である base.html を読み込んでから
2. そこに、子テンプレートの「ページごとの中身」のコードを読み込んでいるよね。

:::message
 🫛 豆知識  
{% block title %} や {% block content %} の content や title は『自分でつける名前』だよ。<br>
この名前を
**「子テンプレート（ index.html や create.html ）」で {% block content %} … {% endblock %} と書いて対応させることで、どの部分を差し替えるか Django が判断してくれる**よ。

名前はなんでもいいんだけど、慣習的に「 title / content / header / footer 」みたいな分かりやすい言葉をよく使うかな。
これらの名前にしておけば、HTML コードとしても、どこを指してるいるのかが、直感的に分かりやすいからね。
:::

これで、何回も同じことを繰り返さずにコードが書けるようになった！
見やすい・直しやすい・増やしやすい の三拍子が揃った！
まごうことなき、コードの進化！！！


## 04. Bootstrap GO ON!!
いま時点でも、フォームを使用したデータ登録の流れに問題はないよ。
ちゃんと登録できるよね？

だけど・・・・・・なんだかフォームがデフォルトすぎて、無骨なイメージじゃない？
それも味があるけど、ちょっと色をつけたりしてみたいのが、オンナゴコロってやつだ！


「てことは、CSS いじるのか？」と思って人は、HTML + CSS に馴染んでいるね。
ふふふ、素晴らしい。

しかし、今回に限っては、None！！！

ここでは、CSSフレームワークの Bootstrap を使用していこう！
Bootstrap ってなに？って？

それはだね、CSSを自分で書かなくても、クラスをつけるだけで“それっぽく”整う便利ツール！
HTMLにクラスを付けるだけで、なんか見た目が整ってくれるやつ。これはシビィぜ！

Bootstrap についての詳細設定は、公式がたくさん説明してくれているからね。
[Bootstrap5.3 公式ドキュメント](https://getbootstrap.jp/docs/5.3/getting-started/introduction/)
自分なりに設定をいじってみて、見た目を変えて遊んでみるのもいいと思うよ〜

ということで、まずは親テンプレートの base.html ファイルを修正して、Bootstrap を使える状態にしていこ。
<link href="">で Bootstrap 呼び込みをすることと、ページにス余白をつけて見やすくしたいので、<main class="">を使用して少し整地。

```html
<!-- sg_pieces/templates/sg_pieces/base.html -->
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}PRIVATE GALLERY{% endblock %}</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.7/dist/css/bootstrap.min.css" rel="stylesheet">
  </head>
  <body>
    <main class="container py-4">
        {% block content %}
        <!-- ページごとの中身がここに入る -->
        {% endblock %}
    </main>
  </body>
</html>
```

子テンプレートたちは、Bootstrap 対応コードに修正していくね。
でも、Bootstrapクラスを HTMLテンプレートファイルだけで完結させることはしない。
せっかく Django を使っているし、さっき forms.py を作成したよね！？
せっかくだからそのファイルを活用して最小燃料＆最大火力の組み込みをする！！

今度は index.html ファイルの修正。
ここは単純に Bootstrapクラスで見た目とリンクの見栄えだけを変える。


```html
<!-- sg_pieces/templates/sg_pieces/index.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー TOP{% endblock %}
{% block content %}

  <h1 class="text-center mb-4 mt-4">秘密のプライベートギャラリー</h1>
  <div style="max-width: 300px;">
    <a href="{% url 'piece_create' %}" class="list-group-item list-group-item-action">作品を登録する</a>
  </div>

{% endblock %}
```

次に本命の sg_pieces/forms.py 修正。
ここで、ModelForm に Bootstrap クラスを付けていくよ！
理解できなくても大丈夫よ！表示なんて、変えたいときにググって修正すれば良いから！！

ModelForm に直接 Bootstrap クラスを付けるメリットとしては、CreateView からさらに View を拡張したときにも、拡張先 View で同じ見た目や入力補助を維持できることかな。
つまり、「何度も同じことを書かないオシャレなコーディング」ができる！

```python
# sg_pieces/forms.py
from django import forms
from .models import GalleryPiece

class GalleryPieceForm(forms.ModelForm):
    class Meta:
        model = GalleryPiece
        fields = "__all__"
        widgets = {
            "name": forms.TextInput(attrs={"class": "form-control", "placeholder": "名前"}),
            "created_at": forms.DateTimeInput(
                attrs={"type": "datetime-local", "class": "form-control"},
                format="%Y-%m-%dT%H:%M",
            ),
            "memo": forms.Textarea(attrs={"class": "form-control", "rows": 5}),
        }

    # datetime-local はタイムゾーン情報を持たない→入力形式を受け入れる
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields["created_at"].input_formats = ["%Y-%m-%dT%H:%M"]
```

:::message
**class Meta 内での widgets について、ちょっとだけ補足するね**
class Meta における widgets は、Django の ModelForm などや Form で「フィールドの見た目」を指定するところ。
フォームにおける、「このデータはどんな風に入力させる？」とか「このデータの入力欄は、どんな UI（見た目）にする？」といった部分の設定ができるよ。

- widgets = { … } の辞書キー（左側）は、モデル定義のフィールド名（models.py で定義した name, created_at, memo ）と一致させることが大事
- 値（右側）は、そのフィールドに使いたい入力ウィジェットのクラス（forms.TextInput, forms.DateTimeInput, forms.Textarea, …）を指定する
- attrs={ … } は、HTMLにそのまま出力される属性（class, placeholder, rows, type など）を追加できる。要は、HTMLそのものに直書きする属性を、ここでまとめて書いてしまえる便利なやつ。HTMLテンプレートファイルに直接 class="form-control" と書いても同じ見た目になるけど、forms.py にまとめると全部のフォームに効く。そして、後から修正も1か所で済むよ。

Djangoのフォームは、指定しないと「フィールド型に応じたデフォルトの widget 」を自動で選んでくる。
それで大丈夫なら、設定する必要はないけど、もしも「見た目をカスタマイズしたい！」とか「 placeholder で入力欄に薄文字テキストを入れて分かりやすくしたい！」と思ったら、この widgets を使って一発実装できる！

この説明を読んで「自分は、HTML に直書きした方が分かりやすい・・・」って人がいたら、それでも良いよ！
まずは自分のやりやすいやり方でやって良い。開発は、自由に発想すると、ポロッと閃いたりするから、あまりガチガチにならないで😄
:::

forms.py で フォームの振る舞いの設定は完了したので、create.html には、フォームエラーを表示させるように変更しておこう。
あとは最低限の<label class="", for="">だけを設定だけね。

```html
<!-- sg_pieces/templates/sg_pieces/create.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー 登録{% endblock %}
{% block content %}

  <h4 class="text-center mb-4 mt-4">作品を登録する</h4>

  <form method="post" class="mx-auto" style="max-width: 350px;">
    {% csrf_token %}

    {{ form.non_field_errors }}
    <div class="mb-3">
      <label class="form-label" for="{{ form.name.id_for_label }}">名前</label>
      {{ form.name }}
      {{ form.name.errors }}
    </div>

    <div class="mb-3">
      <label class="form-label" for="{{ form.created_at.id_for_label }}">登録日</label>
      {{ form.created_at }}
      {{ form.created_at.errors }}
    </div>

    <div class="mb-3">
      <label class="form-label" for="{{ form.memo.id_for_label }}">メモ</label>
      {{ form.memo }}
      {{ form.memo.errors }}
    </div>

    <button type="submit" class="btn btn-outline-primary">登録する</button>
  </form>

{% endblock %}
```

一気に子テンプレートのコードが増えてしまったね笑
もしも Webデザイナーさんであれば「あ、はいはい。こんなもんね」で理解できるんだろうね。

ぷに蔵なんかはバックエンド開発は好きだけど、フロントエンドがありえんほどに苦手なので、正直これ以上の見た目をいじるとしたら、無限に時間がかかる！
多分、views.py 系のややこしコードを３本以上書いてる時間合わせても、HTMLテンプレート１個の設定も終わらないんじゃないかな笑
得手不得手ってあるからね〜〜〜😐


:::details おまけに、よく使う便利ウィジェットたち（ぷに蔵のテキストメモを公開しとくｗ）
forms.TextInput → 1行テキスト
forms.Textarea → 複数行テキスト
forms.NumberInput → 数値（<input type="number">）
forms.DateInput → 日付（<input type="date">）
forms.DateTimeInput → 日付＋時刻（<input type="datetime-local">）
forms.EmailInput → メールアドレス（ブラウザバリデーション付き）
forms.PasswordInput → パスワード入力（●●●で隠す）
forms.CheckboxInput → チェックボックス
forms.Select → プルダウン（choices付き）
forms.ClearableFileInput → ファイルアップロード（画像投稿で大活躍）
:::

せっかくなので、記念にフォームで登録笑
![](/images/c4_p4_7_bootstrap.png =580x)


## 05. いつでもキミを見ていたい…… media/ からの配信
テンプレートファイルの成長を感じる・・・。


しかし、作品展示というからには、何か足りない。

気づいてた？

あのさ・・・絵とか写真とか、画像入れたいよね？
というか、文字だけ投稿スタイルだと、エッセイや詩や小説の展示しかできない。
きっと、絵を描くことが得意な人だっているよね！？（二次創作好きとしては、いつも楽しませていただいています！）

ということで、画像登録もしていこう！


実は、メディア投稿って、もしもインターネットに公開にするとなったら少し扱い方が


## 06. 登録された作品たちを並べてみるは ListView
## 07. 愛しの DetailView
## 08. 間違い発見！！修正View はあるのか？
## 09. 悲しみのデリート作業。（DeleteView）
## 10. 本当の現場の delete 作業
## 11. View フィナーレは TemplateView で華麗に締める！
## 📕 突然現れた {% csrf_token %} さんは、どちら様？

