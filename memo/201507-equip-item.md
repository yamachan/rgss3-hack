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
