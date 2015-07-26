# 装備品に関するメモ

ハクスラ等のシステムを作成するのに、理解が必要なのは武器・防具などの装備品。
データの保存のされ方、利用のされ方など理解してみる。

## 基本的なデータ構造

武器/防具のクラス階層と主な属性値は以下の通り。

RPG::BaseItem
* id, name, description, icon_index, note
* features 特徴リスト (RPG::BaseItem::Feature 配列)

RPG::EquipItem < RPG::BaseItem
* price 価格
* etype_id 装備タイプ
  * 0:武器,1:盾,2:頭,3:身体,4:装飾品

* params 能力値変化量 (整数配列)
  * 最大HP,最大MP,攻撃力,防御力,魔法力,魔法防御,敏捷性,運

RPG::Weapon < RPG::EquipItem
* wtype_id 武器タイプ ID
* animation_id アニメーション ID

RPG::Armor < RPG::EquipItem
* atype_id 防具タイプ ID

RPG::BaseItem::Feature
* code 特徴コード
* data_id 特徴の種類に応じたデータ (属性、ステートなど) の ID
* value 特徴の種類に応じた設定値

## RPG Makerで定義した装備品

Data/Weapons.rvdata2 に保存されている。実際のゲーム中で

    p $data_weapons[1]

を実行した結果、コンソールに出力されたのが以下。

    #<RPG::Weapon:0x7619430
      @description="Small sx used for harvesting wood",
      @name="Hand Ax",
      @icon_index=144,
      @price=100,
      @animation_id=7,
      @note="",
      @id=1,
      @features=[
        #<RPG::BaseItem::Feature:0x76192b4
          @code=31,
          @data_id=1,
          @value=0>,
        #<RPG::BaseItem::Feature:0x7619264
          @code=22,
          @data_id=0,
          @value=-0.1>,
        #<RPG::BaseItem::Feature:0x7619228
          @code=33,
          @data_id=0,
          @value=-5.0>],
      @params=[0,0,15,0,0,0,0,0],
      etype_id=0,
      @wtype_id=1>

RPG Makerのデータベースでみると 001:Hand Ax の特徴は
* 攻撃時属性 [Physical]
* 追加能力値 命中率 -10%
* 攻撃速度補正 -5

## ゲーム中の装備データ

ゲーム初期化時にパーティメンバーを Data/Actors.rvdata2 から読み込む。
$data_actors[1] の内容は以下。

    #<RPG::Actor:0x75e34fc
      @Name="Alex"
      @face_index=0,
      character_index=0,,,
      @equips=[1, 46, 0, 1, 0],,,
      @max_level=99>

@equips が装備品で、RPG::EquipItem の装備タイプ(etype_id)に対応するようだ。
またプレイ中に $game_party の中身は

    #<Game_Party:0x77dc9c0
      @in_battle=false,
      @gold=0,
      @steps=0,,,
      @actors=[1],
      @items={},
      @weapons={},
      @armors=[]>

で、最初のキャラの装備を外すと以下に値が変化した。

    #<Game_Party:0x77dc9c0
      ,,,
      @weapons={1=>1},
      @armors={46=>1, 1=>1}

weapons, armors にはそれぞれの装備のIDと個数が格納されるようだ。
装備すると持ち物(items)データからは削除されることがわかる。

## ここまでのまとめと検討

武器・防具など装備品のデータは静的なもので、ゲーム開始時に読み込まれる。
ゲーム中に所有もしくは装備している装備品は、そのIDと個数で管理されている。
セーブされるのはこのIDと個数の情報だけである。


これは非常にシンプルで合理的な仕組みだが、Diablo系RPGにあるようなアイテムごとの特徴を表現できないことがわかる。
Hand Axe に何か特徴(feature)を追加すれば、ゲーム中に出てくる全ての Hand Axe がその特徴を持つことになります。
今回入手した Hand Axe だけが特別、という処理はこのままでは難しい。


例えば火の属性をもった Hand Axe of Fire という特殊な武器を実装することを考えてみよう。
Hand Axe のRPG::Weaponデータに feature を追加するだけだと、全ての Hand Axe に影響が出てしまう。
もし feature として実装するのであれば

* $game_party の属性 items, weapons に格納する値を拡張する

がまず考えられる。
オブジェクト指向言語でクラス属性に対してインスタンス属性があるように、個々のアイテムにも features の属性を付与する感じになる。
ベタなやり方だと、@weapon_featuresのような配列を用意し、追加のfeatureを個別管理する、など。

しかしこの方法にはいくつか課題もある。
わかりやすいのは他の Hand Axe との取扱いをどうするか、だ。
アイテム一覧で他の Hand Axe と一緒に表示すると利用者に混乱が生じる。
アイテム表示や装備画面など、いろいろ対する必要がありそうだ。


よってRGSS3(RPG Maker VX Ace)でユニークな装備品を実装するには

* 固有のIDを持った別の装備品として実装する

のが簡単そうだ。
具体的には Hand Axe のRPG::Weaponデータをコピーして、別IDのデータとして追加する。
そしてそれに火のfeatureを加え、名前を Hand Ax of Fire に変更すればOKだ。


作成した追加の武器は、当然だがセーブデータに含める必要がある。
[2014/06 My 1st study](201406-1st-study.md "2014/06 My 1st study") に記載したセーブデータの拡張を応用すれば実装は可能。
データの連携はアドレスではなくIDベースのようなので、処理に問題は出ないはず。
ゲーム開始直後には持っていない(持っているならば普通に定義しておけば良い)ものなので、セーブデータの整合性も問題ないだろう。


この方法の懸念点は、利用できるIDの上限だ。
システム的には配列だし、ゲーム設定で最大数を変更できるのでコントロールは可能におもえる。
ただし武器データをセーブデータに含むので、持ち物の最大数を制限する必要はありそうだ。


またセーブデータには武器のデータそのままではなく、差分だけを格納するのも良さそうだ。
Hand Ax of Fire であれば、元のIDと追加する feature ひとつ、新しい名称さえ得られればゲーム用データを再生成できる。
