2024/12 研究室進捗
=
## 12/10
- line 62 以降の、中心差分法の関数に実際に引数を与えて実行するとエラーが出る件について
  - GPhys objectにはgridとVArrayの情報が入っており、VArrayのような配列情報を取り出す
  - 但しこれでも、作った関数は1変数関数をその変数で微分することを想定して作っており、gp_Therm.valしたar_Thermは4次元配列なので、それをそのままdataとして引数に与えても処理できない
    - dataの次元を下げてあげる or 関数の部分で多変数関数をある変数で微分するような形に書き換える
  - dataの次元を下げるべく、GPhys::IO.openで呼び込むところで、timeについて特定のindexで切り出し、lonについて平均をとった(東西平均時間平均)
- まず、あるdataについて中心差分などで計算できるようにしてから、汎用できる関数に起こしていくのがよさそう
- dT_dyについて、いまdataは二次元まで下がっているので、もう一つ変数を固定してあげると計算できる。
  - line.88 の [true, plev_index] は lat_index を true としている
    - GPhys object ではなくarrayなので、indexを指定してやらないと動かない
- やるべきこと
  - 温度風のもう片方の微分
    - これはplevをzに直してあげる段階が必要
      - 厳密に変換していくか、スケールハイトを利用する
  - 両辺の微分結果を格納した配列ができるので、GPhys objectにするためにGrid情報を付加してあげる必要がある
  - 最終的に、温度の南北勾配と高度の圧力勾配をGPhys objectとして扱える状況で、どのような描画を得て確認するか
  - 中心差分の関数を改良する
    - 多次元dataを引数にとれるように
    - GPhys obejectを引数にとって、NArrayに変換して処理しGridを戻してGPhys objectを出力する、というところまでを関数にしてあげることもできる
  - JRA55と3Qの比較をする 
## 12/09
- line62以降を追加するとcentr_differ関数がエラー
  - これも下記と同様の理由 + 中心差分のdifferが非配列だとダメ
  - さらにgp_Thermは多次元配列状(配列とは違う)なので切り出さないと無理なのかも
- GPhys objectを加工できない問題
  - 結論：値を読み込ませなくてはならなかった
    ```
    ar_latitude=gp_latitude.val
    puts "ar_latitude: #{ar_latitude.inspect}"
    ```
    とすると結果は
    ```
    ar_latitude: NArray.float(37): 
    [ -90.0, -85.0, -80.0, -75.0, -70.0, -65.0, -60.0, -55.0, -50.0, -45.0, ... ]
    ```
    と数値配列が確認できた。
  - `delta_lat_rad = (gp_latitude[1] - gp_latitude[0]) * Math::PI / 180.0`を実行するとエラーが出る
  - `Shapes of grid and data do not agree. [] vs [1] (ArgumentError)`は、引数のエラーで、gridとdataの形状があっていないことを意味しているはず
  - いまnetcdfのなかの緯度(lat)が等間隔なので、この間隔を距離に直そうとしている。
  - ```gp_latitude```なる配列に格納されている値とクラスを確認したい。
    ```
    puts "gp_latitude: #{gp_latitude.inspect}"
    puts "gp_latitude class: #{gp_latitude.class}"
    ```
    `.inspect`は数値や配列の中身を文字列として表示してくれるという認識 [[参考]](https://docs.ruby-lang.org/ja/latest/method/Object/i/inspect.html)]
  - 結果は以下
    ```
    gp_latitude: <NumRu::GPhys grid=<1D grid <axis pos=<'lat' in './data/5_JRA55/t.2008-2009.nc'  float[37]>>>
    data=<'lat' in './data/5_JRA55/t.2008-2009.nc'  float[37]>>
    gp_latitude class: NumRu::GPhys
    ```
    1次元軸latであり、float型の37要素?、classはGPhysであるとわかり、これが軸情報と数値配列がセットになっているから配列のように`[要素番号]`で取り出すことができないっぽい
  - [[Dennou]](https://www.gfd-dennou.org/library/ruby/products/gphys/tutorial.old/)より
    >9行目(prs = gphys.coord(2).val)では, (ゼロから数えて)第2次元, 即ち圧力 (変数名level) の値を読み込む. VArray のメソッド val は データの値を NArray で返す. 従ってここでは (ここで初めて) ファイルからの変数値の読み込みが発生する.
## 12/08
- NArrayとは
- Error
  - ```wrong number of arguments (1 for 2) (ArgumentError) from ondoWind.rb:13:in `<main>'　```
    - 呼び出し側の引数の数と、メソッド側の引数の数が一致していないことを示すエラー　[[参考]](https://qiita.com/TOSHIMITSU_MIYACHI/items/82417ab6126d816af4e4)
    - ```GPhys::IO.open(file, varname)```であり、このメソッドは引数２つを期待しているのに、file_PATHしか記述しなかったことによるもの
    - [メソッドと関数の違い](https://wa3.i-3-i.info/diff97function.html)
## 12/05
- A について
  - 問題点
    - 温度風の式がどのように導かれるかの理解不足
    - ***圧力は等間隔ではない***
    - Narrayの理解不足
  - 改善案
    - 現在のcodeに対して、圧力を不等間隔で処理する必要がある
    - そもそも書き直した方がいいかも
      - ```begin-end```やエラー処理はしない方が初心者感があってよい
        - そもそもサンプルコードに忠実に書いた方がよい
  - 発展案
    - 微分法にバラエティを持たせて比較してみる
- B について
  - サンプルコードでデータをどのように見ているかを理解する
  - 少なくとも3Qの層を間引いて55と比較してみる
#### 月初現状
- A : 温度風が成立することの確認を試みる
  - 微分は中心差分法を用いて考える
  - rubyを使う
- B : JRA55と3Qの比較