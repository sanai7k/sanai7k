進捗メモ

=
## 10/29
- Python
  - how to define function
	- 
	```
	def func_name(argument)
	```
	で
	
## 10/24
emacs setting
- no window modeで処理落ちやクラッシュの問題は未解決
- window modeで日本語が文字化けする理由はxming側の問題だった
  - ``` sudo apt-get install xfonts-intl-japanese```で解決
## 10/22
emacsの設定、pythonの基本プログラム、gphys周りの整備
#### emacsの設定
- `emacs .emacs &`でemacsの設定ファイルを開き、
  - ```
    (setq␣initial-frame-alist
    (append␣(list
              ‘(width␣.␣80)
              ‘(height␣.␣55)
              )
      initial-frame-alist))
    (setq␣default-frame-alist␣initial-frame-alist)
    ```
    を追加することで、起動時のdefaultのwindowサイズを変更
#### emacs,vimでmd fileを編集するとクラッシュする問題
- intuition: if edit in japanese, they'll be crashed.
- 直感：日本語で編集するとクラッシュする。
  - 最終的な結論は、この文章を書いている途中でクラッシュしなければ、xmingの設定でフォントサポートができていなかったこと。
  - xmingはGUI利用のためのソフトウェアであり、emacs等を-nw optionなしに起動すると開くGUIのwindowはxmingを利用している
  
- pythonについて
  - 
- gphys周りの整備
  - gslなるlibraryをrubyで呼び出せるようにrb-gslをgem installした。
## 10/21
#### C言語
変数の寿命、配列、文字列、アドレス、ポインタについて
- 変数の寿命
  - global,local,static-localによって適用範囲が異なる
  - local > globalである
- 配列を用いれば複数の変数をまとめて扱える
- for文はloop命令
  - for (始点;終点;始点から終点までの変化法)
- 文字列は扱いが厄介
  - 場合によって新たなヘッダーファイルをincludeする
- アドレスについて
  - 変数はメモリに保存されている
  - メモリは最小単位bitごとに0,1の状態があり、8bitでもって1byteなる単位をなし、1byteを基本単位として情報を記憶している
  - 各byteには16進法のアドレスが割り当てられている
  - 変数はその型ごとに適当な量のbyteが割り当てられ、その保存領域の先頭のbyteのアドレスが変数の保存領域のアドレスとして認識される
  - &はアドレスを求める演算子である
    - アドレスを知る必要性
      - 値渡しについて、関数の引数として変数を指定した際、実際に渡されるのは変数の中身の何らかの実数なり整数であり、変数それ自体を変数として渡すわけではない
      - これは、変数それ自体を変更しないことを意味するため、変数の中身を変更しても変数は変更されないことを意味する。
      - 変数それ自体を変更したい場合は、変数の保存されているアドレスを指定して書き換える。
      - scanf関数で&を指定する理由はまさに、&(変数)とすることで変数のアドレスを指定しその中身それ自体を変更するためである。
      - 文字列入力時に&を使わなかったのは、そもそも配列名とは配列先頭要素の先頭byteのアドレスを表しているため。
- ポインタについてその１
  - 関数の引数として、変数ではなく変数のアドレスを渡せば、変数の中身を変更できる。
  - 引数として、アドレスの値を記憶できる変数を指定したい
  - ポインタは３つの機能の総称
    - ポインタ型
      int型と同様の型の種類で、それらと異なる特徴がある。
    - ポインタ値
      integer値やreal number値と同様な、アドレス値。ポインタ型で扱える数値、すなわちアドレスのこと
    - ポインタ変数
      アドレスを記憶する変数のこと。int型の変数などと基本的には同じ
## 10/19
#### git及びgithubの諸々について
- README.mdだとプロフィールに進捗が掲載される問題
  - ユーザーレポジトリ内のREADME.mdは掲載されるので、ファイル名を変更
    - 方法１(推奨)
      ```
      git mv old_filename.md new_filename.md #ローカル上でファイル名変更
      git commit -m "Rename old_filename.md to new_filename.md" #変更をcommit
      git push origin master #変更をpush
      ```
    - 方法２
      githubで名称変更し
      ```
      git pull origin master #変更をpull
      ```
      その後、localの名称を手動で変更
- new_file.mdの追加
  - wslの該当dir内で
    ```
    touch new_file.md #ファイル作成
    git add new_file.md #変更をステージ
    git commit -m "Add new_file.md" #変更をcommit
    git push origin master #変更をpush
    ```
- ローカルとリモート間の更新差が生じた場合
  - 以下のエラーについて
    ```
    ! [rejected]        HEAD -> master (non-fast-forward)
    error: failed to push some refs to 'https://github.com/sanai7k/sanai7k.git'
    ```
    ローカル上で、リモートの変更をpullしてからpush
    ```
    git pull origin master  
    git push origin master
    ```
  - 上記のpullで以下のエラーが出るとき
    ```
    fatal: Need to specify how to reconcile divergent branches.
    ```
    エラーには、リモートとローカルのmasterブランチの間に差異があり、どのように統合するかを指定する必要がある
    マージを利用して解消
    ```
    git pull --no-rebase origin master
    ```
    これでconflictが起きなければ、pushできる

#### rubyについて
- [依存関係を調べられるサイト](https://rubygems.org/gems/gphys/versions/1.5.6/dependencies)
- エラー例
  ```
  Building native extensions. This could take a while...
  ERROR: Error installing zsteg:
  ERROR: Failed to build gem native extension.
  (中略)
  RUBYLIBDIR=/var/lib/gems/2.3.0/extensions/x86_64-linux/2.3.0/rainbow-2.2.2
  /usr/bin/ruby2.3: No such file or directory -- /usr/share/rubygems-integration/all/gems/rake-12.0.0/exe/rake(LoadError)
  rake failed, exit code 1
  Gem files will remain installed in /var/lib/gems/2.3.0/gems/rainbow-2.2.2 for inspection.
  Results logged to /var/lib/gems/2.3.0/extensions/x86_64-linux/2.3.0/rainbow-2.2.2/gem_make.out
  ```

  No such fileの文からrakeなるgemがないことがわかるので

  ```
  gem install rake
  ```

  で入れてあげる。[参考](https://b1nary.hatenablog.com/entry/2017/10/06/175642)

#### gphysについて

代替の可能性がありそうな、netcdf可視化ツール

- [ncvis](https://github.com/SEATStandards/ncvis)
- [panoply](https://www.giss.nasa.gov/tools/panoply/)

## 10/18

#### git及びgithubの設定と、gphysの導入

- gitとgithubの設定
  - gitとは
   差分管理システムで、簡単に言うと過去の変更履歴を保持してくれる
  - gitの設定(wsl)

    - ```
      git init (初回のみ)
      git add -A
      git commit -c"任意のコミットメッセージ"
      git remote add origin https://github.com/お前のアカウント名/お前のレポジトリ名(初回だけ)
      git push origin HEAD
      これらは紐付けたいrepositoryの位置で行う
    - gitconfigに

      ```
      git config user.name  < my name >
      git config user.email < my email >
      ```

      を加える。
    - 403errorが出たら```git config credential.helper```を入力。[参考](https://qiita.com/mtc465/items/c2e8472f797e9f5bbc43)
  - githubの設定(web登録)
    - user nameとpassを設定
  - 留意事項
   .mdファイルは任意のエディタで作ってよい(現在、vimとemacsだとなぜかクラッシュする)
   ~/gitの中に.mdファイルを記述し、git設定の項に従って更新
   この際、user nameとpassが要求される
   ここでのpassはgithubの設定で生成するpersonal access tokenで、権限付与が必要(一旦repoのみ付与した)
- rubyの環境構築
  - rbenvを導入して、複数のバージョンのrubyを利用できるようにした
  - bundlerを導入して、適切な依存関係のパッケージを導入できるようにした
    (追記 1019) [bundler,Gemfileについて](https://qiita.com/nishina555/items/1b343d368c5ecec6aecf)
  - Gemfileに以下を記述
      source "<https://rubygems.org>"
      gem 'hdf5'
      gem 'gphys', '1.5.6'
      ~~gem 'ruby-netcdf'~~　#これはgphysに入っているので不要
      gem 'numo-narray'
- gphys導入(wsl)について
  - gphysはrubyで作られている
  - rubyの環境構築とnetcdf、hdf5というファイル形式をrubyで扱えるようにする必要がある
    - HDF5 : Hierarchical Data Format
    - netCDF : Network Common Data Form
  - 発生した問題と解決策
    - gemをupdateしたあと、bundle installを実行すると以下のエラーが発生
      ```missing required library to compile this module: No such file or directory - gsl-config *** extconf.rb failed ***```
      - ```sudo apt install libgsl-dev```でgslを手動で入手
    - 再度bundle installすると、エラーが発生
      - エラーを見るとC言語まわりの記述みられており、なんらかのパッケージの中身と思われる。一例として
        - ```include/rb_gsl_array.h:57:1: error: unknown type name ‘EXTERN’ 57 | EXTERN VALUE cgsl_matrix_complex_view_ro;```
         → 現行のC言語にはexternはあるがEXTERNはなく、過去に存在
        - ```cc1: note: unrecognized command-line option ‘-Wno-self-assign’ may have been intended to silence earlier diagnostics```
         → cc1はCのプリプロセッサかつ狭義コンパイラ
        - 結論: cc1(gcc)のバージョンが問題か
         参考: [Githubのこの件と同じ質問](https://github.com/SciRuby/rb-gsl/issues/69)
         → 参考サイトの回答に[Ruby3 compatibility](https://github.com/SciRuby/rb-gsl/pull/66)があり、開くと  Changes EXTERN to externと書かれていた。
         → 解決策は、回答の通りだと難しいので、rubyのバージョンを2.2.0に下げて、bundle installを実行した。
         → bundlerを使うとgphysがはいらないので、gem installした
         → 起動成功

## コマンドまとめ

- ubuntuのコマンドまとめ
  - ```bundle install --verbose```
   bundlerがインストールの際に行っている処理を詳細に表示させるoption
  - ```gem uninstall -I -a -x --user-install --force```
   gemをアンインストールするコマンドに
    - ```-I```:依存関係を無視する
    - ```-a```:複数のバージョンがあればすべて
    - ```-x```:gemによってインストールされた実行可能ファイル(バイナリ)も含める
    - ```--user-install```:ユーザーローカルにインストールされたgemのみに対象を絞る
    - ```--force```:強制
  - ```bundle install --redownload```
   既にキャッシュされているgemを無視してすべてのgemをダウンロードする。破損等の上書き。
  - ```gem update --system```
    - RubyGems のバージョンを最新のものにする。
    - 新しい機能やバグ修正を含んだ最新のRubyGems を利用できるようにする。
    - 新しいバージョンの gem に対応できるようにする。
    - 以下ではzコマンドを使えるようにした
      ```git clone https://github.com/rupa/z.git ~/z```
  - ```dpkg -l|grep paclage_name```
    - ```dpkg```:debian packageの意、パッケージに関することをする 
    - ```-l```:list
    - ```|```:パイプ、前のコマンドの出力を次のコマンドの入力として渡すための記号
    - ```grep```:テキスト検索ツール、後ろに入力した言葉を検索する
## コンピュータに関する雑記

- 広義コンパイラと狭義コンパイラがある
   プリプロセッサ、狭義コンパイラ、アセンブラ、リンカのまとまりが広義コンパイラという
