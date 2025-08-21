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
ここで表示されたパスが **「/自分で作成したディレクトリ/venv/bin/pip」** になっていることが重要だよ！
```
/Users/punizo/django_workspace/venv/bin/pip
```
もしも全く違うところを参照していたら、
① 「 deactivate 」で一度その仮想環境を抜ける
② 「 cd xxxxx 」で現在のディレクトリから、これから作業する django_workspace に移動（自分で任意の名前を付けたなら、適宜コマンドは調整して）
③ 改めて仮想環境を ON ！「 source venv/bin/activate 」
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

ここらへんをチェックして、依存関係を把握していくことが大切だよ。<br>
もしここで python2.x とか出てきたら、python3.x にアップデートしないと危険！
python2 は2020年1月1日で公式サポートも終了しているし、なんならこれから使う Django でも、新しいバージョンでは非対応になっているんだ。
::::


では、いこう！！！
```bash
django-admin startproject secret_gallery
```

CUI 上は、おそらく何も変わっていないように見える。だけどディレクトリ内には Django プロジェクトが作成されているはずだよ。
下記を参考に、VSCode で Django プロジェクトを開いてみるんだ！！
![](/images/c2_p1_1_vsc.png)
① → ② の順番でクリックすると、フォルダ選択がポップアップ表示されるので、自分が作ったフォルダを開いてみてくれ。
ぷに蔵が開くと、こんな並びになっていたぞ。
![](/images/c2_p1_2_project.png)
そう！
これが Django プロジェクトの全貌だ！！！（現時点）



## 02. はじめて自分のサーバーを起動させてみるぜ🚀
## 03. プロジェクトにアプリを爆誕🔥 叫べ startapp！！
## 04. アプリの「sg_user」とは、なにに使うのですか？
## 05. settings.pyは設定ファイル（文字どおりすぎる…）
## 06. makemigrations と migrate はセット商品です
## 07. 我はギャラリーを統べる者、也。python3 manage.py createsuperuser
## 08. Djangoには管理画面というものがあるんですよ
## 📕 runserverのオマケ（エラー一覧）
## 🌵 おまけ 🌵 admin.pyのSG_USERのUSERSって……なんだこれ？
