# Godot 4でコインを画面内のランダムな位置に出す基本形

コイン収集ゲームでは、プレイヤーがコインを拾ったあと、次のコインを別の場所に出す処理が必要になります。

このメモでは、Godot 4の小さな2Dゲームで扱いやすいように、画面サイズの内側にコインをランダム配置する基本形を整理します。

## 考え方

最初は難しく考えず、次の3つに分けます。

```text
画面の範囲を決める
-> 端から少し余白を取る
-> x と y をランダムに決めて Coin の position に入れる
```

プレイヤーやコインの画像が画面端で半分切れないように、少し余白を持たせるのがポイントです。

## ノード構成

例として、次のような構成にします。

```text
Main (Node2D)
├─ Player (Area2D)
└─ Coin (Area2D)
```

`Coin` は拾われたら消すのではなく、別の場所へ移動させる形にします。

## RandomNumberGeneratorを用意する

`Main.gd` に乱数用の `RandomNumberGenerator` を持たせます。

```gdscript
extends Node2D

const SCREEN_SIZE := Vector2(800, 600)
const SPAWN_MARGIN := 40.0

@onready var coin: Area2D = $Coin

var rng := RandomNumberGenerator.new()

func _ready() -> void:
    rng.randomize()
    move_coin_to_random_position()
```

`randomize()` を呼ぶと、ゲームを実行するたびに違う乱数になりやすくなります。

## コインをランダム位置へ動かす

x と y をそれぞれ範囲内で決めます。

```gdscript
func move_coin_to_random_position() -> void:
    var x := rng.randf_range(SPAWN_MARGIN, SCREEN_SIZE.x - SPAWN_MARGIN)
    var y := rng.randf_range(SPAWN_MARGIN, SCREEN_SIZE.y - SPAWN_MARGIN)
    coin.position = Vector2(x, y)
```

この例では、画面端から40px内側にだけコインが出るようにしています。

`SCREEN_SIZE` は自分のプロジェクトの画面サイズに合わせます。Project Settingsで800 x 600にしているなら、この値で確認しやすいです。

## コインを拾ったら再配置する

`Coin` から `collected` Signalを受け取ったあと、スコアを増やしてコインを移動します。

```gdscript
var score := 0

func _ready() -> void:
    rng.randomize()
    coin.collected.connect(_on_coin_collected)
    move_coin_to_random_position()

func _on_coin_collected() -> void:
    score += 1
    move_coin_to_random_position()
```

コインを `queue_free()` で消す形にしている場合は、まず再配置型にするか、新しいコインを生成するかを決めます。最初の練習では、同じ `Coin` ノードを動かすほうが分かりやすいです。

## 画面サイズを直接書きたくない場合

固定サイズではなくViewportからサイズを取ることもできます。

```gdscript
func move_coin_to_random_position() -> void:
    var rect := get_viewport_rect()
    var x := rng.randf_range(SPAWN_MARGIN, rect.size.x - SPAWN_MARGIN)
    var y := rng.randf_range(SPAWN_MARGIN, rect.size.y - SPAWN_MARGIN)
    coin.position = Vector2(x, y)
```

小さな入門ゲームでは、最初は固定値で動作を確認し、あとでViewportサイズを使う形に変えると理解しやすいです。

## よくある詰まりどころ

### 毎回同じ場所に出る

`rng.randomize()` を `_ready()` で呼んでいるか確認します。

### 画面の端に出すぎる

`SPAWN_MARGIN` を少し大きくします。コイン画像や当たり判定が大きい場合は、32pxや40pxより大きな余白が必要になることがあります。

### コインを拾った後に反応しなくなる

`queue_free()` でコインを消している場合、同じ `coin` 変数を使って再配置することはできません。

再配置するなら、コインは消さずに `position` を変えます。消して生成する方式は、インスタンス化を学んでからにすると整理しやすいです。

### Signal接続は動いているのに位置だけ変わらない

`coin` 変数が正しいノードを指しているか確認します。

```gdscript
@onready var coin: Area2D = $Coin
```

Sceneツリー上の名前が `Coin2D` や `CoinArea` なら、このパスも合わせる必要があります。

## 関連リンク

本文はZennで無料公開しています。

- Zenn無料版: https://zenn.dev/batstudio/books/godot-2d-coin-collect-ja?utm_source=github_assets&utm_medium=random_spawn_note&utm_campaign=godot_coin_ja_random_spawn


PDF/EPUBと章ごとの完成サンプルZIPをまとめて受け取りたい方には、Gumroad版があります。

- Gumroad版: https://batstudio.gumroad.com/l/arcxuq?utm_source=github_assets&utm_medium=random_spawn_note&utm_campaign=godot_coin_ja_random_spawn

この教材はGodot公式資料ではなく、BatStudioによる個人制作の日本語実習教材です。
