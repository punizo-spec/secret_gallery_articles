---
title: "第２章　ロケットを打ち上げろッ🚀"
emoji: "♋️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["django", "python"]
published: true # trueを指定する
published_at: 2025-09-02 08:12
---
## 01. プロジェクト始動！ SAY**☆STARTPROJECT☆**
いま、君の CUI の頭には（venv）が刻み込まれているかな？
それではまず、Django を使用する準備からいくね。
```bash
pip install django
```
pip については、前章で学んだね！パッケージをインストールするコマンドだった。
それを使って、仮想環境に Django パッケージをインストールしたよ。
無事にインストールできた？

::::details エラーが出た人はこちら
エラーの種類的には
- Defaulting to user installation because normal site-packages is not writeable
- ERROR: Could not find a version that satisfies the requirement django (from versions: none)
- SyntaxError: Missing parentheses in call to 'print'
うーん。あとは Permission denied もあり得るかな？

エラーが出たときは、まず最初に **仮想環境に入っていること** を確認しよう！
**コマンドラインの頭に (venv) が付いている？とっても大事** だよ！<br>
もし (venv) が付いているのにエラーが出たら、下記コマンドを打ってみて。
```bash
which pip
```
これで、pip がどこを参照しているのかを確認する。
ちゃんと、今回ディレクトリ内に作成した仮想環境（venv）内の pip を参照している？
ここに表示されるパスが **「/自分で作成したディレクトリ/venv/bin/pip」** になっていることが重要！
```
/Users/punizo/django_workspace/venv/bin/pip
```
もしも全く違うところを参照していたら、
1. 「 deactivate 」で一度その仮想環境を抜ける
2. 「 cd xxxxx 」で自分で作成したディレクトリ（ 例：django_workspace ）に移動
3. 改めて仮想環境をON！「 source venv/bin/activate 」
4. そこで改めて Django のインストールコマンド！！「 pip install django 」

<br>

「 which pip 」の結果が、今回作ったディレクトリ内の仮想環境で **間違いない場合** は、pip が使用する python のバージョンも確認して。
```
pip --version
``` 
↓ ぷに蔵の環境で出力した結果はこれ ↓
```
% pip --version
pip 25.2 from /Users/punizo/django_workspace/venv/lib/python3.12/site-packages/pip (python 3.12)
```
今回作成した「 django_workspace 」フォルダ内の「 venv 」の中の pip が「 python3.12 」を使用していることが分かる。

ここらへんをチェックして、依存関係を把握していくことが大切だよ。<br>
もしここで python2.x とか出てきたら、python3.x にアップデートしないと危険！
python2 は2020年1月1日で公式サポートも終了しているし、なんならこれから使う Django でも、新しいバージョンでは非対応になっているからね。
::::


では、いこう！！！
```bash
django-admin startproject secret_gallery
```

CUI 上は、おそらく何も変わっていないように見える。だけどディレクトリ内には Django プロジェクトが作成されているはずだよ。
下記を参考に、VSCode で Django プロジェクトを開いてみるんだ！！
![](/images/c2_p1_1_vsc.png)
① → ② の順番でクリックすると、フォルダ選択がポップアップ表示されるので、自分が作ったフォルダを開いてみてくれ。
そうすると、こんなフォルダとファイルが表示されてないかな？
![](/images/c2_p1_2_project.png =450x)

これが Django プロジェクトの全貌だ！！！（現時点）


## 02. 自分のサーバーを起動させてみるぜ🚀
さっそくだが、コマンドだ！
```bash
python3 manage.py runserver
```
Django でローカルサーバーを起動するコマンド実行で、下記のように表示がされる。
青枠で囲んだ URL を見てほしい！
http://127.0.0.1:8000/ と書いてあるので、ブラウザに貼り付けて読み込んでみてくれ！！
![](/images/c2_p2_3_runserver.png)

ロケット LAUNCHED 🚀
![](/images/c2_p2_4_rocket.png)

**これが Django のローカルサーバーだ！！！**


素晴らしすぎる！！第一回スタオベの瞬間よ👏👏👏
環境構築乗り越えて、Django プロジェクト始動させて、サーバー起動させた。
確実にステップ上がってきてる！！この調子で、次はアプリ編！

ローカルサーバーを起動できることが確認できたら、まずは一度サーバーを落としてね、
コマンドラインに『 Quit the server with CONTROL-C. 』と書いてあるので、これに倣って、
**Ctrl + C** でサーバーを停止させるよ。


::::details サーバー起動でエラーが出た場合はこちら。
この他ミングでの起動エラーは、コマンドミスでよく出るよ！<br>
たとえば・・・
1. can't open file '/Users/punizo/Desktop/django_workspace/secret_gallery/manage,py': [Errno 2] No such file or directory
→ manage.py の ピリオドがカンマになっていたら怒られる。
2. Unknown command: 'runserve'. Did you mean runserver? Type 'manage.py help' for usage.
→ runserver のスペルミスでも怒られる。

しっかり正しいコマンドを入力しているか、確認してみてね☆

ちなみに、このコマンドで起動できる人もいると思う。
```bash
python manage.py runserver
```
でもこれは、環境によって起動できる・できないが変わるから、最初のうちは安全のためにも **python3 manage.py runserver** を使うようにしておくことを推奨するよ。<br>
どんどん開発に慣れていって、やがて「シンボリックリンク」という言葉に遭遇するときがきたら、**python manage.py runserver** でも起動ができる理由が紐解けるはず。

::::

:::message
##### ローカルサーバーとは？？？
「ローカルサーバー」って言葉、聞き慣れないよね？

簡単に言うと、
🟢 **自分のパソコンの中だけにある、テスト用のWebサーバー** 🟠

通常、Webサイトはインターネットにあるサーバーにアップロードしないと見られないのね。
でも Django では開発のときに「とりあえず自分のパソコンの中だけで動かせるサーバー」を用意してくれているの。さすが。<br>
だから http://127.0.0.1:8000/ をブラウザで開くと、まるでインターネットを見ているかのような表示がされるんだけど、実はこれは自分の PC の中を見に行っているだけなんだよ！<br>
でも、この仕組みのおかげで、わざわざ開発中にインターネットにアプリを公開しなくても、作成中の Webアプリ の現状を確認できるの。<br>
だから、開発中に アプリを確認したいときには「ローカルサーバー」を動かして確認するよ。

:::


## 03. プロジェクトにアプリを爆誕🔥 叫べ startapp！！
今度はアプリを作成するよ。
第１章の最後に、「Django は機能ごとにアプリを持つ」って説明したよね。
なので、今回はプロジェクトに機能を付加させるための「アプリ」を作成しよう！

```bash
python3 manage.py startapp sg_user
```

secret_gallery と同じ階層に sg_user がフォルダが増えている！
![](/images/c2_p3_5_project.png =450x)
アプリの作成成功！！

**プロジェクトとの違いが分からない？**
うん。分かりづらいよね！！
いまのところ、目安は「settings.py」があればプロジェクト、「apps.py」があればアプリと思っておいて！


## 04. アプリの「sg_user」とは、なにに使うのですか？

「アプリを作ったはいいけど、『sg_user』アプリって一体何のために作ったの？」という疑問が出てくるよね？
その疑問について、ちょっと考えてみよう！

これから Django で作っていくのは、**秘密のプライベートギャラリー**という Webアプリケーション。ギャラリーというからには、作品が展示されているイメージじゃない？

ということは、
> 1. 作品ごとに「タイトル」「画像」「コメント」などの情報がありそうだから
> 2. それらの情報をデータベースに登録しておいて、
> 3. 表示するときにはデータベースの内容を読み込んで、ブラウザに表示しようかな

・・・という流れが想像できると思う。<br>
このとき必要になるのが、「**作品を登録・編集・削除できる人＝データベースを操作できる人**」なんだよね。<br>

実は Django には、データベースの中身を管理できる「管理画面」というものが、最初から標準装備で用意されている。でも、この管理画面はセキュリティがしっかりしているから、ログインしないと入れない仕様なんだ。

ということは、「管理画面に入ってデータベースを操作する人」が必要になるよね。

そこで、「sg_user」アプリの出番。
アプリ＝機能ごとのパーツを作る、って説明をしていたの覚えている？

管理画面に入れるユーザー（管理者）を管理するための機能を「 sg_user 」アプリで作成しよう！！ということなの。


それでは早速、作ってみよう！
使用するファイルは、sg_user アプリ内にあるmodels.py
![](/images/c2_p4_6_customuser.png)

↓ コード詳細 ↓
```python
# sg_user/models.py
from django.contrib.auth.models import AbstractUser

class CustomUser(AbstractUser):
    # ログインには username を使います
    USERNAME_FIELD = "username"
    REQUIRED_FIELDS = []

    def __str__(self):
        return self.username
```

はじめて触るファイルが出てきたね！

> #### models.py

あまり馴染みがないファイル名だけど、Django でデータベースを扱ううえで、とても重要なファイルになるよ。
それでは、公式説明を読んでみよう！基本的には公式は絶対なので、（一応）確認してみる。

>英語版：A model is the single, definitive source of information about your data. It contains the essential fields and behaviors of the data you’re storing. Generally, each model maps to a single database table.[（django公式 英語版より）](https://docs.djangoproject.com/en/5.2/topics/db/models/)
>日本語版：モデルは、データに関する唯一かつ決定的な情報源です。あなたが保持するデータが必要とするフィールドとその動作を定義します。一般的に、各モデルは単一のデータベースのテーブルに対応付けられます。[（django公式 日本語版より）](https://docs.djangoproject.com/ja/5.2/topics/db/models/)

公式が絶対のなので確認はしたが・・・・・理解できるとは言っていない！！
いや、無理だよ、これは笑笑
ちょっと公式のハードルが高いので、ぷに蔵なりに解説する🌻

みんなはこれまでに、データベースを触ったことはある？
MySQL や PostgreSQL を使ったことがある人は、「テーブル」を作って、そこにデータを登録した経験があるかもしれないね。

:::message
##### 今回はじめてデータベースを触る人は、こちらも読んでほしい
データベースを作るときには、１つデータベースを作ったら、その中に「テーブル」を作成してあげる。そして、このときのテーブルは、データの種類ごとに作成することが一般的だよ。
（稀に、すべてが１つのテーブルにぎゅうぎゅうに入っている DB を見かけるけど、その作り方はおすすめしない！管理できなくなるし、汎用性ゼロだから）<br>
例えば、社員番号・名前・部署番号を記録する「employee」テーブル、部署番号・所属部署を設定した「department」テーブル、入社・異動・退社などの履歴を記録する「history」テーブルを作るとき、テーブルごとに入力しないといけない情報は違うよね。
「社員番号・名前・部署番号・所属部署」みたいな、個々のデータのことを「フィールド」と呼ぶんだけど、その「フィールド」は、作成するとき「**データ型**」を指定してあげる必要があるの。
ここのフィールドは「文字列」・保存文字数は50・入力必須、ここのフィールドは「数値」・負の数も可能・default値は0、みたいに、ひとつひとつ設定したテーブルを集合させたものが、１データベースの構成要素になるよ。<br>
:::

データベースを作成した後は、テーブル（データを保存する箱みたいなもの）を作るんだけど、その「箱」を作るためのツールが Django の場合には models.pyなの。

簡単に言うと、**models.py に「テーブルの設定詳細（モデル定義）」を書くと、Django がその定義どおりにデータベースにテーブルを作ってくれる**んだ。

例えば、Game というモデルを定義してみるね。
```python
class Game(models.Model):
    title = models.CharField(max_length=100)
    released_at = models.DateField()
```
とってもシンプルだけど、これだけで、「 Game 」という名前のテーブルが作成されて、中には「タイトル」と「発売日」というフィールド（＝カラム）が作られる。
書き方のパターンは、色々書いていけば覚えるから、いまは「ふーん」くらいに思っておいて。

:::message
 🫛 豆知識
モデル名（クラス名）は、基本的に **単数形** を使うのが Django の慣習だよ。
理由は長く、ややこしい話になるので割愛ｗ
:::

例えば、さっき上記に書いたコードの構造を見てみると・・・
![](/images/c2_p4_7_cumodel.png)
1行目：Djangoの「管理ユーザー機能の土台（モジュール）」を読み込む
3行目：CustomUser クラスを定義。AbstractUser を継承して、「ログインできる人」のモデルを作っていく宣言
5行目以降は、CustomUser クラスが持つメソッドなどを書く

基本的な構造はいつも同じ。
> 1. クラスやメソッドが必要とするモジュールを読み込んで
> 2. クラス名を定義したうえで
> 3. 継承するクラスを定義して
> 4. 定義したクラスの中に、メソッドを書いていく

:::message
ぷに蔵の肌感覚的に、Django をはじめて触った最初のタイミングで、 models.py に AbstractUser を使ったモデル定義を理解・・・は、難しすぎる！！！
だから、いま時点で何か覚える必要はまったくなくて、今回はコピペもしくは写経（コードをただ書き写すこと）で良いと思うの。<br>
第２章の最後にちょっとだけ読み物として、この「 AbstractUser 」について書いたんだけど、改めて考えてみても、結構難しいなと感じたのよ。<br>
なので、今回ここで理解してもらいたいことは１つだけ！
> AbstractUser クラスというものを継承して、管理画面にログインできるユーザーを作成する機能を作った

この事実だけで良い。<br>
もしも、「**username でログインするように「USERNAME_FIELD = "username"」って書いた**」ということも覚えてくれたら、それでもおう十分すぎるくらだよ！！
:::

< models.py コード再掲>
```python
# sg_user/models.py
from django.contrib.auth.models import AbstractUser

class CustomUser(AbstractUser):
    # ログインには username を使います
    USERNAME_FIELD = "username"
    REQUIRED_FIELDS = []

    def __str__(self):
        return self.username
```

::::details AbstractUserってなに？（書いてみたけど、読まなくても良いかも）
AbstractUser は、Djangoが用意している「ログインに必要な機能（パスワード、権限、最終ログインなど）」が最初から入った“ユーザーの雛形”のこと。
継承するだけで、管理画面にログインできる人を自分のプロジェクト用に持てるの。
標準装備のフィールドがいくつかあるんだけど、その標準に追加したいフィールドがあれば、自分で追加できることが強み。
（フィールド例：username(必須), password(必須), email(任意), ファーストネーム(任意), ラストネーム(任意) ...and more）
今回は中身を増やさず“雛形を受け継いだだけ”。<br>
USERNAME_FIELD = "username"　← こう書くと、今回作成したユーザーは、username を使用してログインすることになる。
AbstractUser は username + password でのログインがデフォルトだから書かなくても良いんだけど、後から見たときに明示的に分かりやすいから書いておいただけ（AbstractUser は、他の任意のフィールドを使用したログインに変更することも可能。email + password とかね）。<br>
REQUIRED_FIELDS = []　← こう書くと、必須項目以外は、ユーザー作成のときに入力が必要ではありません、という意味。<br>
この管理ユーザーって、いまはとってもイメージしづらいから、第２章が全部終わってから、気になったらもう一度読んでみる軽さで進めてね🌻
::::

## 05. settings.pyは設定ファイル（文字どおりすぎる…）

「sg_user」アプリの models.py で、管理者ユーザーの設定が終わったね。
今度は、作成した管理者ユーザーの情報を、Django に「これからは管理者、このモデルでいきますね」って教えてあげる設定をするよ〜。

::::details settings.py に AUTH_USER_MODEL を設定する意味
Django にはデフォルトの Userモデル（django.contrib.auth.models.User）というものが最初から存在するんだけど、今回自分で作った CustomUserモデルを使用するためには、この「教えてあげる設定」が必要。<br>
これを書かないと、Django 的には「え？管理者ユーザーっていつも（標準）の Userモデル使うに決まってるでしょ」って思ってしまうから、<br>
もしもこの設定を忘れて、Django さんが管理者モデルを標準 User として記憶してもらったら、とっても大変！！<br>
なぜなら、この管理者ユーザーは後から変えることができない！！！（……わけじゃないけど、変えるならプロジェクト作り直した方が手っ取り早いくらい面倒くさい！）
だから、ここで忘れずに設定しよ。
::::

**使うファイルは settings.py**
このファイルは、プロジェクト全体にかかる挙動を設定するよ。

![](/images/c2_p5_8_settings.png)

プロジェクトを開始してから、
1. sg_user アプリを作成して
2. 管理者ユーザーを設定した
よね。だから、それを settings.py に追記する。

まずは、ファイルの中から INSTALLED_APPS を探して。
33行目くらいにあると思う。
そして INSTALLED_APPS の一番最後に、「 sg_user 」を追加する。
元々初期設定されていた django.contrib たちは Djangoの標準機能なので、ここでは触らなくていいよ。（コメントは任意。今回は説明のために入れておくね）

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
]
```
こんな感じになったかな？

そしたら次に、管理者ユーザーモデルを設定する。
INSTALLED_APPS の下に MIDDLEWARE があるから、その下に追記しよう。

```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

AUTH_USER_MODEL = "sg_user.CustomUser"
```
どうかな？こんな並びになった？
これで設定はOK！次にいこう！！

:::message
何回かプロジェクトを作って慣れてきたら、settings.py へ AUTH_USER_MODEL の追記は、モデルを定義するより先に書くのが安全！
いまはファイルの流れで理解して欲しいから、settings.py を後回しにしているだけだよ。
:::

## 06. makemigrations と migrate はセット商品です
06, 07 は一気に駆け抜けるよ！ここだけは途中離脱厳禁！！
説明は全部後回し！！！

ターミナルでコマンドを打て-------💥💥💥

```bash
python3 manage.py makemigrations
```

```bash
python3 manage.py migrate
```

ターミナルはこんな感じになったかな！？？
![](/images/c2_p6_9_migrate.png)

ここでエラーが出たら、落ち着いて、下記２つを確認。
1. (venv) がコマンドラインの先頭についている？
2. コマンドの入力ミスはない？


それでは、このままのノリで 07 にいく！！

## 07. 我はギャラリーを統べる者、也。python3 manage.py createsuperuser
ここで全てが結実する！

いっけぇーーーーーー！！！
```bash
python3 manage.py createsuperuser
```

コマンドラインで username 聞かれたら、好きな名前を入れて！
password は入力しても、画面表示はされないよ！！
成功したら successfully だ！！！

![](/images/c2_p7_10_superuser.png)

第２章の伏線全回収！！！
1. アプリ作って
2. 管理者モデル定義して
3. settings.py で新しい自作 Userモデルを設定して
4. マイグレートして
5. スーパーユーザーを作成した

素晴らしい完走だ・・・。

しかし、ぷに蔵がぶっ飛ばしすぎて、「4、5 なんだ！？！？」ってなっていると思うので、説明させていただきます⭐️

まず、この２つのコマンドの意味から。
```bash
python3 manage.py makemigrations
python3 manage.py migrate
```
- `makemigrations` : モデル定義（＝テーブル構造）を DB に反映させるため、保存用ファイルに変換します。「こういう形で保存しますね」という準備作業
- `migrate` : makemigrations で保存用変換ファイルが作成されたので「DB に反映させます」という実行コマンド

> 🟢 **ファイル作成して → 実行** 🟠

> 🟢 **makemigrations → migrate** 🟠

ここで、06パートのタイトル回収させていただく。

> 🟢 **06. makemigrations と migrate はセット商品です**🟠

作ったら、実行。
これは完全ニコイチ！！
覚えておいてほしいなーー。
<br>
そして次に、これ。
```bash
python3 manage.py createsuperuser
```
createsuperuser : スーパーユーザー権限をもつ管理ユーザーを作成します

ちなみに、スーパーユーザーという言葉が出てきたので、ここで管理ユーザーの種類について説明しておくね。

AbstractUser を継承して作成した場合、次のようなユーザー権限を表す以下のフィールドが、あらかじめ設定されているの。
> 💠 **is_active : 現在有効（＝ログイン可能）なユーザーか否か**
> 💠 **is_staff : 管理画面（admin/）に入れる権限を持つか**
> 💠 **is_superuser : 管理画面において、操作に制限がないユーザーか否か（全権限持ち。権限チェックも無視される存在）**

今回は createsuperuser ということで、「管理画面において、操作に制限がないユーザー」を作成したということ。
会社でも、役職によってシステム権限が変わったりするじゃない？そんなイメージ！
<br>
あと、気になる人もいるかもしれないやつなんだけど・・・
> manage.py ← ナニコレ？

突然出てきたよね。startapp から突然登場したのよ。

これはどんなときに使うのかというと、**Django プロジェクトに対して何かしらの操作を行いたいときに使用します！**

さっきまでの例を見てみよう。
```bash
python3 manage.py startapp sg_user
python3 manage.py runserver
python3 manage.py makemigrations
python3 manage.py migrate
python3 manage.py createsuperuser
```
全部共通して「 python3 manage.py 」から始まっている。
なぜなら manage.py は Django 操作の受付窓口のためです！！

いままで実行してきたコマンドは、こんな意味だったんだ。
> python3 : python3 で実行する指令を出しますね
> manage.py : manage.pyさん（窓口業務）
> startapp sg_user : sg_user というアプリを開始したいな（→ 作成してくれる）
> 
> python3 : python3 で実行する指令を出しますね
> manage.py : manage.pyさん（窓口業務）
> runserver : ローカルサーバー起動して
> 
> python3 : python3 で実行する指令を出しますね
> manage.py : manage.pyさん（窓口業務）
> makemigrations : モデル定義のテーブル構造を DB に反映させるためのファイルを作成して

全部、Django プロジェクトへの指示のために manage.py を経由しているよね。
（ちなみにこのファイルは、startproject のときに自動生成されるファイルだ！）

これも、なんとなーく「へー」と思っておいてくれればいいお話です🌻


## 08. Djangoには管理画面というものがあるんですよ

これまで管理ユーザーの作成していたよね。
そして、さっきから当然のように「管理画面、管理画面」言っていたじゃない？

そうなの。Django は、デフォルトで管理画面を準備してくれているの！
実際に、見に行こう！！！

その前に、まずは設定だ！！！

使うファイルは sg_user アプリ内の admin.py だよ。
（admin.py は、アプリごとに存在するファイル）
このファイル（ admin.py ）に、「このモデルを管理画面に表示させる」と書いて Django に伝えることで、管理画面に表示されるようになるのね。

今回は、管画面上に CustomUser を表示させたいから、以下のように書いていくよ！

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import CustomUser

@admin.register(CustomUser)
class CustomUserAdmin(UserAdmin):
    pass
```

この書き方は、いまはそのまま「こういうものなんだな」と思っておいて！

1. Django の管理画面機能（admin モジュール）を読み込む
2. Django 標準の User 管理機能を持つクラス（UserAdmin）を読み込む
3. 管理画面に表示させたい CustomUserモデル定義を読み込む
4. CustomUserモデルを管理画面に登録するクラスを定義する  
   （UserAdmin を継承することで、パスワード変更や権限設定などの便利機能も使える！）  
   ※ @admin.register(CustomUser) は「このクラスを CustomUser に結びつけます」という合図


![](/images/c2_p8_11_admin.png)
*VSCode で見たときはこんな感じになっているよ〜*


ではここで、ローカルサーバーを起動🚀
```bash
python3 manage.py runserver
```

そしたら、下記 URL へ、飛べ！！！
[http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/)

ログインは、さっき createsuperuser で作成のときに入力した username と password を使用してね！

こんな画面になっていると思う！

![](/images/c2_p8_12_adminscreen.png)
*SG_USER の表示がなかったら、admin.py のコードミスがあるかもしれない。もう一度、確認してみて？*

Users の中に入ると、自分が作成した管理ユーザーが表示されていると思う。
ここで、他の管理ユーザーを増やしたり、そのユーザーが superuser かどうかの設定をしたりもできる！

管理画面まできたね！！
確実に、ゼロだったプロジェクトを育てあげた成果だよ！！

すごい、すごいっっ🎉🎉🎉


## 🌵 おまけ 🌵 管理画面の SG_USER の USERS ……なんだこれ？
管理画面で、ここ、不思議に思わなかった？
![](/images/c2_p8_13_users.png =280x)

ぷに蔵、最初ここに違和感めちゃくちゃ感じたのよ。
だって、Users なんてモデル定義してないじゃん？て。

でもこれ、Django の基本的な機能部分にも関係してくる話なの。

さっき settings.py に
```python
AUTH_USER_MODEL = "sg_user.CustomUser"
```
って設定したでしょ。

これで Django は Userモデルを sg_user.CustomUser だと認識はしてくれた。
だけど Django 内部では、Userモデル＝sg_user.CustomUser になっただけで、Userモデルが上書きされたわけではない。

Django の管理画面は、モデル名をそのまま表示するわけじゃなくて、
1. デフォルトで CustomUser を 「User」っていう単語として解釈して
2. 英語複数形に修正して（ Users ）表示
というステップを踏んでいるの。

だから、CustomUserモデルが正しく定義されても、ここの表示は Users になっているのよ。
これを修正して「ユーザー」とかにもできる。

::::details Users 表示を「ユーザー」に修正したいひと向け
sg_user/models.py の CustomUser の下部にコード追加。これだけ。

```python
from django.contrib.auth.models import AbstractUser

class CustomUser(AbstractUser):
    # ログインには username を使います
    USERNAME_FIELD = "username"
    REQUIRED_FIELDS = []

    def __str__(self):
        return self.username

    class Meta:
        verbose_name = "ユーザー"
        verbose_name_plural = "ユーザー一覧"
```

ファイルを保存したら、管理画面をリロード。（これは DB のテーブル構造に影響がないから、makemigrations / migrate は不要）
変更された？<br>
でも、表示が変わるだけなので、これをやらなくても「秘密のプライベートギャラリー」上では何の問題もない笑
::::
