---
title: "第５章　ユーザー認証の彼方へ"
emoji: "♎️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["django", "python"]
published: true
published_at: 2050-08-20 00:00 # 未来の日時を指定する
---
## 01. 秘密のための　ログイン / ログアウト機能
「秘密のプライベートギャラリー」シリーズの最後は、ログイン機能の実装で締めよう！！

これまで結構大変だったから、わりと楽かも？

実装ステップとしては、
♦️ login.html の作成＆設置
♦️ ビューに「ログイン中じゃないと表示させない」クラス（ LoginRequiredMixin ）の継承を追加
♦️ プロジェクト urls.py にログイン機能のルーティング追加
♦️ settings.py のログイン廻りの設定
・・・といったところかな。

1. **login.html の作成**
まずは、作成するフォルダなんだけど、いままでの アプリ内 templates ではなく、さらにその直下に「 registration 」フォルダを作って、その中に login.html を置いてもらう必要がある。
つまり、構造としては、templates/registration となるね。そして、そのフォルダの中に login.html を作成するよ！
```html
<!-- sg_pieces/templates/registration/login.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー{% endblock %}
{% block content %}

<h1 class="container text-center mb-4 mt-4">秘密のプライベートギャラリー</h1>
  <div class="container mt-3" style="max-width: 400px;">
    <h3 class="text-center mb-4"> 🔐 ログイン 🔐</h3>
    <form method="post" class="text-center mb-4">
      {% csrf_token %}
      {{ form.as_p }}
      <button type="submit" class="btn btn-outline-info w-50 mt-4">ログイン</button>
    </form>
</div>

{% endblock %}
```

::::details {{ form.as_p }}を使わないログインフォーム
{{ form.as_p }}を使用すると、すべてのフォームフィールドが一気に並んでしまう！
だから、個々のフィールド UI を CSS でカスタムしたい人には不向きなんだよね。
その場合は、こんな感じで、フォームをバラすことも可能！
```html
<!-- sg_pieces/templates/registration/login.html -->
{% extends 'sg_pieces/base.html' %}
{% block title %}秘密のプライベートギャラリー{% endblock %}
{% block content %}

<h1 class="container text-center mb-4 mt-4">秘密のプライベートギャラリー</h1>
  <div class="container mt-3" style="max-width: 400px;">
    <h3 class="text-center mb-4"> 🔐 ログイン 🔐</h3>
    <form method="post" class="text-center mb-4">
      {% csrf_token %}

      <div class="mb-3">
        <label for="{{ form.username.id_for_label }}" class="form-label">ユーザー名</label>
        {{ form.username }}
        {{ form.username.errors }}
      </div>

      <div class="mb-3">
        <label for="{{ form.password.id_for_label }}" class="form-label">パスワード</label>
        {{ form.password }}
        {{ form.password.errors }}
      </div>

      <button type="submit" class="btn btn-outline-info w-50 mt-4">ログイン</button>
    </form>
</div>

{% endblock %}
```
自分の作り方に合う方を選んでね🌻
::::

2. **ビューに「ログイン中じゃないと表示させない」クラスの継承を追加**
Django では、ログイン認証の便利機能として LoginRequiredMixin というクラスが用意されているよ。
これを継承して CBV でクラスを作成すれば、そのクラスを読み込むためには、ログイン認証が必須になる！

説明よりも、実際のコードを見た方が理解できると思う！
```python
# sg_pieces/views.py
from django.contrib.auth.mixins import LoginRequiredMixin
from django.contrib.auth.decorators import login_required
from django.shortcuts import render
from django.urls import reverse_lazy
from django.views.generic import CreateView, ListView, DetailView, UpdateView, DeleteView
from django.contrib import messages
from .models import GalleryPiece
from .forms import GalleryPieceForm

@login_required
def index(request):
    return render(request, "sg_pieces/index.html")

class GalleryPieceCreateView(LoginRequiredMixin, CreateView):
    model = GalleryPiece
    template_name = "sg_pieces/piece_form.html"
    form_class = GalleryPieceForm

class GalleryPieceUpdateView(LoginRequiredMixin, UpdateView):
    model = GalleryPiece
    template_name = "sg_pieces/piece_form.html"
    form_class = GalleryPieceForm

class GalleryPieceListView(LoginRequiredMixin, ListView):
    model = GalleryPiece
    template_name = "sg_pieces/list.html"
    context_object_name = "pieces"
    ordering = ["id"]
    paginate_by = 10

class GalleryPieceDetailView(LoginRequiredMixin, DetailView):
    model = GalleryPiece
    template_name = "sg_pieces/detail.html"

class GalleryPieceDeleteView(LoginRequiredMixin, DeleteView):
    model = GalleryPiece
    template_name = "sg_pieces/delete.html"
    success_url = reverse_lazy("piece_list")

    def post(self, request, *args, **kwargs):
        obj = self.get_object()  # 削除対象を取得
        messages.success(request, f"{obj.id}. {obj.name}を削除しました。")
        return super().post(request, *args, **kwargs)

class GalleryPieceView(LoginRequiredMixin, ListView):
    model = GalleryPiece
    template_name = "sg_pieces/gallery.html"
    context_object_name = "pieces"
    paginate_by = 3
```

全部のビューの左側に「 LoginRequiredMixin 」をくっつけるだけ！
ただし、クラスの継承は左側から順番に行われるから、必ず LoginRequiredMixin はビューの左側に書くことが大事だよ⭐︎

そして、もうひとつ注目ポイントがあるのだよ。
```python
from django.contrib.auth.decorators import login_required

@login_required
def index(request):
    return render(request, "sg_pieces/index.html")
```

- @login_required
これは **デコレータ** と呼ばれるもので、関数ビューにくっつけて使うもの。
request を引数にとる関数ビュー（FBV）の前に付けるだけで、「ログイン必須ビュー」になるよ！
ただくっつけるだけで良いなんて、うひょ〜〜〜便利！

でも、そんな便利なものなら、CBV のビューにもくっつけて、
```python
@login_required
class GalleryPieceView(ListView):
    model = GalleryPiece
    template_name = "sg_pieces/gallery.html"
```
・・・みたいにしちゃえば？って思ったりする？


でもね、デコレータは、Django のクラスビュー（CBV）にはくっつけられない。
その理由は、クラスメソッドと関数ビューの変換とか、Django の処理手順の話も関係してくるから、いまは言及するタイミングじゃないかな。
だから、下記のように覚えてしまえばOK！

- クラスビューでログイン必須にしたいときは「 LoginRequiredMixin 」を継承する
- 関数ビューでログイン必須にしたいときは、「 @login_required 」デコレータを頭につける

これで完璧🌻

::: message alert
🚨 **重要ポイント** 🚨
ログイン不要でアクセス許可する仕様のトップページ（index）やログインページ自体には LoginRequiredMixin を付けないこと！
もし付けると、無限リダイレクト地獄になるから。
Django と、
「ここはログインが必要だ、ゴラァ🤛」
「でもログインするためのフォームは login.html に表示されるはずなんです💦」
「知るか！ログイン済じゃないと表示させるなって言われてるんだ、こっちは‼️」
「でも、そのためのフォームを表示してくれないと😩」
・・・という、無限リダイレクトの乱が勃発します。

わりと最初の頃にやりがち。（経験談）

お気をつけてね笑
※ 今回のギャラリーの index は、ログイン後の表示を想定しているから、LoginRequiredMixin 付けて大丈夫！

ちなみに、管理画面（/admin/）には標準でログイン必須がかかっているので安心よ。
:::

::::details def index() を CBV で書き直したい場合はこちら

いま、views.py は index 以外、すべて CBV で書いているよね。
これも統一感を持たせて CBV で書きたい場合は、views.py と urls.py を書きかえればいい。
元の FBV のところはコメントアウトで残してあるから、お好みで好きな方を有効にして使ってみてね。

```python
# sg_pieces/views.py 書きかえ箇所のみ
# @login_required
# def index(request):
#     return render(request, "sg_pieces/index.html")

class GalleryIndexView(LoginRequiredMixin,TemplateView):
    template_name = 'sg_pieces/index.html'
```

```python
# sg_pieces/urls.py 書きかえ箇所のみ
    # path('', views.index, name='index'),
    path('', views.GalleryIndexView.as_view(), name='index'),
```
::::


3. **プロジェクト urls.py にログイン機能のルーティング追加**
```python
# プロジェクトの urls.py。アプリ内の urls.py じゃないことに注意！
from django.contrib import admin
from django.urls import path, include
from django.contrib.auth import views as auth_views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('sg_pieces.urls')),
    path('login/', auth_views.LoginView.as_view(template_name='registration/login.html'), name='login'),
    path('logout/', auth_views.LogoutView.as_view(), name='logout'),
]
```

:::message
**login / logout 処理**
- auth_views.LoginView は Django が用意してくれているログインビュー
- template_name を指定すると、自作の login.html が使えるようになる
- auth_views.LogoutView は ログアウト処理をやってくれる
:::


4. **settings.py にログイン回りの設定**
```python
# settings.py
LOGIN_URL = "/login/"            # 未ログインでアクセスしたときに飛ばされる先
LOGIN_REDIRECT_URL = "/"         # ログイン成功後に飛ぶ先
LOGOUT_REDIRECT_URL = "/login/"  # ログアウト後に飛ぶ先
```

:::message
**login / logout 処理後のジャンプ先**
LOGIN_URL：LoginRequiredMixin や @login_required が「ログインして！」ってなった時の飛び先。
LOGIN_REDIRECT_URL：ログインが成功したらここに戻る。
LOGOUT_REDIRECT_URL：ログアウトが終わったらここに戻る。
:::

## 02. ログイン中画面の表示はどうする？
機能としては、実装が完了したね。

ここでは、現在ログイン中のとき、どのように画面上に表示するのかを設定していくよ！

まずは突然ですが、base.html を下記に書きかえていただけるかな？
```html
<!-- sg_pieces/templates/sg_pieces/base.html -->
{% load static %}
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{% block title %}PRIVATE GALLERY{% endblock %}</title>
    <link rel="stylesheet" type="text/css" href="{% static 'sg_pieces/css/style.css' %}">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.7/dist/css/bootstrap.min.css" rel="stylesheet">
  </head>
  <body class="bg-light">
    <nav class="navbar navbar-light bg-light justify-content-end px-3">
      {% if user.is_authenticated %}
        <span class="navbar-text me-3">ログイン中：{{ user.username }}</span>
        <span class="navbar-text me-3"><a href="{% url 'index' %}">TOP</a></span>
        <span class="navbar-text me-3"><a href="{% url 'admin:index' %}">管理画面</a></span>
        <form method="post" action="{% url 'logout' %}">
          {% csrf_token %}
          <button type="submit" class="btn btn-sm btn-outline-secondary">ログアウト</button>
        </form>
      {% endif %}
    </nav>

    <main class="container py-1">
      {% block content %}
      <!-- ページごとの中身がここに入る -->
      {% endblock %}
    </main>
  </body>
</html>
```

ここで出てきた **user** は、Django がテンプレートに自動で渡してくれるオブジェクト！
最初に作った sg_user（CustomUser）のユーザーがログイン中なら、ここにデータが入るよ。

- {% if user.is_authenticated %} … {% endif %}
ログイン中なら True、ログアウト済なら False。
これで「ログインしている人にだけ見せるナビバー」を作ったよ。

- {{ user.username }} 
user オブジェクトの中から username を取り出して表示。
自分の名前が表示されると「ログインした」感が出るし、分かりやすいよね。

- ログアウト部分だけは **<form> を使って POST** しているよ。
これは Django が **LogoutView は安全のために POST で送信**を推奨しているから。  
{% csrf_token %} を使えば、CSRF（クロスサイトリクエストフォージェリ）対策ができる。だから、特別な仕様の理由がない限り、公式推奨に従うのがベターかな。


> 下記の２つに関しては、次のパートでやるから、とりあえずコードの記入だけしておいてね！
> ♦️ {% load static %}
> ♦️ <link rel="stylesheet" type="text/css" href="{% static 'sg_pieces/css/style.css' %}">


## 03. まさかの最後に static

ログイン機能の実装までを目標に始めたこのシリーズ。
実は、まだ説明していなかった大事なテーマがあるんだよね。

その名も、**static ファイル**！！！

> 🟦  **「え？static って、すでに urls.py で扱ってなかったっけ？」**

そう思った人。
記憶力良すぎて心配になる笑


たしかに、urls.py 内に書いた！
アップロードした画像（media/）を配信をするための設定だったよね。
```python
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```
開発中だけ Django が直で MEDIAファイル（ユーザーがアップロードする画像とか）を配信してくれる便利機能を備えた関数のことでした。


でも、今回取り上げるのは **staticファイルのこと**で、ちょっと違う。

プロジェクトに最初から用意しておく CSS / JavaScript / アイコン / 画像 のこと。
ユーザーがアップロードするんじゃなくて、開発者が Webアプリケーションで使うために「あらかじめ置いておく固定ファイル」のことだから、静的ファイル配信関数の static() とは違うの！

そう。ここは、わりと混乱するポイントだから、おさえて欲しい！

**🟦 「static() 関数」と「static フォルダ」は別モノだということを！！**
- static() 関数（urls.pyで使うやつ）は？
Django が「開発モード（DEBUG=True）のときだけ、指定したフォルダをWeb経由で配信するよ〜」という便利関数。
通常は、下記コードのように、urlpatterns += static() のように使うよ。
```python
from django.conf import settings
from django.conf.urls.static import static

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

- 「 staticファイル 」は？
CSS / JSファイル / アイコン画像などの、開発者があらかじめ置いておくファイルの総称。
CSS や JS を static フォルダに置いたときに、それらをまとめて **staticファイル** と呼ぶの。
こちらは、**settings.py にルート/URL 設定をして、{% load static %} というテンプレートタグとセットで使用する**。

> static() 関数 = Django の便利機能（配信用の関数）
> static フォルダ = ファイルの置き場所（CSS/JSなど固定資産）

名前は同じでも、役割は全然違うことがお分かりいただけたかな？
Django の公式が「static file = 静的ファイル」って呼んでるから、そこに関数もフォルダも乗っかっちゃってるだけ。雑。混乱する笑


ということで、static フォルダを設置していくよ！
場所は、sg_pieces アプリ内。
下記の構造で、style.css ファイルまで作成してくれ！
sg_pieces/static/sg_pieces/css/style.css

:::message
Djangoの推奨ルールとして、アプリ名/static/アプリ名/ の中に置くのがベスト。
こうしておくと、複数アプリを持ったときに static ファイルが衝突しない！
これは、templates/アプリ名/ のときと同じ考え方だから、一貫した Django の推奨ルールだよね。
:::

そして、設置場所、分かったかな？
一応、現在のディレクトリ構造を確認してみるね。
```
secret_gallery
├── db.sqlite3
├── manage.py
├── media
│   └── images
│       ├── image_ahiru_xFhdZO7.png
│       ├── image_ankho_CsFwWQr.png
│       └── image_mendako_ZZmYmNe.png
├── secret_gallery
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── sg_pieces
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── forms.py
│   ├── models.py
│   ├── static
│   │   └── sg_pieces
│   │       └── css
│   │           └── style.css
│   ├── templates
│   │   ├── registration
│   │   │   └── login.html
│   │   └── sg_pieces
│   │       ├── base.html
│   │       ├── delete.html
│   │       ├── detail.html
│   │       ├── gallery.html
│   │       ├── index.html
│   │       ├── list.html
│   │       └── piece_form.html
│   ├── tests.py
│   ├── urls.py
│   └── views.py
└── sg_user
    ├── __init__.py
    ├── admin.py
    ├── apps.py
    ├── models.py
    ├── tests.py
    └── views.py
```

**sg_pieces/static/sg_pieces/css/style.css** は見つけられた？
その位置に、ファイルを作ってほしい。

そして、style.css ファイルの中身はこちら。
```css
.navbar a{
  font-size: 14px;
}

.navbar a {
  text-decoration: none;
  border: none;
  background: none;
  outline: none;
}

/* ホバー時に点線の下線 */
.navbar a:hover {
  text-decoration: underline dotted;
}

.navbar link:hover {
  border-bottom: 1px dotted #0d6efd; /* Bootstrap の secondary カラーに合わせた */
  cursor: pointer;
}
```
最低限の css だけど、ログイン中の表示周りを少し整えているよ！
ここに css を自由に書いて、画面をカスタマイズしてみるのも良いね😊


次は、settings.py の設定に取りかかろう・・・と言いたいところだが、開発環境において settings.py に static 設定は、特に必要ない！
なぜなら、デフォルトで書かれた下記の記述で十分だから！
```python
STATIC_URL = "static/"
```

base.html に、まだ触れていなかったコードがあったよね。
> ♦️ {% load static %}
> ♦️ <link rel="stylesheet" type="text/css" href="{% static 'sg_pieces/css/style.css' %}">

- {% load static %}
これは、static ファイルを読み込むためのテンプレートタグ。
- <link rel="stylesheet" type="text/css" href="{% static 'sg_pieces/css/style.css' %}">
{% static 'sg_pieces/css/style.css' %} は staticファイルのURLを自動で組み立ててくれるタグ。
STATIC_URL の設定をもとに、正しいパスに変換してくれるよ。

例えば、STATIC_URL = "static/" の場合、
{% static 'sg_pieces/css/style.css' %} → /static/sg_pieces/css/style.css
というURLに変換される。

> 🔷 **どうしてわざわざテンプレートタグを使うの？**
> 開発環境では http://127.0.0.1:8000/static/... で staticファイルが配信される。
> 
> だけど、本番環境だと CDN や別サーバーから配信する場合もあるんだ。
> そんなときでも Django が STATIC_URL に応じて自動で正しいURLに変換してくれるから、テンプレートの修正は一切不要！
> 
> これは、{% url 'ルート名' %} と同じ考え方だよ。
> URLパターンが変わっても name='xxx' が同じなら、テンプレートの修正は不要だったよね？
> それと同じで、{% static 'xxx' %} を使っていれば 「どこから配信されるか」（大元のURL） が変わっても、テンプレートはそのままで大丈夫なのよ。

::::details CDN が気になる人はこちら
CDN（Content Delivery Network）は、画像やCSS、JavaScriptみたいな「よく使うファイル」を、世界中のサーバーにコピーしておいて、ユーザーリクエストがあった場所に近いサーバーから配信してくれる仕組みのこと。<br>
東京の人がアクセスしたら東京のサーバーから、アメリカの人がアクセスしたらアメリカのサーバーから届けてくれる！
そのおかげで「どこからアクセスしても素早い配信」が実現されるの。
いまは使わないけど、いつかグローバル対応のアプリケーションを作るときには、かなりお世話になる仕組みだよ👍
::::


では、ここでブラウザをリロードして、index 画面を表示させてみようか！？


おそらく、static の効果が細かすぎて伝わらない笑
少し「 TOP 」「 管理画面 」の文字を小さくしただけだから！！

あとは、お好みに合わせて style.css に追加装飾してみてください🌻


でもさすがに、これだと色味が少なすぎるので、最後にホームページのヘッダーでも設置してみようかな。
適当な画像を保存して使ってみてくれて良いけど、準備するのが面倒な人は、ぷに蔵渾身のサイトヘッダーを保存して使ってね。

こちらを使う。
![](/images/title_logo.png)

この画像の保存場所は、こちら。
🔷 **sg_pieces/static/sg_pieces/title_logo.png**

静的ファイルなので、static の中にあることが大事！

それでは、これを index.html に設置しよう。
```html
{% extends 'sg_pieces/base.html' %}
{% load static %}
{% block title %}秘密のプライベートギャラリー TOP{% endblock %}
{% block content %}

  <div class="text-center">
    <img src="{% static 'sg_pieces/title_logo.png' %}" alt="秘密のプライベートギャラリー">
  </div>

  <div class="list-group d-grid gap-3 mx-auto text-center mt-3" style="max-width: 300px;">
        <a href="{% url 'piece_create' %}" class="list-group-item list-group-item-action">作品を登録する</a>
        <a href="{% url 'piece_list' %}" class="list-group-item list-group-item-action">作品を確認する</a>
        <a href="{% url 'piece_gallery' %}" class="list-group-item list-group-item-action">ギャラリー</a>
      </div>

{% endblock %}
```

ブラウザをリロードすると・・・どう！？
![](/images/c5_p3_21_logo.png)

・・・・はみ出してるわ笑

こんなときこそ css で調整しよう！！
style.css の一番上に、下記のコードを追加して！
```css
/* sg_pieces/static/sg_pieces/css/style.css の先頭に追加 */
.logo-img {
  max-width: 300px;
  height: auto;
}
```

そして、index.html の画像読み込み箇所を、下記に変更。
class の追加だよ！
```html
<!-- sg_pieces/templates/sg_pieces/index.html に一部追加のみ -->
  <div class="text-center">
    <img src="{% static 'sg_pieces/title_logo.png' %}" alt="秘密のプライベートギャラリー" class="logo-img img-fluid">
  </div>
```

これで読み込むと・・・・・

![](/images/c5_p3_22_logofit.png)

ちゃんとブラウザの枠内に収まった！！！
これで本当に完成！！！！！

## STATIC という魔術
:::message
**Django の静的ファイルって、わりと混乱しがち**
Django プロジェクトで静的ファイルの配信は、本番環境だと下記のように settings.py に設定する。
```python
STATIC_URL = "static/"

STATICFILES_DIRS = [
    BASE_DIR / "static",
]

STATIC_ROOT = BASE_DIR / "staticfiles"
```

ひとつずつ見ていくね。

- STATIC_URL
ブラウザからアクセスするときのURLの先頭部分。
例: STATIC_URL = "static/"
 → 実際のURLは http://127.0.0.1:8000/static/xxx

- STATICFILES_DIRS
開発モードで追加の静的ファイル置き場。
プロジェクト直下でまとめたいときに指定する。
例: BASE_DIR / "static" に置いたCSSやJSを読み込む場合。

- STATIC_ROOT
本番用に静的ファイルを1か所に集める場所。
python manage.py collectstatic を叩くとここに全部コピーされる。
本番サーバーはこのフォルダだけをNginxやCDNで配信する。
（開発モードでは不要）

:::
## 📕 Webアプリケーションできちゃった（現代社会へ降り立ったローカルDjango戦士）
