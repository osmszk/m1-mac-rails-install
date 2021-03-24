# M1 Mac上でRubyとRailsの開発環境構築やっていきます

## これはYouTube動画の資料です
- M1 MacにRubyとRailsの環境構築してみた youtu.be/z3pjrohuhec

---

## はじめに
- なるべく、プログラミング初学者向けにもわかりやすく解説していきます
- いっしょにやっていきましょう
- 2021年2月時点の情報です
  - M2のMacが出る頃(いつ?)には 古い情報になってるはずなのでお気をつけください

## 自己紹介：オサミー
- ソフトウェアエンジニア。株式会社プレジニア代表取締役。
- iPhoneアプリ開発歴10年。企画開発したiPhoneアプリ160万ダウンロード以上。
- 最近はiOS(Swift)アプリやフロントエンド(React, TypeScript)やってる。

## 動画(RubyとRails環境構築)の目次
- 理論編: Intel MacとM1 Macの違い 
- 実践編: RubyとRailsインストール

# 理論編
## 1.「M1 Mac」の環境は 「Intel Mac」の開発環境と大きく違うことに注意しよう！
- これは、「M1 Macでプログラミングする上で注意すべき」ポイント①
- 例えば、progateの記事「Rubyの開発環境を用意しよう！」だと、
  - https://prog-8.com/docs/rails-env
  - これは「Intel Mac」の情報
    - 「M1 Mac」では：rbenvでruby2.6.5をインストールできない(Stable Releaseのみ。2021年2月時点。)
    - 「M1 Mac」では：Homebrewはバージョン3.x以降ではないと動かない(armアーキテクチャ最適化バージョンは3.xから)
- RubyとRailsに関してはあんまり問題ないけど、「Intel Mac」の古い情報に注意しよう！
  - Intel Macとは：
    - Intelのチップ(CPU)が搭載されたMac
    - 古い
    - RubyやRailに関する記事はだいたい「Intel Mac」の情報　→　古い（気をつけて）2021年2月時点
  - M1 Macとは：
    - Apple社が設計したM1チップが搭載されたMac
      - ARM社がApple社へチップの回路図を提供してる
      - ので、M1チップのアーキテクチャ(設計方法)を「ARMアーキテクチャ」と呼ぶ
    - M1チップを「Apple Silicon」とも呼ぶ
    - 2020年11月に発売！（新しい）

## 2. Rosseta上なのかARMアーキテクチャ上なのか意識しよう！
- これは、「M1 Macでプログラミングする上で注意すべき」ポイント②
- いま動かそうとしてるプログラムは、「Rosseta上なのか」「ARMアーキテクチャ上なのか」意識すべし！
  - Rosetta上では動くが、ARMネイティブで動かないプログラムがある

- Rosettaって何？
  - M1 Macの特徴：Rosetta を使ってIntel Mac用のソフトウェアを使うことができる(例外あり)
  - Rosetta とは
    - 従来のインテル用のMacアプリを Apple Silicon Mac上で自動的に変換して実行できるようにする仕組み
    - 「Rosettaを使用してひらく」チェックボックスつけてアプリ起動すると、Rosetta上で動く
      - 例) ターミナル, Xcode, iTerm2 など
  - Rosetta使えば動くのか ARMネイティブ対応(M1最適化されてる)なのか ソフトウェア一覧まとめサイト
    - Is Apple silicon ready?
    - https://isapplesiliconready.com/

- いま「Rosseta上なのか」「ARMアーキテクチャ上なのか」確認できるコマンドは後述

---

# 実践編
## 実践編 目次
- 1.前提
- 2.rbenvでRubyインストール
- 3.RailsでHelloWorldまで

## 1.前提
### 1-1.インストール環境
- macOS BigSur 11.2.1
- MacBook Air(M1, 2020)
- Ruby 初期状態

### 1-2.前提: Ruby初期状態
- `ruby -v` -> `ruby 2.6.3p62 (2019-04-16 revision 67580) [universal.arm64e-darwin20]`
- `which ruby` -> `/usr/bin/ruby`
- `rbenv -v` ->  `zsh: command not found: rbenv`
### 1-3.前提: その他
- VSCode(Visual Studio Code)インストール済み
  - おすすめなエディタなのでインストール必須

### 1-4.Rubyの文法だけ試したい人はコチラがおすすめ！
- 文法だけ色々試したい場合は、「ブラウザで動く環境」がおすすめ！　
  - Rubyの開発環境を構築するだけでも 大変なので
  - ブラウザで動く環境（例）:
    - https://runrb.io/
    - https://repl.it/languages/ruby
    - 「文法学びたいだけの人」はこっちがおすすめ
- すでにRubyは最初からMacに入ってる！
  - ターミナルで `ruby -v` で叩くとバージョンが返ってくるよ
  - インタラクティブモードもあるよ(Interactive Ruby Shell)
    - ターミナルで、コマンド `irb` 叩くとはじまる
      - irbは Control + D で終了
- なのでRubyの文法だけ学びたい人 や ちょっとしたスクリプト書きたい人にとっては...
  - これから説明するrbenvとか必要なし

### 1-5.いま 「Rosetta上なのか」「ARMアーキテクチャ上なのか」意識することが重要！
- 確認コマンド:ターミナルで `uname -m` 打つ
  - `arm64`と出力 : ARMアーキテクチャ で実行中
  - `x86_64`と出力 : Rosetta利用 または ネイティブIntelアーキテクチャ で実行中
- オサミーの場合：
  - ARMアーキテクチャで実行したい場合「ターミナル.app」で実行
  - Rosettaで実行したい場合「iTerm2.app」で実行
  - VSCode上では適宜 `$ arch -x86_64 zsh` や `$ arch -arm64 zsh` で切り替えする

## ---------ここまで実践編の前提---------

## 2.rbenvでRubyインストール
### 目次
- 2-1. Command Line Toolsをインストール
- 2-2. Homebrewをインストール
- 2-3. rbenvをインストール
- 2-4. Rubyをインストール

### 2-1. Command Line Toolsをインストール
- ターミナルひらく
  - アプリケーション > ユーティリティ > ターミナル.app
  - 「Rossetaを使用して開く」これはチェックしないでひらく ＝ ARMアーキテクチャ上で実行する
- ターミナルでコマンド `xcode-select --install` 実行
- 「Command Line Tools」とは：
  - macOSのターミナルで、コマンド使うために必要なツール
  - Apple LLVM complier, Linker, Makeなどが内包されている
- Xcodeをインストールしている人は Xcode自体に「Command Line Tools」が含まれているのでスキップしてね
  - オサミー → Xcodeインストール済みなのでスキップ

### 2-2. Homebrewをインストール
- Homebrewとは：
  - macOSのパッケージマネージャー
- パッケージマネージャーて何よ？
  - パッケージを管理するソフトウェア
    - Homebrew以外にもいろいろある
      - 🔽例
      - npm: "Node Package Manager".  Node.jsパッケージの管理
      - yarn: npmの代替. npmより速い
      - gem: "RubyGems" Rubyパッケージの管理
      - pip: "Pip Installs Packages" Pythonパッケージの管理.
  - パッケージ？ = ある機能を有するソフトウェア. いろいろある
    - 「ライブラリ」とか「フレームワーク」とも呼んだりする
    - Homebrewのパッケージ例) rbenv, python@3.9, yarn...
    - Homebrewではパッケージを「formula」と呼んでる

#### 2-2-1.インストール方法
- ① https://brew.sh/ にアクセスする
- ② 「Install Homebrew」に記載しているコマンドをコピーする
  - 2021.2.19時点では `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
- ③ターミナルで コマンドを貼り付けて 実行(エンターを押す)
  - 何か聞かれたらすべてyesでOK
  - オサミーはインストール済みなのでスキップ

#### 2-2-2.brewコマンドが使えるか確認しよう
- `brew -v` を実行
  - 下記のようにバージョンが表示されたら成功
```
% brew -v    
Homebrew 3.0.1
Homebrew/homebrew-core (git revision 0b9a7; last commit 2021-02-18)
```
- `which brew` でどこにbrewコマンドいるのか確認しよう
```
$ which brew
/opt/homebrew/bin/brew
```

- `brew list` で「Homebrewでインストール済みのパッケージ一覧表示」しよう

#### 2-2-3.注意点：Homebrewの「バージョン」に気をつけて！
- バージョン 2.x はM1 Macには対応していないので注意
  - `brew -v` と叩いて 2.6とか表示されていたら、アップデートしよう
    - アップデート方法: `brew update`
- 3.0 でM1 Mac対応した！ 2021.2.5 リリース！
  - https://brew.sh/2021/02/05/homebrew-3.0.0/

#### 2-2-4.オサミーの場合： brewはインストール済み
- 自分は2.7.0を `/opt/homebrew` にインストールした
  - https://brew.sh/2020/12/01/homebrew-2.6.0/
  - インストール先は `/usr/local` ではなく 自分でつくった `/opt/homebrew`
  - `which brew` -> `/opt/homebrew/bin/brew`
  - `brew update` して 3.0.1に更新した

### 2-3. rbenvをインストール
#### 2-3-1.ruby-buildとともにrbenvをインストールしよう
- コマンド `brew install ruby-build rbenv` 叩いてインストールだ！
  - ruby-build と rbenv という2つのformulaをインストールする

- 余談：rbenvとは
  - いろんなRubyのバージョンをインストールできる環境をつくれるソフトウェア
  - 「Ruby version manager」 
  - https://formulae.brew.sh/formula/rbenv#default
  - ruby-buildに依存している
    - ruby-buildとは
      - 「Install various Ruby versions and implementations」
      - https://formulae.brew.sh/formula/ruby-build

#### 2-3-2.パスを通して初期化しよう
- ①`~/.zshrc` を編集しよう！2行追加しよう！
  - 2種類の編集パターン紹介
  - 方法①2つechoコマンド実行するパターン
    - ひとつめのコマンド：`echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshrc`
      - これを「パスを通す」と言う
    - ふたつめのコマンド：`echo 'eval "$(rbenv init -)"' >> ~/.zshrc`
      - これは初期化コマンド
  - 方法② エディタで`~/.zshrc`を編集するパターン
    - Finderで ホームディレクトリにいって、
        - Command + Shift + .(ドット) で隠しファイルを表示すれば .zshrc が現れる
        - なかったら .zshrc を新規作成しよう（エディタで新規ファイル作成 -> ".zshrc" という名前をつけて保存でOK ）
    - 下記2行直接書き込む
  ```
  export PATH="$HOME/.rbenv/bin:$PATH"
  eval "$(rbenv init -)"
  ```

- ② `~/.zshrc` を読み込もう
  - コマンド `source ~/.zshrc` で .zshrc を読み込んで rbenv初期化しよう
  - これ実行しないとrbenvコマンドを使えない

- 補足： `.zshrc` とは
  - `~/.zshrc` とは ターミナルひらくときに最初に読み込まれるファイル
    - 初期化コマンドや PATH通す処理や エイリアス(=別名,ショートカット)を記述
  - 余談：シェルが zsh ではなく bashだったら `.bashrc` に該当する
    - シェルとは：ターミナルでコマンド実行するソフトウェア(みたいなもの)
    - macOS BigSurからデフォルトで シェルが zsh になった(いままではbash)
  - コマンド `source ~/.zshrc` で .zshrc を読み込める

#### 2-3-3.rbenvコマンドを使えるか確認しよう
- コマンド `rbenv -v`
  - rbenvのバージョンを表示してみる
  - バージョンが表示されたら成功(rbenvコマンド使える)
    - 例：`rbenv 1.1.2`

- よく使う `rbenv`コマンド一覧
  - `rbenv versions`
    - いま何のバージョンのrubyがインストールされてるか一覧で確認
    - *マークついてるのが いま動いてる(設定されてる)Rubyのバージョン
  - `rbenv install -l`
    - インストールできるRubyバージョンを表示
  - `rbenv install {version}`
    - 指定のバージョンのRubyをインストール
    - 例：`rbenv install 3.0.0`
  - `rbenv global {version}`
    - 指定したRubyバージョン をmac全体で設定
    - 例：`rbenv global 3.0.0`
  - `rbenv local {version}`
    - いまいるディレクトリだけ指定Rubyバージョンに設定
    - 例：`rbenv local 3.0.0`
  - `rbenv rehash`
    - 更新

### 2-4. Rubyをインストール
- `rbenv install -l` 叩いて インストールできるRubyバージョンを確認しよう
- `rbenv install 3.0.0` 叩いて バージョン`3.0.0`をインストールしよう
  - 3〜5分くらいかかる
- `rbenv rehash` してrbenvを更新しよう

- `rbenv versions` 叩いて　rbenvでインストールされたrubyバージョン一覧を確認しよう
  ```
    %  rbenv versions
* system (set by /Users/osuzuki/.rbenv/version)
  3.0.0
  ```
- `rbenv global 3.0.0` 叩いて 3.0.0に切り替えよう
- `rbenv versions` 叩いて　rbenvでインストールされたrubyバージョン一覧を確認しよう
- `ruby -v` 叩いて 現在のrubyが3.0.0なのか確認しよう
  ```
  % ruby -v
  ruby 3.0.0p0 (2020-12-25 revision 95aff21468) [arm64-darwin20]
  ```
- `which ruby` 叩いて 現在のrubyコマンドの場所を.rbenv/下になってるか確認しよう
  ```
  % which ruby
  /Users/osuzuki/.rbenv/shims/ruby  
  ```
  - なってなかったら `rbenv rehash` や `source ~/.zshrc` 叩く
- `gem -v` でgemコマンド使えるかも確認しよう
  ```
  % which gem
  /Users/osuzuki/.rbenv/shims/gem
  ```
  - gemは rubyで書かれたパッケージの管理ソフトウェア
  - `gem list` でgemでインストールされたソフトウェア一覧確認
  - `gem install {package}` で指定したパッケージをインストールする
    - 例： `gem install rails` など

- 〜〜〜〜〜〜ここまででRuby 3.0.0 インストール完了！〜〜〜〜〜〜

## 3.RailsでHelloWorldまで
### 目次
- 3-1.railsをインストール
- 3-2.yarnをインストール
- 3-3.railsでプロジェクト作成
- 3-4.Hello World実行

### 3-1. railsをインストール
- `gem install rails` でインストールしよう！

- インストール成功か確認しよう
  - `gem list` でインストール済みパッケージ表示
  - bundlerもインストールされてる
  ```
  % gem list | grep rails
  rails (6.1.3)
  rails-dom-testing (2.0.3)
  rails-html-sanitizer (1.3.0)
  sprockets-rails (3.2.2)

  % gem list | grep bundler
  bundler (default: 2.2.3)

  % which bundle
  /Users/osuzuki/.rbenv/shims/bundle
  % bundle -v
  Bundler version 2.2.3
  ```

- `rails` コマンドが使えるか確認しよう
  - その前に、、 `rbenv rehash` してrbenvを更新しよう
  - `which rails`, `rails -v`の結果
```
% which rails            
/Users/osuzuki/.rbenv/shims/rails
% rails -v
Rails 6.1.3
```

### 3-2. yarnをインストール
- Rails 6から webpackerが必須になったのでまずはyarnをインストールする！
  - yarnというのはNode.jsのパッケージ管理ソフトウェア（パッケージマネージャー）
  - そのyarnを使って、webpackerをインストールする
    - `rails new <project_name>` 叩いたときに 勝手にyarnによってwebpackerがインストールされる
- yarnのインストール方法は主に2種類ある
  - https://classic.yarnpkg.com/en/docs/install#mac-stable
  - 方法1: Homebrewでインストールする
  - 方法2: npmでインストールする

#### yarnインストール方法①Homebrewでインストールする
- コマンド `brew install yarn`叩く
  - たぶん視聴者さんはこちらでOKだと思う...
  - もし Homebrewでyarnをインストールできなかったら コメントください
    - その場合→方法2のnpmでインストールしてください
      - Node.jsを[ここ](https://nodejs.org/ja/)からインストールしてから
      - `npm` コマンドが使えるはずなので、方法2のコマンドでyarnインストールしてください

#### yarnインストール方法②npmでインストールする
- コマンド `npm install --global yarn` 叩く
  - オサミーはnpmがインストール済みなのでこちら

#### yarnインストールできたか確認
- `yarn -v`　叩いてバージョン返ってきたらyarnインストール成功

### 3-3. railsでプロジェクト作成
- ①ホームディレクトリで `mkdir ruby_project` 叩いてフォルダ作成する
- ②`cd ruby_project` 叩いてカレントディレクトリを移動する("C"hange "D"irectory)
- ③`rails new hello_app` 叩いて 「hello_app」というrailsプロジェクトをつくる
  - いろいろファイルができる
  - VSCodeで確認してみよう
    - Finder上のフォルダごと VSCodeアプリにドラッグアンドドロップ

### 3-4. Hello World実行
- これからはVSCode 上のターミナルで実行してみよう
  - Command + J で「VSCodeの中のターミナルをひらく」
- まずは `uname -m` 叩いて ARMアーキテクチャで動いてるか確認しよう
  - もし `x86_64` と返ってきたらARMに切り替えよう. `arch -arm64 zsh` これで切り替えられる
  - `uname -m`　で `arm64`と返ってきたら ARM上で動いてる

#### 以下、ターミナル.appではなく VSCodeのターミナルで実行する
- `rails server` 叩いてローカルサーバー立ち上げてみよう
  - `http://127.0.0.1:3000` をブラウザでひらくとRailsアプリが表示される

- Control + C でローカルサーバーを止める

#### Hello World書き換えてみよう
1. `app/controllers/application_controller.rb` に追記しよう
```
  def hello
    render html: "Hello, world!"
  end
```

2. `config/routes.rb`に 追記しよう
```
  root 'application#hello'
```

- 再度 `rails s` で確認
  - `rails server` と同義

---

## お疲れ様でした！！！