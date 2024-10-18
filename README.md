進捗メモ
=
## 10/18
git及びgithubの設定と、gphysの導入
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
    - これらは紐付けたいrepositoryの位置で行う
    - 以下ではzコマンドを使えるようにした
     ``` git clone https://github.com/rupa/z.git ~/z```
  - githubの設定(web登録)
    - user nameとpassを設定
  - 留意事項
   .mdファイルは任意のエディタで作ってよい(現在、vimとemacsだとなぜかクラッシュする)
   ~/gitの中に.mdファイルを記述し、git設定の項に従って更新
    この際、user nameとpassが要求される
    ここでのpassはgithubの設定で生成するpersonal access tokenで、権限付与が必要(一旦repoのみ付与した)
- gphys導入(wsl)について
  - gphysはrubyで作られている
  - rubyの環境構築とnetcdf、hdf5というファイル形式をrubyで
    扱えるようにする必要がある
    - HDF5 : Hierarchical Data Format
    - netCDF : Network Common Data Form
  - 問題点
    - これらのバージョンが原因でgphysが起動できないとみられる
    - 現状、gphysはrubyでrequireできるがnetcdfを読み込まない状況
- gphys の導入に際するトラブル
- 広義コンパイラと狭義コンパイラがある
- プリぷろっせさー、りんかをふくめたのが広義コンパイラ
- rubyの環境構築とhdf5、netcdfを扱えるよう設定、gphysの導入
- rubyの環境構築
  - rbenvを導入して、複数のバージョンのrubyを利用できるようにした
  - bundlerを導入して、適切な依存関係のパッケージを導入できるようにした
  - Gemfileに以下を記述
      source "https://rubygems.org"
      gem 'hdf5' 
      gem 'gphys', '1.5.6'
      ~~gem 'ruby-netcdf'~~　#これはgphysに入っているので不要
      gem 'numo-narray'
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
  - 発生した問題と解決策
    - gemをupdateしたあと、bundle installを実行すると以下のエラーが発生
      ```missing required library to compile this module: No such file or directory - gsl-config *** extconf.rb failed ***```
      - ```sudo apt install libgsl-dev```でgslを手動で入手
    - 再度bundle installすると、エラーが発生
      - エラーを見るとC言語まわりの記述みられており、なんらかのパッケージの中身と思われる。一例として
       ```include/rb_gsl_array.h:57:1: error: unknown type name ‘EXTERN’ 57 | EXTERN VALUE cgsl_matrix_complex_view_ro;```
       や
       ```cc1: note: unrecognized command-line option ‘-Wno-self-assign’ may have been intended to silence earlier diagnostics```
       - 現行のC言語にはexternはあるがECTERNはなく、過去にあった
         また、c1はCのプリプロセッサ
         = c1(gcc)のバージョンが問題か
       　参考: [Githubのこの件と同じ質問](https://github.com/SciRuby/rb-gsl/issues/69)
       　→ 参考サイトの回答に[Ruby3 compatibility](https://github.com/SciRuby/rb-gsl/pull/66)があり、開くと  Changes EXTERN to externと書かれていた。
       　→ 解決策は、回答の通りだと難しいので、rubyのバージョンを2.2.0に下げて、bundle installを実行した。
       　→ bundlerを使うとはねられるので、gem installした
       　→ 成功