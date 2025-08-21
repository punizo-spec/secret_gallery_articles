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

エラーが出たときは、まずは確実に **仮想環境に入っていること** を確認しよう！
**コマンドラインの頭に (venv) が付いている？とっても大事** だよ！！<br>
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
① 「 deactivate 」で一度その仮想環境を抜ける
② 「 cd xxxxx 」で自分で作成したディレクトリ（ 例：django_workspace ）に移動（自分で任意の名前を付けたなら、コマンドは適宜調整してね）
③ 改めて仮想環境をON！「 source venv/bin/activate 」
④ そこで改めて Django のインストールコマンド！！「 pip install django 」

<br>

「 which pip 」の結果が、今回作ったディレクトリ内の仮想環境で **間違いない場合** は、pip が使用する python のバージョンも確認して！
```
pip --version
``` 
↓ ぷに蔵の環境で出力した結果はこれ ↓
```
% pip --version
pip 25.2 from /Users/punizo/django_workspace/venv/lib/python3.12/site-packages/pip (python 3.12)
```
今回作成した「 django_workspace 」フォルダ内の「 venv 」の中の pip が「 python3.12 」を使用していることが分かる。

ここらへんをチェックして、依存関係を把握していくことが大切だよ☆<br>
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
青枠で囲んだ URL を見てほしい！http://127.0.0.1:8000/ と書いてあるので、ブラウザに貼り付けて読み込んでみてくれ！！
![](/images/c2_p2_3_runserver.png)

ロケット LAUNCHED 🚀
![](/images/c2_p2_4_rocket.png)

これが Django のローカルサーバーだ！！！


素晴らしすぎる！！第一回スタオベの瞬間よ！！
環境構築乗り越えて、Django プロジェクト始動させて、サーバー起動させた。
確実にステップ上がってきてる！！この調子で、次はアプリ編だ！

ローカルサーバーを起動できることが確認できたら、一度サーバーを落とそう！
コマンドラインに『 Quit the server with CONTROL-C. 』と書いてあるので、これに倣って、
**Ctrl + C** でサーバーを停止させてね。


::::details サーバー起動でエラーが出た人はこちら。
エラーは、コマンドミスでよく出るよ！<br>
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
でもこれは、環境によって起動できる / できないが変わるから、最初のうちは安全のためにも python3 manage.py runserver を使うようにしておくことを推奨するよ！<br>
どんどん開発に慣れていって、やがて「シンボリックリンク」という言葉に遭遇するときがきたら、python manage.py runserver で起動できる理由が紐解けると思う。

::::

:::message
##### ローカルサーバーとは？？？
「ローカルサーバー」って言葉、聞き慣れないよね？

簡単に言うと、
🟢 **自分のパソコンの中だけにある、テスト用のWebサーバー** 🟠

通常、Webサイトはインターネットにあるサーバーにアップロードしないと見られないんだ。
でも Django では開発のときに「とりあえず自分のパソコンの中だけで動かせるサーバー」を用意してくれているの。さすが。<br>
だから http://127.0.0.1:8000/ をブラウザで開くと、まるでインターネットを見ているかのような表示がされるんだけど、実はこれは自分の PC の中を見に行っているだけなんだ。<br>
でも、この仕組みのおかげで、わざわざ開発中にインターネットにアプリを公開しなくても、現状の Webアプリ が確認できるの！<br>
だから、開発中に アプリを確認したいときには、この「ローカルサーバー」を動かして確認するんだ。

:::


## 03. プロジェクトにアプリを爆誕🔥 叫べ startapp！！
今度はアプリを作成するよ。
第１章の最後に、「Django は機能ごとにアプリを持つ」って説明したよね。
今回はプロジェクト全体から、プロジェクトに機能を付加させるためのアプリを作成しよう！

```bash
python3 manage.py startapp sg_user
```

secret_gallery と同じ階層に sg_user がフォルダが増えている！
![](/images/c2_p3_5_project.png =450x)
アプリの作成成功！！

**プロジェクトとの違いが分からない？**
うん。分かりづらいよね！！
目安は「settings.py」があればプロジェクト、「apps.py」があればアプリと思っておいて！


## 04. アプリの「sg_user」とは、なにに使うのですか？
## 05. settings.pyは設定ファイル（文字どおりすぎる…）
## 06. makemigrations と migrate はセット商品です
## 07. 我はギャラリーを統べる者、也。python3 manage.py createsuperuser
## 08. Djangoには管理画面というものがあるんですよ
## 📕 runserverのオマケ（エラー一覧）
## 🌵 おまけ 🌵 admin.pyのSG_USERのUSERSって……なんだこれ？
