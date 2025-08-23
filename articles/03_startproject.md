---
title: "第２章　ロケットを打ち上げろッ🚀"
emoji: "♋️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["django", "python"]
published: true # trueを指定する
published_at: 2050-08-20 00:00 # 未来の日時を指定する
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

::::details エラーが出た人はこちら。
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
![](/images/c2_p3_6_customuser.png)

↓ コード詳細 ↓
```python
# sg_user/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    # メールアドレスは将来的に使うかもしれないので、一応残しておきます
    email = models.EmailField(
        max_length=254,
        blank=True,
        null=True,
        unique=True)

    # ログインには username を使います
    USERNAME_FIELD = "username"
    REQUIRED_FIELDS = []

    def __str__(self):
        return self.username

```

はじめて触るファイルが出てきたよ！！

> #### models.py

あまり馴染みがないファイル名だと思う。Python 学習では出てこないファイル名だよね。
それではまず、公式説明を読んでみよう！基本的には公式は絶対なので、（一応）確認してみる。

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

データベースを作成した後は、テーブルを作成してデータを保存する箱を作っていたと思うんだけど、その「箱」を作るためのツールが Django の場合には models.pyなの。

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

例えば、さっき上記に書いたコード。

![](/images/c2_p3_7_cumodel.png)




:::details Djangoが自動で用意してくれるフィールド一覧（"AbstractUser"の継承）

実は、`AbstractUser` を継承すると、以下のようなフィールドが最初から付いてくるよ。

| フィールド名 | 説明 | null / blank | default |
|--------------|------|--------------|---------|
| `id` | 主キー（自動） | ❌ / ❌ | 自動生成 |
| `username` | ログイン時のユーザー名 | ❌ / ✅ | なし（必須） |
| `email` | メールアドレス | ✅ / ✅ | 空欄OK |
| `password` | ハッシュ化されたパスワード | ❌ / ❌ | なし（必須） |
| `first_name` | ファーストネーム | ❌ / ✅ | 空欄OK |
| `last_name` | ラストネーム | ❌ / ✅ | 空欄OK |
| `is_active` | 利用可能ユーザーかどうか | ❌ / ❌ | `True` |
| `is_staff` | 管理画面に入れるか | ❌ / ❌ | `False` |
| `is_superuser` | 全権限持ってるか | ❌ / ❌ | `False` |
| `date_joined` | 登録日時 | ❌ / ❌ | `timezone.now` |
| `last_login` | 最後にログインした日時 | ✅ / ❌ | `None` |
| `groups` / `user_permissions` | パーミッション管理用 | ✅ / ✅ | 空リスト |

※ `null=True` は「DBでNULLを許可する」、`blank=True` は「フォームで空欄を許可する」の意味

:::







## 05. settings.pyは設定ファイル（文字どおりすぎる…）
## 06. makemigrations と migrate はセット商品です
## 07. 我はギャラリーを統べる者、也。python3 manage.py createsuperuser
## 08. Djangoには管理画面というものがあるんですよ
## 📕 runserverのオマケ（エラー一覧）
## 🌵 おまけ 🌵 admin.pyのSG_USERのUSERSって……なんだこれ？
