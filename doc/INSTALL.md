インストールマニュアル
===========

tDiaryのインストールと設定
----------------

一般的なCGIの実行を許可しているISPやレンタルサーバ上で利用する場合を想定し、以下のような環境を例に説明します。

  - WWWサーバ: Apache 1.3.x
  - ユーザ名: foo
  - 日記のURL: http://www.hoge.example.org/~foo/diary/
  - 上記URLのパス: /home/foo/public\_html/diary

### CGIスクリプトの設定

配布アーカイブを展開し、中身をすべて/home/foo/public\_html/diaryにコピーします。以下の2つのファイルがCGIスクリプト本体なので、WWWサーバの権限で実行可能なようにパーミッションを設定してください。

  - index.rb
  - update.rb

また、/usr/bin/envを使った起動ができない環境では、各ファイルの先頭を、rubyのパスに書き換える必要があるでしょう。ISPのホームディレクトリにこっそりRubyを入れたような場合を除き、通常はあまり気にしなくても良いはずです。

### .htaccessの作成

続いて、CGIの実行環境を整えます。dot.htaccessを.htaccessにリネームして、環境に合わせて書き換えます。添付のサンプルは以下のようになっています。

```
Options +ExecCGI
AddHandler cgi-script .rb
DirectoryIndex index.rb

<Files "*.rhtml">
   deny from all
</Files>

<Files "tdiary.*">
   deny from all
</Files>

<Files update.rb>
AuthName      tDiary
AuthType      Basic
AuthUserFile  /home/foo/.htpasswd
Require user  foo
</Files>
```

ここでは、

  - CGIの実行を可能にし、
  - サフィックス「.rb」のファイルをCGIと認識させ、
  - index.rbをデフォルトのファイルに設定し、
  - *.rhtmlとtdiary.*のファイルの参照を禁止して、
  - update.rbへのアクセスにはユーザ認証が必要

という設定になっています。とりあえず書き換えが必要なのは、AuthUserFileとRequire userでしょう。意味はWebででも調べて下さい。AuthUseFileは、あらかじめhtpasswdコマンドで作成しておく必要があります(これもWebで調べればわかります)。

また、利用するWWWサーバの設定が、CGIの実行ファイルのサフィックスを固定(例:.cgi)にしている場合があります。この場合、AddHandlerやDirectoryIndexも変更する必要があるでしょう。これに応じて、index.rbやupdate.rbのファイル名も変更する必要があります。

### tdiary.confの作成

次に、tDiaryの設定ファイルであるtdiary.confを作ります。

初めてtDiaryをインストールする人には、付属のtdiary.conf.beginnerを使うのがオススメです。最初に使えるようにしておくと良いプラグインがあらかじめONになっていたり、spamフィルタにある程度の設定がされているなど、インストールしてすぐに使いやすい状態になっています。

tdiary.conf.beginnerをtdiary.confにリネームして、内容を書き換えます。tDiaryの設定はほとんどWebブラウザ経由で行えます。ただし、@data\_pathは必要に応じて最初に書き換えてください。

```
@data_path = 'data'
```

@data\_pathは、日記のデータを保存するディレクトリです。ほとんどのレンタルサーバで使われているApache HTTPサーバ向けには、このディレクトリをWWWサーバ経由で参照されないように.htaccessファイルによるアクセス制御を設定済みです。他のWebサーバを使用する場合には、WWWサーバ経由でアクセスできない(public\_html配下でない)ディレクトリを指定するか、アクセス制御を設定してください。また、このディレクトリはWWWサーバの権限で書き込めるパーミッションにしておく必要があります。

tdiary.confには、他にもいろいろな設定項目を記述できます。これらの項目には以下の3つの種類があります。

#### CGIで設定できない項目

@data\_pathのように、CGIでは設定できない項目です。これらの項目は、tdiary.confファイルを直接編集して変更しなければいけません。

#### CGIで設定できる項目

変更画面のメニューにある「設定」を開くと、ブラウザからtDiaryの設定を変更できます。ほとんどの項目はここから設定できるので、わざわざtdiary.confを手で書き換える必要はありません。

#### CGIで追加設定できるが、標準設定を記述できる項目

tdiary.confに記述しておくことで、CGIの設定画面からは編集できないが追加はできるといった設定をできる項目があります(リンク元記録除外や、リンク元変換表)。あらかじめtdiary.confに記述しておくことで、複数の人が同一サーバ上でtDiaryを使うような場合に手間を省くことができます。

各々の項目については、tdiary.conf.sampleの説明を読んでください。一般的な使用では、@data\_pathだけを正しく設定すれば、あとはブラウザから変更が可能です。

また、サフィックス.rbのファイルをCGIスクリプトとして指定できない環境では、index.rbやupdate.rbのファイル名を変更する必要がありますが、この変更をtDiaryに教えるために、@indexや@updateという変数が用意されています。環境によってはこれも指定する必要があるでしょう。

tdiary.confの設定が終わったら、http://www.hoge.example.org/~foo/diary/にアクセスしてみましょう。からっぽの日記ページが出れば設定は正しいです。不幸にして「Internal Server Error」が出てしまったら、Apacheのエラーログを参照して間違った設定を修正してください。

### セキュリティ

tdiary.confには、tDiaryの設定をCGIから行う場合のセキュリティレベルを調整する変数があります。自分が管理するWebサーバで、自分だけが使う場合にはあまり気にしなくても良いですが、知人に貸したり、tDiaryを使った日記サービスを提供するような場合には、ユーザにできることを制限したい場合が少なくありません。

そこで、tdiary.conf中でセキュリティの設定を行います。通常、tdiary.confの末尾には以下の2行が書かれています。

```
@secure = false
load_cgi_conf
```

@secureは、セキュリティ設定を指定する変数です。この値がfalseの場合、セキュリティチェックはいっさいかかりません。ユーザはCGI設定で好き放題ができます。それでは危険という状況下でtDiaryを運営する場合には、@secureの値をtrueにします。そうすると、CGI設定中における危険な変数操作やファイル操作が禁止されます。

また、@secureの値は、日記の表示時に後述するプラグインを実行する場合にも影響を及ぼします。これにより、@secureがtrueの場合には、いくつかのプラグインが利用できなくなります。

load\_cgi\_confはその位置でCGIによる設定を読み込む指令です。つまり、@secureでセキュリティレベルを設定したあとでファイルを読み込むようになっています。

なお、両者の指定位置は独立しているので、両者の位置を組み合わせることで様々な設定を行うことが可能です。また、@secureを指定しない場合のデフォルト値はtrueです。

tDiaryの実行
---------

### 日記の更新

ページの先頭には、「トップ」「更新」の2つのリンクがあります。「トップ」は@index\_pageで指定した表紙へ、「更新」は日記を更新するフォームへ移動します。もし「更新」をクリックした時、認証ダイアログが出なかったら、.htaccessの記述が正しくない可能性があります。

更新ページの先頭にもメニューがあります。一番右端に「設定」が増えているでしょう。ここをクリックすると、設定用のページが開きます。詳しくはを参照してください。

更新ページには、日記の日付とその日のタイトル、本文を入力するフォームがあります。日付、タイトル、本文を入力して「追加」ボタンを押すと、その日の日記に追加されます。タイトルと本文はどちらも省略可能です。追加なので、一日に何度も日記を書く場合に、わざわざ以前のデータを呼び出す必要はありません。また、すでにタイトルが指定されている場合、タイトルを入力しなければ以前指定したものが使われます。

フォームで日付を入力して「この日付の日記を編集」ボタンを押すと、(その日の日記がすでに存在すれば)タイトルと本文に過去の日記のデータが読み込まれます。この時、フォームの最後のボタンは「登録」になります(つまり、追加ではありません)。

日記本文には日記向けに少し特殊化したHTMLを使います。多少癖があって人によってはなかなか馴染めないことがあるようなので、[日記の書き方](HOWTO-write-tDiary.html)には必ず目を通して下さい。

### 日記の設定

更新画面で「設定」をクリックすると、設定画面になります。ここではtDiaryのさまざまな設定項目をブラウザから設定できます。各項目の説明は画面中に記述してありますから、それを参考にいろいろと設定を変えてみてください。また、ページ中には利用しやすくするために「OK」ボタンがたくさん置いてありますが、すべて同じものです。つまり、どの「OK」を押してもすべての項目が保存されます。

なお、この設定画面で行った変更は、@data\_pathで指定したディレクトリに別のtdiary.confとして保存されます(初期設定時に手動で書き換えたtdiary.confではありません)。このファイルは、元のtdiary.confのあとに読み込まれるので、設定の内容はブラウザから指定したものが優先されます(ただし、元のtdiary.conf中の設定を変えることで、読み込むタイミングは変更できます)。

### 日記の参照

日記の参照には、最新、月別、日別の3種類のモードがあります。デフォルトページは最新です。月別は、ページの最初の方に出るカレンダーをクリックすると参照できます。また、日別は日付をクリックすると参照できます。

最新・月別と日別には、表示される内容に違いがあります。最新・月別では「本日のツッコミ」「本日のリンク元」が省略されて表示されるのに対し、日別ではすべて表示されます。また、日別にはツッコミ用のフォームがあります。ツッコミをしてもらいたかったら、読者を日別ページに誘導するように、ヘッダ(@header)を工夫する必要があるかも知れません。

月、日、セクション、ツッコミには、それぞれアンカーがあり、他の場所からリンクできるようになっています。それぞれのアンカーはリンクにもなっているので、そこにポインタを合わせることで、そのURLを知ることができます。

携帯端末からの参照ではデータ量に制限があるため、上記の機能はほとんど使えません。最新は最新日付の日記だけが表示され、前日・翌日へ移動できるだけです。ただし、すべてのページにツッコミ用フォームが付いているので、ツッコミを入れることは可能です。

### プラグインによるカスタマイズ

tDiaryにはプラグインと呼ばれる機能があります。プラグインを追加することで、tDiaryの機能を増やしたり、変更したりすることが可能です。プラグインについての詳しい説明は、[HOWTO-use-plugin.html](HOWTO-use-plugin.html)(使い方)・[HOWTO-make-plugin.html](HOWTO-make-plugin.html)(作り方)を参照してください。

### あとは……

日記をつけ続けるだけです(これが一番難しい:-)。Have fun!!

著作権、サポートなど
----------

tDiary本体は、原作者であるただただし(t@tdtds.jp)が、GPL2の元で配布、改変を許可するフリーソフトウェアです。

また、tDiaryフルセットに付属するテーマ、プラグインはすべて、それぞれの原作者が著作権を有します。ライセンス等に関しては個々のファイルを参照してください。

tDiaryは[tDiary.org](http://www.tdiary.org/)でサポートを行っています。ご意見・ご要望はこちらへどうぞ。パッチ歓迎です。
