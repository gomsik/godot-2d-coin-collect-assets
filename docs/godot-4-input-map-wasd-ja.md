# Godot 4でInput Mapを使ってWASDと方向キーを同じ移動処理にする

Godot 4でプレイヤー移動を作るとき、最初に決めておくと後が楽になるのがInput Mapです。

キー入力をコードに直接書くこともできますが、`move_left` や `move_right` のようなアクション名を先に作っておくと、方向キーとWASDを同じ移動処理で扱えます。

このメモでは、2Dゲームの最初のプレイヤー移動に使いやすいInput Map設定と、GDScript側の最小コードを整理します。

## 追加するアクション

Project SettingsのInput Mapで、次の4つのアクションを追加します。

| Action | 割り当てるキー |
| --- | --- |
| `move_left` | Left Arrow, A |
| `move_right` | Right Arrow, D |
| `move_up` | Up Arrow, W |
| `move_down` | Down Arrow, S |

この名前はあとでコードから使います。`move_left` と `move-left` のように、1文字でも違うと別のアクションとして扱われるので注意します。

## 方向ベクトルを作る

Playerノードにスクリプトを付けて、入力から方向ベクトルを作ります。

ここでは、2Dコイン収集ゲームのPlayerと同じように、Playerルートを `Area2D` として作る前提にします。小さなサンプルとして、`position` を直接動かす形にしています。

```gdscript
extends Area2D

const SPEED := 260.0

func _process(delta: float) -> void:
    var direction := Vector2.ZERO

    if Input.is_action_pressed("move_left"):
        direction.x -= 1
    if Input.is_action_pressed("move_right"):
        direction.x += 1
    if Input.is_action_pressed("move_up"):
        direction.y -= 1
    if Input.is_action_pressed("move_down"):
        direction.y += 1

    if direction.length() > 0.0:
        direction = direction.normalized()

    position += direction * SPEED * delta
```

ポイントは、キーそのものではなくInput Mapのアクション名を見ることです。

たとえばLeft ArrowでもAキーでも、どちらも `move_left` として扱えます。コード側は `move_left` だけを見ればよいので、後からキー設定を変えるときもInput Mapだけを直せば済みます。

## 斜め移動の速度をそろえる

右と上を同時に押すと、方向ベクトルは `Vector2(1, -1)` になります。

このまま速度を掛けると、横だけ・縦だけの移動より少し速くなります。そこで、入力があるときだけ `normalized()` を呼び、方向ベクトルの長さを1にそろえます。

```gdscript
if direction.length() > 0.0:
    direction = direction.normalized()
```

入力がないときの `direction` は `Vector2.ZERO` です。何も押していない状態では移動方向がないので、長さを確認してから正規化しています。

## 画面外に出ないようにする

800 x 600の小さなゲームなら、移動後に位置を範囲内へ収めておくと扱いやすくなります。

```gdscript
position += direction * SPEED * delta
position.x = clamp(position.x, 32.0, 768.0)
position.y = clamp(position.y, 32.0, 568.0)
```

プレイヤーの見た目や当たり判定に合わせて、端の余白は調整します。

## よくある詰まりどころ

### Input Mapの名前が一致していない

`Input.is_action_pressed("move_left")` と書いた場合、Input Map側にも `move_left` が必要です。

`move left`、`move-left`、`left` は別の名前です。

### deltaを掛けていない

`position += direction * SPEED` だけにすると、フレームレートによって移動量が変わりやすくなります。

`_process(delta)` の `delta` を掛けて、1秒あたりの移動速度として扱います。

### 斜めだけ速い

上下左右の入力を足したあと、入力があるときだけ `normalized()` を呼びます。

これで、上下左右でも斜めでも同じ基準の速度になります。

## 関連する教材

このInput Map設定を含めて、空のGodotプロジェクトから2Dコイン収集ゲームを完成させる日本語の実習書としてまとめています。

- Zenn無料版: https://zenn.dev/batstudio/books/godot-2d-coin-collect-ja?utm_source=github_assets&utm_medium=tech_note&utm_campaign=godot_coin_ja_input_map
- Gumroad版: https://batstudio.gumroad.com/l/arcxuq?utm_source=github_assets&utm_medium=tech_note&utm_campaign=godot_coin_ja_input_map
