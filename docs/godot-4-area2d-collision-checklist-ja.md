# Godot 4でArea2Dの当たり判定が反応しない時のチェックリスト

Godot 4でコイン収集やアイテム取得を作るとき、`area_entered` や `body_entered` が呼ばれないことがあります。

コードを直す前に、まずScene、Inspector、Signal接続を順番に確認すると原因を見つけやすくなります。このメモでは、2Dコイン収集ゲームで `Player (Area2D)` と `Coin (Area2D)` を使う前提で、よくある詰まりどころをチェックリストにします。

## 想定するノード構成

ここでは、次のような小さな構成を想定します。

```text
Main (Node2D)
├─ Player (Area2D)
│  ├─ Sprite2D
│  └─ CollisionShape2D
└─ Coin (Area2D)
   ├─ Sprite2D
   └─ CollisionShape2D
```

`Player` と `Coin` の両方に `CollisionShape2D` が必要です。見た目の `Sprite2D` だけでは当たり判定は発生しません。

## 1. CollisionShape2DにShapeが入っているか

`CollisionShape2D` を追加しても、Inspectorの `Shape` が空のままだと判定されません。

コインなら `CircleShape2D`、プレイヤーも丸い画像なら `CircleShape2D` から始めると扱いやすいです。

確認する場所:

- `Player/CollisionShape2D`
- `Coin/CollisionShape2D`
- Inspector > `Shape`

`Shape` を作ったあと、半径が小さすぎないかも確認します。半径がほぼ0に近いと、画面上では重なっているように見えても判定しにくくなります。

## 2. MonitoringとMonitorableが有効か

`Area2D` には、他のAreaやBodyを検出するための設定があります。

確認する場所:

- `Player (Area2D)` または `Coin (Area2D)`
- Inspector > Collision > `Monitoring`
- Inspector > Collision > `Monitorable`

まずは、検出する側の `Monitoring` をオンにします。検出される側の `Monitorable` もオンになっているか確認します。

## 3. Collision LayerとMaskが合っているか

LayerとMaskが合っていないと、ノード同士が重なっても検出されません。

最初の小さなゲームでは、複雑に分ける前に次のような状態で確認します。

| ノード | Layer | Mask |
| --- | --- | --- |
| `Player (Area2D)` | 1 | 1 |
| `Coin (Area2D)` | 1 | 1 |

慣れてきたら、PlayerをLayer 1、CoinをLayer 2に分けて、PlayerのMaskにLayer 2を含めるようにします。

## 4. Signalを正しいノードに接続しているか

コイン側で `area_entered` を使うなら、`Coin (Area2D)` のSignalを `Coin` のスクリプト、または `Main` のスクリプトへ接続します。

例として、`Coin.gd` に接続する場合は次のような関数になります。

```gdscript
extends Area2D

signal collected

func _on_area_entered(area: Area2D) -> void:
    if area.name == "Player":
        collected.emit()
        queue_free()
```

GodotエディタでSignalを接続した場合、関数名が自動で作られます。コード内の関数名と、NodeタブのSignal接続先が一致しているか確認します。

## 5. area_enteredとbody_enteredを混同していないか

`Area2D` 同士を検出するなら `area_entered` を使います。

`CharacterBody2D` や `RigidBody2D` のようなPhysicsBodyを検出するなら `body_entered` を使います。

今回のように `Player (Area2D)` と `Coin (Area2D)` で作る場合は、まず `area_entered` から確認します。

## 6. ノード名に依存しすぎていないか

サンプルでは読みやすさのために `area.name == "Player"` と書くことがあります。ただし、ノード名を変えると判定も変わります。

少し安全にするなら、PlayerにGroupを付けて確認する方法もあります。

```gdscript
func _on_area_entered(area: Area2D) -> void:
    if area.is_in_group("player"):
        collected.emit()
        queue_free()
```

この場合は、`Player (Area2D)` を選択してNodeタブのGroupsから `player` を追加します。

## まず見る順番

当たり判定が動かないときは、次の順番で見ると切り分けやすいです。

1. `Player` と `Coin` の両方に `CollisionShape2D` があるか
2. `CollisionShape2D` の `Shape` が空ではないか
3. `Monitoring` / `Monitorable` が有効か
4. LayerとMaskが噛み合っているか
5. `area_entered` と `body_entered` を間違えていないか
6. Signal接続先と関数名が合っているか

## 関連する教材

この当たり判定を含めて、空のGodotプロジェクトから2Dコイン収集ゲームを完成させる日本語の実習書としてまとめています。

- Zenn無料版: https://zenn.dev/batstudio/books/godot-2d-coin-collect-ja?utm_source=github_assets&utm_medium=tech_note&utm_campaign=godot_coin_ja_collision_checklist
- 無料デモ: https://batstudio.gumroad.com/l/godot-coin-ja-demo?utm_source=github_assets&utm_medium=tech_note&utm_campaign=godot_coin_ja_collision_checklist
- Gumroad版: https://batstudio.gumroad.com/l/arcxuq?utm_source=github_assets&utm_medium=tech_note&utm_campaign=godot_coin_ja_collision_checklist
