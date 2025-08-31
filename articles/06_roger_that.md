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
```django
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
```django
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
もし付けると、無限リダイレクト地獄になるから。<br>
Django と、
「ここはログインが必要だ、ゴラァ🤛」
「でもログインするためのフォームは login.html に表示されるはずなんです💦」
「知るか！ログイン済じゃないと表示させるなって言われてるんだ、こっちは‼️」
「でも、そのためのフォームを表示してくれないと😩」
・・・という、無限リダイレクトの乱を繰り広げることになります。<br>
わりと最初の頃にやりがち。（経験談）<br>
お気をつけてね笑

※ 今回のギャラリーの index は、ログイン後の表示を想定しているから、LoginRequiredMixin 付けて大丈夫！

ちなみに、管理画面（/admin/）には標準でログイン必須がかかっているので安心よ。
:::

::::details def index() を CBV で書き直したい場合はこちら

いま、views.py は index 以外、すべて CBV で書いているよね。
これも統一感を持たせて CBV で書きたい場合は、views.py と urls.py を書きかえればいい。
元の FBV のところもコメントアウトで残してあるから、お好みで好きな方を有効にして使ってみてね。

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
```django
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
Django が「開発モード（DEBUG=True）のときだけ、指定したフォルダをローカルサーバー経由で配信するよ〜」という便利関数。
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

上記のツリーから **sg_pieces/static/sg_pieces/css/style.css** は見つけられた？
その位置に、ファイルを作ってほしい。

そして、style.css ファイルの中身はこちら。
```css
.navbar a {
  font-size: 14px;
  text-decoration: none;
  border: none;
  background: none;
  outline: none;
}

.navbar a:hover {
  text-decoration: underline dotted;
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

:::message
🫛豆知識
static を使うためには、settings.py の INSTALLED_APPS 内に`django.contrib.staticfiles` が入っていないと使えないよ。
間違えて削っちゃった人がいたら、入れておいてね。
```python
INSTALLED_APPS = [
    ...
    "django.contrib.staticfiles",
    ...
]
```
:::

base.html に、まだ触れていなかったコードがあったね。
> ♦️ {% load static %}
> ♦️ <link rel="stylesheet" type="text/css" href="{% static 'sg_pieces/css/style.css' %}">

- {% load static %}
これは、static ファイルを読み込むためのテンプレートタグ。
- <link rel="stylesheet" type="text/css" href="{% static 'sg_pieces/css/style.css' %}">
{% static 'sg_pieces/css/style.css' %} は staticファイルのURLを自動で組み立ててくれるタグ。
STATIC_URL の設定をもとに、正しいパスに変換してくれるよ。
{% static %} を書くと、STATIC_URL の設定をもとに、Django が自動的にルートディレクトリやアプリ内の static ディレクトリを探してくれる。

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
```django
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
style.css の一番上にロゴに関するコードを追加するので、style.css の完成形はこれ！
```css
/* sg_pieces/static/sg_pieces/css/style.css */
.logo-img {
  max-width: 300px;
  height: auto;
}

.navbar a {
  font-size: 14px;
  text-decoration: none;
  border: none;
  background: none;
  outline: none;
}

.navbar a:hover {
  text-decoration: underline dotted;
  cursor: pointer;
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

## 04. テンプレートタグの {% extends %} と {% static %}

'static' フォルダ / ファイルについては理解してもらえた？
今回は、ローカル環境における STATIC 設定と、本番環境における STATIC 設定を理解する前提知識として、{% static %}テンプレートタグについて書いていくね。

**Django の静的ファイル 'static' って、わりと混乱しがち**

ローカル環境であれば、settings.py にこれだけ書いてあれば、静的ファイルが配信される。
```python
STATIC_URL = "static/"
```

ただし、必ずテンプレートファイルの読み込みだけは忘れちゃだめね。
これを最初に書く必要がある。
```django
{% load static %}
```


あれ？
でも、さっき書いた index.html では、{% load static %} の前に {% extends 'sg_pieces/base.html' %} があったよね。
これはどういうことだ？？

実はこの２つ、明確に読み込む順番が決まっている！
というか、**{% extends %} で読み込むのは base.html だけ！**
いっそ、そう覚えてしまってくれ！！

**🟦 {% extends %} は base.html 専用のテンプレートタグなの？**
**YESSSSSS！！！**
正確には、base.html で共通テンプレを作成しないことも可能だから、base.html 専用ではないんだけど笑
でも Django での共通テンプレートは一般的に base.html で作成するから、
**{% extends %} は base.html 専用のテンプレートタグ**て言い切っちゃう笑


そして、{% load static %} よりも先に {% extends %} を書く理由は、
🟢 **'static' よりも前に base.html を継承しておかないと、** 🟠
🟢 **テンプレートエンジンがブロック（{% block %}）の意味を解釈できないから** 🟠

index.html の最後の修正は、こんな最終形態コードになったよね。
```django
...
{% block content %}

  <div class="text-center">
    <img src="{% static 'sg_pieces/title_logo.png' %}" alt="秘密のプライベートギャラリー" class="logo-img img-fluid">
  </div>
...
{% endblock %}
```

{% block content %} 内で {% static %} を使っている。
ということは、先に {% block %} の位置をテンプレートタグが理解しておかないと、Django はテンプレートタグ迷子（エラー）になってしまうの。
だから、読み込み順番とても大事！！

```django
{% extends %}
{% load %}
```

:::message
「ということは、base.html の冒頭で {% load static %} してるから、index.html では {% load static %} だね？」
という疑問に、お答えします。

**Non！！！！！**

ここが、ぷに蔵的にも「なぜだ・・」ポイントなんだけど笑
{% load static %} は、テンプレートタグとして static を使いたいファイルで、必ず読み込まないとダメ！
なぜなら、Django の天才エンジニアたちがそういう風に作ったから！
理屈じゃない！！天才に従う以外の道はない！！<br>
でも、ま、多分、継承とか干渉とか色々あったんじゃない〜？（テキトー）<br>
そういうわけなので、base.html に {% load static %} が書いてあって、そのファイルを {% extends base.html %} しても、その先で {% static %} 使いたいなら、もう一度 {% load static %} を読み込まないとダメですよ〜〜〜。
:::


settings.py の STATIC 設定の話をしようと思ったが、だいぶズレてきたね。
でも、もう少しズレたまま、テンプレートタグの話をしておきたい。

{% extends %} が base.html 専用テンプレートタグだと言ったけど、もしかすると、他にも切り出して使いたい HTMLテンプレートファイルが、無いとは言えないよね。
たとえば、すっごく簡単だけど、こういうテンプレート。

例: _back_to_top.html
```html
<div>
  <a href="{% url 'index' %}" class="mt-3 btn btn-outline-primary" style="width:130px;">トップに戻る</a>
</div>
```

毎回「トップに戻る」を書くのは面倒。
そんなとき、パーツとして「 _back_to_top.html 」というテンプレートを作っておいて、それを読み込めば、メンテナンス性も上がって楽できる⭐︎

そんなときに使うテンプレートタグは **{% include %}** で、使い方も同じ。
```html
{% include '_back_to_top.html' %}
```

これで、パーツ読み込みが可能になる！
便利な小技なので、良かったら覚えてね🌻
※ テンプレートファイルの先頭につけたプレフィックスの「_(アンダースコア)」は、パーツを意味するよ（慣習的にね。絶対的な決まりではない）。


## 05. エラー出がちな STATIC 設定

STATIC ファイルを静的配信しようとしたとき、わりと
「static の設定が効かない！」
「style.css が反映されない」
「プロジェクト直下に cssファイル置きたい」
という声を聞くのね。


これは、ルート設定を理解すれば解決できる！
これが最後のパートだから、がんばろうね！！

まず、設定の仕方からいくね。
設定する項目は、最大で３つ。
1. settings.py の設定
```python
STATIC_URL = "static/"
STATICFILES_DIRS = [
    BASE_DIR / "static",
]
STATIC_ROOT = BASE_DIR / "staticfiles"
```

ひとつずつ見ていこうと思う。
１つ１つちゃんと役割があるから、しっかり理解しよう！

1. **STATIC_URL**

`STATIC_URL` は、{% load static %} テンプレートタグを使用するためには、必ず必要な設定。
```python
# そして、この設定があれば Django がアプリ直下の `static` ディレクトリを探しに行く
STATIC_URL = "static/"
```
※ **{% load static %} が 、どこの static ディレクトリを探すのか** を理解することが重要。
これを理解していないと、下記のようにルート設定する意味が理解できないの。
```django
{% load static%}
<link rel="stylesheet" type="text/css" href="{% static 'sg_pieces/css/style.css' %}">
```
> secret_gallery/sg_pieces/static/sg_pieces/css/style.css ← これが cssのルート。
> secret_gallery/ は「プロジェクト」
> sg_pieces/ は「 secret_gallery プロジェクト内のアプリ」
> - **STATIC_URL = "static/"**
> この設定をしていれば、Django は **sg_pieces アプリ内の static ディレクトリまで**を探す。
> (secret_gallery/sg_pieces/static/ ← ここまでが検索対象で {% load static %} で届く場所)
>
> だから、「 href="{% static 'sg_pieces/css/style.css' %} 」というルート指定をする。
> しつこく言っているけど、ここを理解していれば、これからの設定を迷わない最重要ポイントだから、何度も言わせて！

2. **STATICFILES_DIRS**

次に、STATICFILES_DIRS だけど、これは追加の探索ディレクトリ。
つまり、**アプリ配下以外に static フォルダを置きたい場合に設定する箇所**。
逆をいえば、すべての static フォルダがアプリ内に設置してあるなら、この設定は必要ない。
```python
# アプリ以外の場所に staticフォルダ置きたいなら、この設定が必須
STATICFILES_DIRS = [
    BASE_DIR / "static",
]
```

settings.py の 「STATICFILES_DIRS 設定の下」くらいに、こんなコードを入れ込んで runserver してみよう。
```python
print("♦️ STATICFILES_DIRS:",STATICFILES_DIRS)
```
サーバー起動のターミナル画面に、ルート情報が出てくるはず。

```
♦️ STATICFILES_DIRS: [PosixPath('/…(略)…/secret_gallery/static')]
```
こんな風にルートが出力される。（…(略)… の部分は環境によって違うので、プロジェクトルートから記載）
プロジェクト直下の static フォルダを参照していることが分かったね。

・・・ということは、**この設定をすると、プロジェクト直下に staticフォルダを設置しても、Django は自動的に探しに行ってくれる！だから、プロジェクト直下にも、staticファイル を置くことが可能！**
プロジェクト全体で使いたい css / js / アイコン / 画像 があれば、アプリ配下じゃない場所に置いた方が、プロジェクト構造的にマッチする場合もありそうね！

![](/images/c5_p5_23_static.png =500x)
これは初出の知識だよ！
（本当なら templates でも同じ構造という説明ができるから出したかったんだけど、プロジェクトルート下で共通テンプレ化する構造が思い浮かばなかった🙏懺悔）


3. STATIC_ROOT

これは、本番用サーバーのための設定。
いままでは、Django のプロジェクトルート内の話だったよ。
そしてローカル環境であれば、ROOT_URL / STATICFILES_DIRS を設定するだけで、Django が静的ファイル配信をしてくれるんだった。

だけど、本番環境だと、静的ファイルを Django が配信することができない。
なぜなら、Django はルートディレクトリしか分からないから。
本番では https://グローバルIP/static/css/style.css みたいな配信 URL になるんだよね。
そうなると、Django ではグローバルIP を知ることができないから、本番サーバーに静的ファイルを乗せられないの。


そんなときのために、STATIC_ROOT が存在する。
```
STATIC_ROOT = BASE_DIR / "staticfiles"
```
この設定をして、本番サーバーで、staticファイルを集めるコマンドを唱える。
```
python3 manage.py collectstatic
```

そうすると、STATIC_URL / STATICFILES_DIRS に設定されている staticファイル全てが、
ルートディレクトリ直下の staticfiles というディレクトリの中に集まってくる！

:::message
🫛豆知識

静的ファイルを集めてくる魔法の言葉。
**python3 manage.py collectstatic**

最後に、流れを確認して終わりにしよう！

- STATICFILES_DIRS：「アプリ内ではないけど、ここに置いた CSS / JS ...などを使いたい！」というときに設定する。複数のときは、リストで指定可能。

もしも、アプリ直下に static フォルダを置かずに、プロジェクト直下で staticファイルを全て管理したい場合には、こういう構造にもできるね。
```django
project/
├── static/             # 🔴 プロジェクト共通の CSS や JS
├── app1/static/app1/   # 🔵 app1 の CSS や JS
└── app2/static/app2/   # 🟡 app2 の CSS や JS
```
このときは、STATIC_URL の設定はなくてOK！

代わりに、この設定が必要。
```django
STATICFILES_DIRS = [
    BASE_DIR / "static",        # 🔴
    BASE_DIR / "app1/static",   # 🔵
    BASE_DIR / "app2/static",   # 🟡
]
```

この２つ、実は「入力元」のフォルダ設定。
🟩 STATIC_URL = "static/"
🟧 STATICFILES_DIRS = [...]<br>
対するこちらは、python3 manage.py collectstatic を実行したときの「出力先」フォルダ設定。
🟦 STATIC_ROOT = BASE_DIR / "staticfiles"<br>
**さて。ここでクイズです‼️**

**もしも、STATIC_ROOT を下記のように設定して python3 manage.py collectstatic を**
**実行した場合、Django はどのような動きをするでしょうか❓ ❓ ❓**
```
STATIC_ROOT = BASE_DIR / "static"
```
<br>

> ⭐️**こたえ**⭐️
>　**staticフォルダ内のファイルがぐっちゃぐちゃになる！！！！！！**
> 　**きゃーーーーーーーーーーーー**😱😱😱😱😱😱

なぜなら、こういうこと・・・😱
1. static/ にある自分のファイルをコピーしようとする
2. でも出力先も同じ static/ だから、コピー元とコピー先がごっちゃになる
3. 既存ファイルを上書きしたり、 collectstatic の管理ファイルが混ざったりして、ぐっちゃぐちゃになる

<br>
え？やったこと？
<br>
**あるに決まってんだろ！！！！！！笑笑笑**
<br>
まぁ、さ。
STATIC_ROOT は「出力先」であって、「入力元」じゃないからさ。
間違っても **STATIC_ROOT = "static"** なんて、ぷに蔵みたいにアホなことしちゃダメだぞ⭐️
:::

そして、本番サーバーにおける静的ファイルの配信は、Nginx などの Web サーバーを介して行われる仕組みです。
（本番では、Nginx と STATIC_ROOT をつなぐ設定は、別途必要になるからね）


## 📕 現代社会へ降り立った Django 戦士へ

いままでお疲れさまでした！！
そして、一緒にがんばってくれてありがとう！！

ぷに蔵はいま、すべての章を書き終わって、感無量です・・・。
本当に疲れた・・・・・
矢吹ジョーのように真っ白になりそうです。右手がずっと痺れてます。

初めて Django を触った人なら、分からないところも多かったよね。
ぷに蔵も初めて Django に触ったとき、本当に右も左も分からなくて・・・むしろ、「なにこのコード？？？昨日までプログラミングやってなかったっけ？この知識、本当に必要なの？？？」とか思ってた。

必要どころの話じゃないくらい、いまはどっぷり頭まで浸かって、Django を開発しているエンジニア様たちを、日々崇めながら使わせていただいております。


ここから、VPS レンタルサーバーを借りたデプロイをしたり、AWS / GCP などを使用したりして、開発は続いていきます。


デプロイ編や、その先にも色々機能追加を・・・と考えていましたけど、止めました。
これ以上は、無粋かな、って。

きっと、ここまで出来たのなら、自分で調べて、自分で実装する根気強さがあるはずだから、ぷに蔵が一から「ここをこうして、できたー！」ってやる必要はないと思いました。

だから一度、ここで「秘密のプライベートギャラリー」プロジェクトは完成です！！


でも、このプロジェクト自体をぷに蔵は気に入ったので、今後、これを更にゴリゴリに発展させて、個人開発で何かに使えるかな〜🧐とか思っています。


今後のエンジニア人生の中で、いつか「秘密のプライベートギャラリーシリーズで Django 始めたよ」と言ってくれる人に出会えたら良いなという願いを込めて、ここに終了を宣言させていただきます。


ありがとうございました。