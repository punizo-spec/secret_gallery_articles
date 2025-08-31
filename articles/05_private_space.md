---
title: "第４章　CRUDアプリはすぐそこに"
emoji: "♍️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["django", "python"]
published: true
published_at: 2050-08-20 00:00 # 未来の日時を指定する
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
```django
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
POST フォームを Django で扱う場合の頻出テンプレートタグの「改ざん防止トークン」が {% csrf_token %} だよ。
CSRF とは、「クロスサイトリクエストフォージェリ」という、悪意あるサイトから勝手にリクエストを送られる攻撃手法のこと。
Django は {% csrf_token %} をフォームに入れておくだけで、自動的に防御してくれる！
逆にいうと、これがないと「CSRF対策エラー」で、フォーム送信が弾かれる。
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

```django
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
```django
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
```django
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


```django
<!-- sg_pieces/templates/sg_pieces/create.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー Create{% endblock %}
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

```django
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
    <main class="container py-1">
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


```django
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

```django
<!-- sg_pieces/templates/sg_pieces/create.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー Create{% endblock %}
{% block content %}

  <h4 class="text-center mb-4 mt-4">作品を登録する</h4>

  <form method="post" class="mx-auto" style="max-width: 450px;">
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

    <div class="text-center">
      <button type="submit" class="btn btn-outline-primary">登録する</button>
    </div>

  </form>

{% endblock %}
```

一気に子テンプレートのコードが増えてしまったね笑
もしも Webデザイナーさんであれば「あ、はいはい。こんなもんね」で理解できるんだろうね。

ぷに蔵は、バックエンド開発は好きだけど、フロントエンドがありえんほどに苦手なので、正直これ以上の見た目をいじるとしたら、無限に時間がかかる！
多分、views.py 系のややこしコードを３本以上書いてる時間合わせても、HTMLテンプレート１個の設定も終わらないんじゃないかな笑
得手不得手ってあるからね〜〜〜😐


:::details おまけに、よく使う便利ウィジェットたち（ぷに蔵のテキストメモを公開しとくｗ）
forms.TextInput → 1行テキスト
forms.Textarea → 複数行テキスト
forms.NumberInput → 数値（<input type="number">）
forms.DateInput → 日付（<input type="date">）
forms.DateTimeInput → 日付＋時刻（<input type="datetime-local">）
forms.EmailInput → メールアドレス（ブラウザバリデーション付き）
forms.PasswordInput → パスワード入力（●●●で隠れる）
forms.CheckboxInput → チェックボックス
forms.Select → プルダウン（choices付き）
forms.ClearableFileInput → ファイルアップロード（画像投稿で大活躍）
:::

せっかくなので、記念にフォームでデータ登録してみて！
![](/images/c4_p4_7_bootstrap.png =580x)


## 05. いつでもキミを見ていたい…… media/ からの配信
テンプレートファイルの成長を感じる・・・。


しかし、作品展示というからには、何か足りない。

気づいてた？

あのさ・・・絵とか写真とか、画像入れたいよね？
というか、文字だけ投稿スタイルだと、エッセイや詩や小説の展示しかできない。
きっと、絵を描くことが得意な人だっているよね！？（二次創作大好きマンなので、いつも楽しませていただいています！）

ということで、画像登録もしていこう！


実は、メディア投稿って、もしもインターネットに公開にするとなったら少し扱いがややこしくなってくる。
今回はあくまで「最初に Django に触れる」想定だから、ローカル用セットアップを紹介するね！

1. まずは画像処理のためのライブラリをインストール（必ず仮想環境で実行してね）
```bash
pip install Pillow
```

2. media/ ディレクトリの作成
manage.py がある階層（ルートディレクトリと呼ぶよ）に「 media 」フォルダを作成する。

3. sg_pieces/models.py を修正
image フィールドの追加をして、メディア情報を保存できるように変更するよ
```python
from django.db import models
from django.urls import reverse

class GalleryPiece(models.Model):
    name = models.CharField(max_length=100)  
    created_at = models.DateTimeField(null=True, blank=True)
    memo = models.TextField()
    image = models.ImageField(upload_to='images/', blank=True, null=True)

    class Meta:
        verbose_name = "ギャラリー作品"
        verbose_name_plural = "ギャラリー作品一覧"
        db_table = "gallery_piece"

    def __str__(self):
        return self.name

    def get_absolute_url(self):
        return reverse("piece_detail", args=[self.pk])
```
:::message
**新しい関数（def）が２つ。**
これは一体・・・？と思うだろうけど、そんなに難しいことはないの。
- **def __str__(self)**
管理画面でデータの詳細ページにいくと、表示が「 GalleryPiece Object(1)」になっている箇所があるのよ。
![](/images/c4_p5_8_adminobj.png =400x)
それの表示を、分かりやすくするだけ。
return self.name にしたから、models.py のフィールドの "name" の文字列が表示されるようになったね。
![](/images/c4_p5_9_adminname.png =400x)
あと、Pythonシェルで print したときの戻り値にも使われる・・・とかあるけど、いまは必要ないのでスルーしよう笑
<br>

- **def get_absolute_url(self)**
これは、テンプレートからリンクするときに、便利に指定できる関数。
DetailView や ListView のときに恩恵を受けるかな。
管理画面でデータを作成/編集したときに、自動で、そのデータの詳細ページに飛ばしてくれたりする。
Django プロジェクト上での挙動としては、CreateView や UpdateView のクラスで success_url の指定がされていない場合、個別データの URLリンクを取得して、そこのページに飛ばす、という挙動をする。
自分で `redirect(piece.get_absolute_url())` と書いて同じ動きをさせることもできるんだけど・・・

ここらへんはちょっとややこしくなちゃうから、いまは「そんな機能があるんだね〜」とだけ思っておいて！
:::

4. models.py を修正したのでマイグレート
models.py で DB のテーブル構造を修正したので、マイグレートが必要になるね。
```bash
python3 manage.py makemigrations
python3 manage.py migrate
```

マイグレートが完了したら、管理画面で image フィールドが追加されていることが確認できるよ。
![](/images/c4_p5_10_adminedit.png)


5. 次は、settings.py の設定
下記の、メディア設定を追記。
末尾に追加で良いんだけど、おそらくデフォルトの settings.py の場合、末尾付近に
「 STATIC_URL = 'static/' 」
という設定があると思うから、それのすぐ下くらいに追記しておくと、分かりやすいかな。

```python
MEDIA_URL = "/media/"
MEDIA_ROOT = BASE_DIR / "media"
```

6. ローカルサーバーのときだけ、Django が MEDIA を配信するように、 urls.py に設定追記
```python
from django.conf import settings
from django.conf.urls.static import static
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('create/', views.GalleryPieceCreateView.as_view(), name='piece_create'),
]

# /media/ をDjangoが直接配信（DEBUG時のみ）
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

これで、http://127.0.0.1:8000/media/images/xxx.png みたいなURLでブラウザからアクセスできるようになる！

:::message
**if settings.DEBUG について**
settings.py の中に、「 DEBUG = True 」という設定があると思う。
これは、開発中のときだけ True にしておくもので、サーバーにデプロイするときには False にする。
True にしておくと得られるメリットは、下記を行ってくれること。
- エラーページの詳細
- 静的ファイルやメディアの配信など
本番環境でエラーページの詳細が表示されてしまうと、Webアプリケーションのディレクトリ構造などが知られてしまい、セキュリティ的にとても危険。
静的ファイルやメディア配信は、本番環境では、通常「 Nginx や S3 」といったサービスが配信するんだけど、ローカル環境では Django が簡易に配信をしてくれる。

そして、「ローカル環境では Django が簡易に配信をしてくれる」ためには、urls.py で static() 関数を使用する必要があるの。

**if settings.DEBUG**の設定をしておけば、ローカル環境でのみエラー詳細が確認できるようになり、静的ファイルやメディア配信は Django がやってくれるようになるよ。
:::



7. CreateView に image フィールドを追加するので forms.py と create.html の修正
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
            "image": forms.ClearableFileInput(attrs={"class": "form-control"}),
        }

    # datetime-local はタイムゾーン情報を持たない→入力形式を受け入れる
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields["created_at"].input_formats = ["%Y-%m-%dT%H:%M"]

    # ローカル検証の例（任意）
    def clean_image(self):
        img = self.cleaned_data.get("image")
        if not img:
            return img
        if img.size > 5 * 1024 * 1024:
            raise forms.ValidationError("画像サイズは 5MB 以下にしてください。")
        if not img.content_type.startswith("image/"):
            raise forms.ValidationError("画像ファイルをアップロードしてください。")
        return img
```

:::message
**def clean_image(self)？**
画像アップロードって「想定外のサイズ」「画像じゃないファイル」が飛んでくる可能性が高いのよ。
開発環境だと自分しか触らないけど、本番環境だと利用者が自由にアップできるから、ここで制御入れておかないと、危険なファイルや大容量のファイルがアップロードされたりする可能性がゼロじゃない！むしろ、可能性は高め！（画像ファイルにだって、PDF にだって、テキストファイルにだって、ウイルスは仕込めるからね⚠️）

この関数は「追加のバリデーション」だから、書かなくてもアプリは動く。
でもね、もしもルールに合わないファイルをアップロードしようとしたとき、フォーム上でエラーが表示されて、 DB へは保存されない仕組みになるんだ。

Django の ImageField は最低限のバリデーションはしてくれている。
だけど、独自ルールをプラスして ModelForm 経由のデータでも def clean_XXX() でチェックするのが、現代セキュリティのセオリー！
:::

create.html は、変更と追加箇所のみ記載
```django
<!-- sg_pieces/templates/sg_pieces/create.html -->
  <!-- 重要：画像アップロードフォームは 必ず enctype="multipart/form-data" を付ける！付けないと動かない -->
  <form method="post" class="mx-auto" style="max-width: 450px;" enctype="multipart/form-data">

    …（略）…

    <!-- 画像登録フィールドの追加 -->
    <div class="mb-3">
      <label class="form-label" for="{{ form.image.id_for_label }}">画像</label>
      {{ form.image }}
      {{ form.image.errors }}
    </div>
```

これで、画像を保存する土壌は出来上がり！

## 06. 登録作品たちを並べてみるは ListView

```django
<!-- sg_pieces/templates/sg_pieces/index.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー TOP{% endblock %}
{% block content %}

      <h1 class="text-center mb-4 mt-4">秘密のプライベートギャラリー</h1>
      <div class="list-group d-grid gap-3 mx-auto text-center" style="max-width: 300px;">
        <a href="{% url 'piece_create' %}" class="list-group-item list-group-item-action">作品を登録する</a>
        <a href="{% url 'piece_list' %}" class="list-group-item list-group-item-action">作品を確認する</a>
      </div>

{% endblock %}
```

さらに、index.html ファイルをコピーして、ファイル名は list.html に変更。
それを、以下に書き直してみよう。
```django
<!-- sg_pieces/templates/sg_pieces/list.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー List{% endblock %}
{% block content %}

  <h1 class="text-center mb-4 mt-4">秘密のプライベートギャラリー</h1>
    <ul class="list-group list-group-flush mx-auto mt-4" style="max-width: 400px;">
      {% for piece in pieces %}
        <li class="list-group-item">
          {{ piece.id }} {{ piece.name }}
        </li>
      {% empty %}
        <li class="list-group-item">まだ作品が登録されていません😢</li>
      {% endfor %}
    </ul>

{% endblock %}
```

あまり馴染みのない使い方をされている、お馴染み for~ が出てきているよ！


```python
# sg_pieces/urls.py
from django.conf import settings
from django.conf.urls.static import static
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('create/', views.GalleryPieceCreateView.as_view(), name='piece_create'),
    path('list/', views.GalleryPieceListView.as_view(), name='piece_list'),
]

# ★DEBUG時のみ、/media/ をDjangoが直接配信
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

views.py に ListView を継承してクラスを作成
```python
# sg_pieces/views.py
from django.shortcuts import render
from django.urls import reverse_lazy
from django.views.generic import CreateView, ListView
from .models import GalleryPiece
from .forms import GalleryPieceForm

def index(request):
    return render(request, "sg_pieces/index.html")

class GalleryPieceCreateView(CreateView):
    model = GalleryPiece
    template_name = "sg_pieces/create.html"
    form_class = GalleryPieceForm
    success_url = reverse_lazy("index")

class GalleryPieceListView(ListView):
    model = GalleryPiece
    template_name = "sg_pieces/list.html"
    context_object_name = "pieces"
    ordering = ["id"]
    paginate_by = 10
```
ListView の最小構成は model と template_name の指定だけで良いんだけど、今回は少し機能を追加。
> - **`context_object_name = "pieces"`**　オブジェクトに名前を付けると、テンプレートファイル側では「 object_list 」または「 pieces 」（指定した名前）のどちらでも受け取ることができるようになる。表示させるときは、{% for piece in object_list（/pieces） %}…{% endfor %}を使うよ。
> - **`ordering = ["id"]`**　id が小さい方から順番に並ぶよ。大きい方から並べたいときんは ["-id"] にしてみてね。
> - **`paginate_by = 10`**　テンプレートファイル側で10件ずつ表示させる。あとで実装するよ。


これで、ブラウザリロードだぁあ！！！

**でん！！**
![](/images/c4_p6_11_index.png =580x)

**ででん！！！**
![](/images/c4_p6_12_list.png =580x)


・・・・・・立派に並んでおる・・・


が・・・・・・・・


これでは何も発展しないね笑


やっぱり、リストとして並ぶなら、
> - データの詳細を見たり
> - データの編集ができたり
> - データの削除ができたり

したいよね！！！


したいなら、作っちゃおう YO ⭐️


それでは、最初にそれぞれのリンクだけ、すべて設置してしまおう！
ここからは、DetailView / UpdateView / DeleteView まで、ノンストップでいくぞ！！！
振り落とされるな！！！

挫けそうになったら、ガンガンコピペで良いんだよ！！
頭より手を動かしていこう！！考えるのは後からでもできるよ！
心に火がついているうちに駆け抜けるぞ🔥


♦️ リストに「画像表示」＋ 「詳細・編集・削除」のリンクを追加
```django
<!-- sg_pieces/templates/sg_pieces/list.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー List{% endblock %}
{% block content %}

  <h1 class="text-center mb-4 mt-4">秘密のプライベートギャラリー</h1>
    <ul class="list-group list-group-flush mx-auto mt-4" style="max-width: 400px;">
      {% for piece in pieces %}
        <li class="list-group-item">
          {{ piece.id }} {{ piece.name }}
        <span class="float-end">
          <!-- 画像のサムネイル表示 -->
          {% if piece.image %}
          <img src="{{ piece.image.url }}" alt="{{ piece.name }}" style="height:40px; width:40px; object-fit:cover; border-radius:6px;">
          {% endif %}
          <!-- リンク表示 -->
          <a href="{% url 'piece_detail' piece.pk %}" class="btn btn-outline-info btn-sm">詳細</a>
          <a href="{% url 'piece_update' piece.pk %}" class="btn btn-outline-secondary btn-sm">編集</a>
          <a href="{% url 'piece_delete' piece.pk %}" class="btn btn-outline-danger btn-sm">削除</a>
        </span>
        </li>
      {% empty %}
        <li class="list-group-item">まだ作品が登録されていません😢</li>
      {% endfor %}
      <a href="{% url 'index' %}" class="mt-4 mx-auto btn btn-outline-primary" style="width:130px;">トップに戻る</a>
    </ul>

{% endblock %}
```

♦️ views.py に各クラスを配置（仮）
```python
# sg_pieces/views.py
from django.shortcuts import render
from django.urls import reverse_lazy
from django.views.generic import CreateView, ListView, DetailView, UpdateView, DeleteView
from .models import GalleryPiece
from .forms import GalleryPieceForm

def index(request):
    return render(request, "sg_pieces/index.html")

class GalleryPieceCreateView(CreateView):
    model = GalleryPiece
    template_name = "sg_pieces/create.html"
    form_class = GalleryPieceForm
    success_url = reverse_lazy("index")

class GalleryPieceListView(ListView):
    model = GalleryPiece
    template_name = "sg_pieces/list.html"
    context_object_name = "pieces"
    ordering = ["id"]
    paginate_by = 10

class GalleryPieceDetailView(DetailView):
    pass

class GalleryPieceUpdateView(UpdateView):
    pass

class GalleryPieceDeleteView(DeleteView):
    pass
```


♦️ urls.py でルーティング（現在の urls.py の urlpatterns = [] 部分のみ記載）
```python
# sg_pieces/urls.py
urlpatterns = [
    path('', views.index, name='index'),
    path('create/', views.GalleryPieceCreateView.as_view(), name='piece_create'),
    path('list/', views.GalleryPieceListView.as_view(), name='piece_list'),

    # 次回パートから順次実装。いまは仮置き
    path('detail/<int:pk>/', views.GalleryPieceDetailView.as_view(), name='piece_detail'),
    path('update/<int:pk>/', views.GalleryPieceUpdateView.as_view(), name='piece_update'),
    path('delete/<int:pk>/', views.GalleryPieceDeleteView.as_view(), name='piece_delete'),
]
```

♦️ 表示確認（暫定）
どどん！！
![](/images/c4_p6_13_listlink.png)

うん。いい感じ！！
ここに、リンク先の処理を追加していくね！

:::message
**今回のポイント： テンプレートタグ**

**Django テンプレートタグについて**
これまでに出てきた Django のテンプレートタグは、{% url 'piece_list' %} だったよね。
「[03. つなげ！「Hello World」 → 「こんにちは、〇〇さん」](https://zenn.dev/punizo/articles/04_learning_to_views#03.-%E3%81%A4%E3%81%AA%E3%81%92%EF%BC%81%E3%80%8Chello-world%E3%80%8D-%E2%86%92-%E3%80%8C%E3%81%93%E3%82%93%E3%81%AB%E3%81%A1%E3%81%AF%E3%80%81%E3%80%87%E3%80%87%E3%81%95%E3%82%93%E3%80%8D)」で、
**{% url 'ルート名' %} 構文は、HTMLテンプレートファイルで使える Django の独自機能** と言っていたこと、覚えている？

でも今回、ちょっと違う様相の使い方をしているの。
気がついた？
そうなの。
```django
{% for piece in pieces %}
  ...
{% endfor %}
```
コレ、{% url 'xxx' %} と似ているけど、初見の形だよ！

実はね、 Django で使えるテンプレートタグには、大きく分けて2種類あるの。
- **便利系タグ**：**{% url 'xxx' %}**, **{% csrf_token %}** など  
  → Django の機能を呼び出すための専用タグ。

- **制御系タグ**：**{% for %}**, **{% if %}** など  
  → Python の書き方に似てるけど、Django テンプレートの中でだけ使える仕組み。
  本物の Python ではないけど、似た感覚で書けるようにしてくれている。

今回出てきた、これ。
```django
{% for piece in pieces %} 
  ...
{% endfor %}
```
これは「制御タグ」のひとつで、一覧データを順番に取り出して表示するためのものなのよ。
そこに、{% empty %} も書いておけば、データが0件のとき専用メッセージを出すこともできる！
:::

:::message

**今回の新出ポイント**

- **🟦 URLの <int:pk> について**
pk は **primary key**の略（=自動採番のID）。
urls.py で 「path('detail/<int:pk>/', …………)」 と書くと、URL の数字の <int:pk>（＝データ識別の一意の id ）がビューに渡る。
それを、テンプレート側では 
```html
<a href="{% url 'piece_detail' piece.pk %}">詳細</a>
```
みたいな形で、受け取る。

データの詳細確認、編集、削除というのは、一つのデータに対してだけ行う作業だよね。
だから、一意の識別子を渡してあげないと、Django 側は「え？どのデータの詳細表示させるんだよ？？？」って混乱してしまうのよ。


- **🟦 リストにサムネ表示**
{{ piece.image.url }} で画像の URL にアクセスできる（MEDIA_URL 配信が効いてる）。  
サムネは height/width 固定 ＋ object-fit: cover で四角にトリミングしたよ。
これは、あとで画像データを登録してから、リスト表示で改めて確認してみようポイント！
:::


## 07. 愛しの DetailView
ListView、勢いでイケるかと思ったけど、わりと負荷高めな内容だったね笑
書き終えたぷに蔵、けっこうぐったりしている😣

だから少し、まったりモードでいきたい。
やること３つ！
♦️ detail.html ファイルを作って、コードを設置
♦️ views.py に DetailView 継承のクラスを準備
♦️ リスト一覧から「詳細」ページに飛べたら完了！

では、いこう。
1. まずは、detail.html ファイルの作成（ index.html や list.html ファイルをコピペの後、修正でいいぞ！）
中身は、下記のコードを記入。・・・か、自分の好きなように CSS は変えていいからね！
```django
<!-- sg_pieces/templates/sg_pieces/detail.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー Detail{% endblock %}
{% block content %}

<div class="container">
  <div class="row justify-content-center">
    <div class="col-12 col-md-8 col-lg-6">

      <h1 class="text-center mb-4 mt-4">ギャラリー 個別展示</h1>

      <div class="card shadow-sm">
        <div class="card-body text-center"><!-- ← 基本は中央寄せ -->
          
          <!-- 画像（中央） -->
          {% if object.image %}
            <img src="{{ object.image.url }}" alt="{{ object.name }}"
                 class="d-block mx-auto rounded mb-3"
                 style="width:280px; height:280px; object-fit:cover;">
          {% else %}
            <div class="d-flex align-items-center justify-content-center bg-light rounded mb-3"
                 style="width:280px; height:280px; margin:0 auto;">
              <span class="text-muted">画像が登録されていません</span>
            </div>
          {% endif %}

          <!-- タイトル（中央） -->
          <h4 class="mb-3">{{ object.name }}</h4>

          <!-- プロフィール（ここだけ左寄せ） -->
          <div class="text-start mx-auto" style="max-width: 90%;">
            <p class="mb-1"><strong>投稿日：</strong>{{ object.created_at|date:"Y-m-d H:i" }}</p>
            <p class="mb-0"><strong>メモ：</strong></p>
            <div class="border rounded p-2 mt-1" style="white-space:pre-wrap;">{{ object.memo }}</div>
          </div>

          <!-- ボタン群（中央） -->
          <div class="d-flex justify-content-center gap-2 mt-4">
            <a href="{% url 'piece_list' %}" class="btn btn-outline-primary">リストに戻る</a>
            <a href="{% url 'piece_update' object.pk %}" class="btn btn-outline-secondary">編集</a>
            <a href="{% url 'piece_delete' object.pk %}" class="btn btn-outline-danger">削除</a>
          </div>

        </div>
      </div>

    </div>
  </div>
</div>
{% endblock %}
```
個別ページからも編集、削除ができるようにしておくと便利かもしれないよね？


2. views.py を修正する（該当箇所のみ記載）
```python
# sg_pieces/views.py
class GalleryPieceDetailView(DetailView):
    model = GalleryPiece
    template_name = "sg_pieces/detail.html"
```

これで「詳細」ページへGO ！

![](/images/c4_p7_14_detail.png =580x)

おおーーーー！いい感じだね！！！
よし！次にいこう！

## 08. 修正は UpdateView にお任せ
UpdateView 
♦️ update.html ファイルを作って、コードを設置
♦️ views.py に UpdateView 継承のクラスを準備（form_c）
♦️ リスト一覧から「編集」ページに飛べたら完了！


手順は Detail と一緒だ！
1. update.html ファイルを作って、コードを設置
これさ、気づいた人いるかな？
views.py で form_class = GalleryPieceForm を指定すれば、そのまま GalleryPiece のフォームが使えるよね？
ということは・・・これ、create.html と、中身が全く一緒なのよ。

なので、ぷに蔵は面倒くさがりなので、create.html をコピペして、「登録」 → 「編集」に文字列変更程度だけで終わらせました！（白杖）
一応、登録画像のプレビューも入れてあります。
```django
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー Update{% endblock %}
{% block content %}

  <h4 class="text-center mb-4 mt-4">作品を編集する</h4>

  <form method="post" class="mx-auto" style="max-width: 450px;" enctype="multipart/form-data">
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

    <div class="mb-3">
      <!-- 画像のプレビューを表示する -->
      {% if view.object and view.object.image %}
        <div class="text-center mb-3">
          <img src="{{ view.object.image.url }}" alt="{{ view.object.name }}"
              style="width:120px; height:120px; object-fit:cover; border-radius:8px;">
          <div class="text-muted small mt-1">現在の画像</div>
        </div>
      {% endif %}      
      <label class="form-label" for="{{ form.image.id_for_label }}">画像</label>
      {{ form.image }}
      {{ form.image.errors }}
    </div>
    
    <div class="text-center">
      <button type="submit" class="btn btn-outline-primary">保存する</button>
    </div>

  </form>

{% endblock %}
```

2. views.py を修正する（該当箇所のみ記載）
```python
# sg_pieces/views.py
class GalleryPieceUpdateView(UpdateView):
    model = GalleryPiece
    template_name = "sg_pieces/update.html"
    form_class = GalleryPieceForm
```
今回は、success_url 付けない仕様でやってみようかな。


models.py で、下記の設定したじゃない？
```python
def get_absolute_url(self):
        return reverse("piece_detail", args=[self.pk])
```
ということは、編集が完了すれば、{% url 'piece_detail' object.pk %} に遷移してくれるはず！
それも一緒に確かめてみよう！


もし、登録・編集の際に手持ち画像の準備が面倒なら、ぷに蔵 png たちを使ってくれ！ ©︎ぷに蔵

| めんだこ    | あひる | ちょうちんあんこう  |
| --------- | -- | ---------------- |
| ![](/images/image_mendako.png =220x) | ![](/images/image_ahiru.png =220x)  | ![](/images/image_ankho.png =220x) |


♦️ リスト一覧から「詳細」ページに飛べたら完了！
作品登録画面とシンクロ率200%笑
![](/images/c4_p8_15_update.png)

作品編集で、個別ページに飛んだよ！
つまり、`def get_absolute_url(self)` がちゃんと機能しているということだ！
![](/images/c4_p8_16_detailjump.png =550x)

お！サムネ画像も表示されたね！！
![](/images/c4_p8_17_list.png)


次の作業にいく前に、全部に画像入れてみようかな🧐

## 📕 Create / Update を同じテンプレで使い回す小技
ここまで作ってみて、あまりに create.html と update.html の一致箇所が多かったことにびっくりだ。

これって、create.html と update.html で使う共通テンプレを作っちゃった方が、メンテナンス性上がる気がしない？
後から楽になる手間は掛ける派の人は、ぜひ共通テンプレ化に挑戦してみて！
面倒な人は、飛ばしても大丈夫だよ！見た目とか何も変わらないから。

２ステップでいこう！
♦️ 1. piece_form.html の作成
♦️ 2. views.py の書きかえ
たったこれだけだ！！！


1. piece_form.html ファイルを作成して、そこに下記コードを入力
```django
<!-- sg_pieces/templates/sg_pieces/piece_form.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}{{ view.object|default_if_none:"Create" }}{% endblock %}
{% block content %}
  <h4 class="text-center mb-4 mt-4">
    {{ view.object|yesno:"作品を編集する,作品を登録する" }}
  </h4>

  <form method="post" class="mx-auto" style="max-width: 450px;" enctype="multipart/form-data">
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

    <div class="mb-3">
      {% if view.object and view.object.image %}
        <div class="text-center mb-3">
          <img src="{{ view.object.image.url }}" alt="{{ view.object.name }}"
              style="width:120px; height:120px; object-fit:cover; border-radius:8px;">
          <div class="text-muted small mt-1">現在の画像</div>
        </div>
      {% endif %}

      <label class="form-label" for="{{ form.image.id_for_label }}">画像</label>
      {{ form.image }}
      {{ form.image.errors }}
    </div>
    
    <button type="submit" class="btn btn-outline-primary">
      {{ view.object|yesno:"保存する,登録する" }}
    </button>
  </form>
{% endblock %}
```

> **{{ view.object|yesno:"Update,Create" }} の使い方**
> view.object が存在する（yes）なら "Update"、存在しない（no）なら "Create" が表示される。
> これで、create / update それぞれで、下記の設定がされていることと同じ挙動になる。
> - {% block title %}秘密のプライベートギャラリー Create{% endblock %}
> - {% block title %}秘密のプライベートギャラリー Update{% endblock %}
>
> {{ view.object|yesno:"保存する,登録する" }} でも、同じ考え方ができるね。
> view.object は存在する？しない？
> 　yes → 保存する（view.object がある）
> 　no → 登録する（view.object がない）
> もしも「view.object ってなにーー😩」ってなりそうだったら、「DB に登録されているデータ」って考えると分かりやすいかも。
> DB に登録されたデータがある（view.object → yes） or DB に登録されたデータはない（view.object → no）の違いだよ！

:::message
**view.object**は、ビューからテンプレートに渡ってくる１個のデータのこと。
CreateView と UpdateView は、どっちも 1件のデータを扱うでしょ？
Django が「いま対象にしているデータだよ」って渡ってくるのが object。
ビューから渡されるから view.object で取得できる。このとき、ビューからはモデル定義の __str__ に指定したものが渡されるよ。
今回のモデルなら
```python
def __str__(self):
    return self.name
```
となっているから、name フィールドの文字列が渡ってくる。<br>
{% block title %}秘密のプライベートギャラリー めんだこ{% endblock %}
みたいに表示設定したいときには、
- **{% block title %}秘密のプライベートギャラリー {{ view.object|default_if_none:"Create" }}{% endblock %}**

……を使うと、view.object にデータがない（＝新規登録）ときは、下記のように表示される。
新規登録：秘密のプライベートギャラリー Create
編集画面：秘密のプライベートギャラリー めんだこ
:::


2. views.py の書きかえ
```python
# sg_pieces/views.py
# GalleryPieceCreateView と GalleryPieceUpdateView のみ変更
class GalleryPieceCreateView(CreateView):
    model = GalleryPiece
    template_name = "sg_pieces/piece_form.html"
    form_class = GalleryPieceForm

class GalleryPieceUpdateView(UpdateView):
    model = GalleryPiece
    template_name = "sg_pieces/piece_form.html"
    form_class = GalleryPieceForm
```

以上で完了！！
とても簡単に共通テンプレ化できたね。


## 09. 悲しみのデリート作業。（DeleteView）
ビュー、最後のひとつ。
削除まで辿り着いたね！

まずは、下記の２ステップでいくよ！
♦️ 1. delete.html の作成
♦️ 2. views.py の書きかえ

1. delete.html ファイルを作成して、下記のコードを入力
```django

{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー Delete{% endblock %}
{% block content %}

  <h4 class="text-center mb-4 mt-4">作品の削除確認</h4>

  <form method="post" class="mx-auto" style="max-width: 450px;">
    {% csrf_token %}
    {{ form.non_field_errors }}

  <div class="text-center">『{{ object.id }}.{{ object.name }}』を本当に削除しますか？</div>
  
    <div class="text-center">
      <button type="submit" class="mt-5 btn btn-outline-danger btn-sm">削除する</button>
    </div>

  </form>

{% endblock %}
```

2. views.py の書きかえ
```python
# sg_pieces/views.py
class GalleryPieceDeleteView(DeleteView):
    model = GalleryPiece
    template_name = "sg_pieces/delete.html"
    success_url = reverse_lazy("piece_list")
```

これで完了！！
データを登録して、削除をしてみて！
![](/images/c4_p9_18_list.png =680x)

無事に削除できたかな？

でもなんか、削除されたけど、ちょっと味気なかったな〜。
もう少し「削除完了した！」感を出したいから、完了メッセージを出してみよう！

♦️ views.py をもう少し修正（モジュールも、１つ増えるよ）
```python
# sg_pieces/views.py
from django.contrib import messages

class GalleryPieceDeleteView(DeleteView):
    model = GalleryPiece
    template_name = "sg_pieces/delete.html"
    success_url = reverse_lazy("piece_list")

    def post(self, request, *args, **kwargs):
        obj = self.get_object()  # 削除対象を取得
        messages.success(request, f"{obj.id}. {obj.name}を削除しました。")
        return super().post(request, *args, **kwargs)
```

♦️ list.html を修正
追加になるところが、**{% if messages %} … {% endif %} の部分ね。
```django
<!-- sg_pieces/templates/sg_pieces/list.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー List{% endblock %}
{% block content %}

  <h1 class="text-center mb-4 mt-4">秘密のプライベートギャラリー</h1>
    <ul class="list-group list-group-flush mx-auto mt-4" style="max-width: 400px;">
      {% for piece in pieces %}
        <li class="list-group-item">
          {{ piece.id }} {{ piece.name }}
        <span class="float-end">
          <!-- 画像のサムネイル表示 -->
          {% if piece.image %}
          <img src="{{ piece.image.url }}" alt="{{ piece.name }}" style="height:40px; width:40px; object-fit:cover; border-radius:6px;">
          {% endif %}
          <!-- リンク表示 -->
          <a href="{% url 'piece_detail' piece.pk %}" class="btn btn-outline-info btn-sm">詳細</a>
          <a href="{% url 'piece_update' piece.pk %}" class="btn btn-outline-secondary btn-sm">編集</a>
          <a href="{% url 'piece_delete' piece.pk %}" class="btn btn-outline-danger btn-sm">削除</a>
        </span>
        </li>

      {% empty %}
        <li class="list-group-item">まだ作品が登録されていません😢</li>
      {% endfor %}

      {% if messages %}
        {% for msg in messages %}
          <div class="alert alert-{{ msg.tags|default:'info' }} alert-dismissible fade show" role="alert">
            {{ msg }}
            <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
          </div>
        {% endfor %}
      {% endif %}

      <a href="{% url 'index' %}" class="mt-4 mx-auto btn btn-outline-primary" style="width:130px;">トップに戻る</a>
    </ul>

    <!-- Bootstrapの閉じるボタンを効かせる用 -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.7/dist/js/bootstrap.bundle.min.js"></script>

{% endblock %}
```

削除メッセージが出るようになった！
ついでに、Bootstrap で閉じるボタンも入れてみたよ。
![](/images/c4_p9_19_msglist.png =680x)

{% if messages %} ... {% endif %} エリアで削除メッセージを表示させていることは分かるよね？
これがどこから渡ってきたのかだけ確認しておこう！

views.py の下記のコード部分。
```python
# sg_pieces/views.py の一部
def post(self, request, *args, **kwargs):
    obj = self.get_object()  # 削除対象を取得
    messages.success(request, f"{obj.id}. {obj.name}を削除しました。")
    return super().post(request, *args, **kwargs)
```

🟦 今回、**どうやって削除メッセージを表示させたのか**を、ちょっとだけ説明しておくね。

削除ボタンを押すと、DeleteView は対象オブジェクト（データ）の削除処理に入るんだけど、その前に def post() という関数をオーバーライドして、通知メッセージを渡す処理を入れたの。

ちょっと何言ってるか分からないよね。

まず、ビューの処理が走るときって、データの送信には全て POST メソッドを使用しているの。
コードを見返してもらうと分かると思うけど、どのフォームにも {% csrf_token %} が付いてるでしょ？
あれは、POSTメソッドでデータを送りますよ」という Django の合図。

そして、Django のビューは**フォームが送信されたら、まず post() を実行する**の。だから、その post() 実行のときに **削除の直前に messages 通知を仕込む** 処理を追加することにしたんだ。

あとは、テンプレート側で {% if messages %} … {% endif %} の範囲を書いてあげれば、ちゃんと通知が表示されるので、list.html に表示コードを入れ込んだのでした🌿

:::message
 🫛 豆知識
本番では「データを削除せずに見えなくする」論理削除という手法も使っているよ。
データベース上からデータは消さずに、データの有効/無効をフラグで管理するやり方なんだけど・・・実際は、論理削除を選択することが多いんじゃないかな🧐<br>
企業が会員データや決済データなどを持つ場合、そのデータを完全に削除してしまうと、後から問い合わせがあったり、金融監査があったりしたときに、何も答えられなくなってしまうよね。
そうなると、企業経営の論拠が示せない事態に陥ってしまうので、データを根こそぎ削除することはオススメできない。<br>
とはいえ、CRUD アプリを作るときに DeleteView は絶対に必要な経験なので、今回は「論理削除」ではなく「物理削除」にトライしてもらったよ🌻
:::

## 10. ギャラリーフィナーレ
最後は、ギャラリーページを作成して、この章のフィナーレを飾る！！
使用するビューはListView。一気にコーディングしていってくれ！

♦️ gallery.html を作成してコード記入
♦️ views.py にギャラリー用のクラスを追加
♦️ urls.py にルーティング
♦️ index.html にリンクを追加


1. gallery.html を作成してコード記入
```django
<!-- sg_pieces/templates/sg_pieces/gallery.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー{% endblock %}
{% block content %}

  <h1 class="text-center mb-4 mt-4">✨ ギャラリー ✨</h1>
  <div class="container mt-4">
    <div class="row row-cols-1 row-cols-md-3 g-4">
      {% for piece in pieces %}
        <div class="col">
          <div class="card h-100 shadow-sm">
            {% if piece.image %}
              <div class="d-flex align-items-center justify-content-center" style="height: 250px; overflow: hidden;">
                <img src="{{ piece.image.url }}" alt="{{ piece.name }}"
                     class="img-fluid"
                     style="max-height: 100%; max-width: 100%; object-fit: contain;">
              </div>

            {% else %}
              <div class="card-img-top bg-light d-flex align-items-center justify-content-center"
                   style="height: 250px;">
                <span class="text-muted">画像未登録</span>
              </div>
            {% endif %}
            <div class="card-body">
              <h5 class="card-title">{{ piece.name }}</h5>
              <p class="card-text"><strong>登録日：</strong>{{ piece.created_at|date:"Y-m-d H:i" }}</p>
              <p class="card-text">{{ piece.memo|linebreaksbr|truncatechars:120 }}</p>
            </div>
          </div>
        </div>
      {% empty %}
        <div class="col">
          <div class="alert alert-secondary w-100 text-center">まだ作品が登録されていません😢</div>
        </div>
      {% endfor %}
    </div>
  </div>

  <!-- ページネーション -->
  {% if is_paginated %}
    <nav class="mt-4 d-flex justify-content-center">
      <ul class="pagination">
        {% if page_obj.has_previous %}
          <li class="page-item">
            <a class="page-link" href="?page={{ page_obj.previous_page_number }}">&laquo;</a>
          </li>
        {% else %}
          <li class="page-item disabled">
            <span class="page-link">&laquo;</span>
          </li>
        {% endif %}

        {% for num in page_obj.paginator.page_range %}
          {% if page_obj.number == num %}
            <li class="page-item active"><span class="page-link">{{ num }}</span></li>
          {% else %}
            <li class="page-item"><a class="page-link" href="?page={{ num }}">{{ num }}</a></li>
          {% endif %}
        {% endfor %}

        {% if page_obj.has_next %}
          <li class="page-item">
            <a class="page-link" href="?page={{ page_obj.next_page_number }}">&raquo;</a>
          </li>
        {% else %}
          <li class="page-item disabled">
            <span class="page-link">&raquo;</span>
          </li>
        {% endif %}
      </ul>
    </nav>
  {% endif %}

{% endblock %}
```

2. views.py にギャラリー用のクラスを追加
```python
# sg_pieces/views.py
class GalleryPieceView(ListView):
    model = GalleryPiece
    template_name = "sg_pieces/gallery.html"
    context_object_name = "pieces"
    paginate_by = 3
```

3. urls.py にルーティングを追加
```python
# sg_pieces/urls.py の追加箇所のみ
    path('gallery/', views.GalleryPieceView.as_view(), name='piece_gallery'),   # 🌟 追加
```

4. index.html にリンクを追加

```django
<!-- sg_pieces/templates/sg_pieces/index.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー TOP{% endblock %}
{% block content %}

      <h1 class="text-center mb-4 mt-4">秘密のプライベートギャラリー</h1>
      <div class="list-group d-grid gap-3 mx-auto text-center" style="max-width: 300px;">
        <a href="{% url 'piece_create' %}" class="list-group-item list-group-item-action">作品を登録する</a>
        <a href="{% url 'piece_list' %}" class="list-group-item list-group-item-action">作品を確認する</a>
        <a href="{% url 'piece_gallery' %}" class="list-group-item list-group-item-action">ギャラリー</a>
      </div>

{% endblock %}
```

ここまで設定して、ブラウザをリロードすると・・・

出たーーー！！
![](/images/c4_p9_20_gallery.png)

まごうことなき、✨ ギャラリー ✨だね！！！
（絵については、何も言わないでください。。コーディング考えるだけで精一杯だったよ笑）

:::message
**ページネーションの使い方講座**
is_paginated は、「ページ分けされているかどうか」をテンプレート側で判定するための ListView のクラス変数。
ビューに paginate_by = 分割数 を設置するだけで、Django が自動的にページネーションの仕組みを用意してくれるよ。
そして、テンプレートに表示されるデータ数が複数ページに分かれる場合だけ、is_paginated が True になる。<br>
🟦 テンプレートは、最初に is_paginated を使って、**「ページネーションのナビゲーション（前へ・次へ）を表示するかどうか」**を判断する。
これが、ページネーションの基本。
```django
{% if is_paginated %}
  ... ページナビゲーション表示 ...
{% endif %}
```

🟦 ナビゲーション内では page_obj という変数が使えるようになっているよ。page_obj には、現在のページに関する情報が詰まっているの。
- page_obj.has_previous：前のページがあるか？
- page_obj.has_next：次のページがあるか？
- page_obj.paginator.page_range：ページ番号の一覧（すべてのページ数ぶん。たとえば [1, 2, 3, 4] みたいな全ページの数字一覧）
- page_obj.number：いま表示中のページ番号（1ページ目なら 1）

これらは、paginate_by を使ってビュー（ views.py ）でページネーションを有効にしていれば、テンプレートでそのまま使える。

> **⚠️ 少し注意が必要な書き方**
> ♦︎♦︎ **page_obj.previous_page_number と page_obj.next_page_number** ♦︎♦︎
> ♦️ .previous_page_number は .has_previous = True のときに呼ばないとエラーになる。
> ♦️ .next_page_number は .has_next = True のときに呼ばないとエラーになる。
> - だからテンプレートに書くときには、.previous_page_number / .next_page_number = True 判定の中で使用する必要がある。

🟦 それらを踏まえて、テンプレート内でページネーションを実装すると、下記のようになるよ。
```django
{% if is_paginated %}

  {% if page_obj.has_previous %}
    <a href="?page={{ page_obj.previous_page_number }}">前へ</a>
  {% endif %}

    {% for num in page_obj.paginator.page_range %}
      {% if page_obj.number == num %}
        <span>{{ num }}</span>  ← ここが現在のページ
      {% else %}
        <a href="?page={{ num }}">{{ num }}</a>
      {% endif %}
    {% endfor %}

  {% if page_obj.has_next %}
    <a href="?page={{ page_obj.next_page_number }}">次へ</a>
  {% endif %}

{% endif %}

```

🟦 実装コードの中に HTMLの特殊文字（エンティティ）の「 `&laquo;` 」があったよね。
Django のページネーションでよく使うエンティティがあるから、置いておくね🎋

| エンティティ    | 表示 | 用途               |
| --------- | -- | ---------------- |
| `&laquo;` | «  | 最初のページへ（または「前へ」） |
| `&raquo;` | »  | 最後のページへ（または「次へ」） |
| `&lt;`    | <  | 前へ（代替）           |
| `&gt;`    | >  | 次へ（代替）           |

:::


そして、実は、class GalleryPieceListView(ListView): には、すでに paginate_by = 10 を設定していた！「あとで実装する」とだけ書いて放置していたの、誰か気づいていた？？

だから最後に list.html にも、同じページネーションを追加しておこうね。

下記は list.html の完成コード。
ページネーションは、「リスト一覧」と「トップに戻る」の間に入れてみたよ。
```django
<!-- sg_pieces/templates/sg_pieces/list.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー List{% endblock %}
{% block content %}

  <h1 class="text-center mb-4 mt-4">秘密のプライベートギャラリー</h1>
    <ul class="list-group list-group-flush mx-auto mt-4" style="max-width: 400px;">
      {% for piece in pieces %}
        <li class="list-group-item">
          {{ piece.id }} {{ piece.name }}
        <span class="float-end">
          <!-- 画像のサムネイル表示 -->
          {% if piece.image %}
          <img src="{{ piece.image.url }}" alt="{{ piece.name }}" style="height:40px; width:40px; object-fit:cover; border-radius:6px;">
          {% endif %}
          <!-- リンク表示 -->
          <a href="{% url 'piece_detail' piece.pk %}" class="btn btn-outline-info btn-sm">詳細</a>
          <a href="{% url 'piece_update' piece.pk %}" class="btn btn-outline-secondary btn-sm">編集</a>
          <a href="{% url 'piece_delete' piece.pk %}" class="btn btn-outline-danger btn-sm">削除</a>
        </span>
        </li>

      {% empty %}
        <li class="list-group-item">まだ作品が登録されていません😢</li>
      {% endfor %}

      {% if messages %}
        {% for msg in messages %}
          <div class="alert alert-{{ msg.tags|default:'info' }} alert-dismissible fade show" role="alert">
            {{ msg }}
            <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
          </div>
        {% endfor %}
      {% endif %}


    <!-- ページネーション -->
    {% if is_paginated %}
      <nav class="mt-4 d-flex justify-content-center">
        <ul class="pagination">
          {% if page_obj.has_previous %}
            <li class="page-item">
              <a class="page-link" href="?page={{ page_obj.previous_page_number }}">&laquo;</a>
            </li>
          {% else %}
            <li class="page-item disabled">
              <span class="page-link">&laquo;</span>
            </li>
          {% endif %}

          {% for num in page_obj.paginator.page_range %}
            {% if page_obj.number == num %}
              <li class="page-item active"><span class="page-link">{{ num }}</span></li>
            {% else %}
              <li class="page-item"><a class="page-link" href="?page={{ num }}">{{ num }}</a></li>
            {% endif %}
          {% endfor %}

          {% if page_obj.has_next %}
            <li class="page-item">
              <a class="page-link" href="?page={{ page_obj.next_page_number }}">&raquo;</a>
            </li>
          {% else %}
            <li class="page-item disabled">
              <span class="page-link">&raquo;</span>
            </li>
          {% endif %}
        </ul>
      </nav>
    {% endif %}

    <a href="{% url 'index' %}" class="mt-4 mx-auto btn btn-outline-primary" style="width:130px;">トップに戻る</a>
    </ul>
    
    <!-- Bootstrapの閉じるボタンを効かせる用 -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.7/dist/js/bootstrap.bundle.min.js"></script>

{% endblock %}
```

これでギャラリーの骨格が完成！！！
え？質素、だって？
うん、そうだんだけどさ・・・

装飾はお任せ致します🙇だわ笑


## 📕 model と form と widgets と。
ぷに蔵が最初「え？？？」と思って、どうにもこうにも「なに言ってんの？」と思った思い出ポイントが、今回の章に出てきていたので、過去の与太話を紹介しようと思う。

🟣 ここから先は Django と格闘したぷに蔵の与太話です 

なので、読まなくていいと思う。
でも、いつか同じ罠にハマった人のために説明しておきたい笑

```python
widgets = {
  ...
  "image": forms.ClearableFileInput(attrs={"class": "form-control"}),
}
```
モデルフォーム内のウィジェット設定ね。（ここでのウィジェットは、フォームの見た目と入力UIに使うもの）
これ見たとき、本当に意味不明だった。
だってね、models.py では image フィールドは models.ImageField で作ってたのよ。
```python
# models.py
class GalleryPiece(models.Model):
    name = models.CharField(max_length=100)  
    created_at = models.DateTimeField(null=True, blank=True)
    memo = models.TextField()
    image = models.ImageField(upload_to='images/', blank=True, null=True)
```

でも、forms.py の ModelForm の widgets では、こう書かれる。
```python
# forms.py
class GalleryPieceForm(forms.ModelForm):
    class Meta:
        ...
        widgets = {
            "name": forms.TextInput(attrs={"class": "form-control", "placeholder": "名前"}),
            "created_at": forms.DateTimeInput(
                attrs={"type": "datetime-local", "class": "form-control"},
                format="%Y-%m-%dT%H:%M",
            ),
            "memo": forms.Textarea(attrs={"class": "form-control", "rows": 5}),
            "image": forms.ClearableFileInput(attrs={"class": "form-control"}),
        }
```

**この差が、混乱。**
🔹 **image = models.ImageField()**
♦️ **"image": forms.ClearableFileInput()**

いまなら、**こっちの２つだって違うだろ‼️**ってツッコめる。
🔹 **name = models.CharField()**
♦️ **"name": forms.TextInput()**

でもText と Char は、なんか同じ部類じゃん？という意味のわからない思い込みが、自分の中にあったんだと思う。。。

Django にちょっと慣れてきた頃のぷに蔵の頭の中には、
「**models.py で定義したモデルから ModelForm で自動的にフォームを作成してるんだから、データ型だって同じに指定したら良いんでしょ**」という、謎理論が生まれていた。
それこそが間違いだった。
「半分間違い」とか言いたいけど、概ね間違えている笑

そもそも、モデル / フォーム / ウィジェット は、すべてを分けて考えるべきだった。
あくまで、Django が裏で結びつけているだけで、それぞれは独立した機能ということを意識しておかないといけなかったの。
そこを抑えておかなかったから、widgets = {}内に「 "image" = forms.ImageField(…) 」とか書き続けて、永遠にattrsエラー（**'ImageField' object has no attribute 'attrs'**）喰らってた笑 

💠 こちらの表をご覧いただきたい。
| レイヤー | 指定 | 役割 | attrs |
|--------------|------|--------------|---------|
| モデル | models.ImageField() | DB定義（テーブル列の性質） | ない |
| フォーム | forms.ImageField() | フォーム入力のバリデーション等 | ない |
| ウィジェット | forms.ClearableFileInput() | HTMLの表示担当 | ある（class追加とか） |

**forms.ImageField(…) とは、フォームに新しいフィールド追加のときに使うもの**だから、そもそも widgets では使わない。
「なに言ってるか分かんない……」ってなるよね。
これね、色々作っていく過程で「は！そういうことか！」ってなると思う。

もう少し噛み砕いてみる！！

まず、入力フォームを作るときには、２種類の作り方があるのよ。

1. モデルに定義されているフィールドを、Django に自動でフォームに反映してもらうやり方
（forms.ModelForm を継承する。モデル定義を使うから、モデルの指定が必要）
```python
class GalleryPieceForm(forms.ModelForm):
  class Meta:
    model = GalleryPiece
```
2. フォームを１から自作するやり方
（forms.Form を継承する。モデル定義は使わないから、テーブルに保存するフィールドも自分で書いていく。CBV でやるということ。操作対象のモデルは、ハードコーディング内で指定していく）
```python
class UploadForm(forms.Form):
    name = forms.CharField(max_length=100)
    image = forms.ImageField(required=True)
```
「forms.ImageField(…)」という指定は、フォーム作成のフィールドタイプとして準備されている。
（だから、widgets に書いたときにも補完が効いちゃってミスに気づくの遅くなったんだよね〜...という言い訳ｗ）

前出💠表の「フォームレイヤー」というのは、あくまで、自作フォームの話。
モデルフォームの場合にはフィールド自動追加がされるからね。

そして、「ウィジェットレイヤー」の forms.ClearableFileInput() というのは、**モデルの FileField / ImageField の定義箇所にデフォルトで割り当てられるウィジェットのこと。
だから、**モデル定義で models.ImageField() を設定したフィールドに対して、フォームにおけるウィジェット機能を使用したいなら「 forms.ClearableFileInput() 」を使用することが決まり**。

そんなことを理解するまで、時間が掛かりました、というお話でした。


