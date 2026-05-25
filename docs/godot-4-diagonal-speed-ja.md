# Godot 4で2Dプレイヤーの斜め移動が速くなる問題を直す

Godot 4で2Dプレイヤー移動を作るとき、最初によく起きるのが「上下左右は普通なのに、斜め移動だけ少し速い」という問題です。

原因は、横方向と縦方向の入力をそのまま足したとき、斜め方向のベクトルが長くなるためです。このメモでは、`Vector2` と `normalized()` を使って、8方向移動の速度をそろえる基本形を整理します。

## 斜め移動が速くなる理由

左右だけを押した場合、移動方向は次のようになります。

```gdscript
Vector2(1, 0)
```

上だけを押した場合は、次のようになります。

```gdscript
Vector2(0, -1)
```

では、右上を同時に押した場合はどうなるでしょうか。

```gdscript
Vector2(1, -1)
```

このベクトルは、横に1、縦に1動くため、長さが1より大きくなります。そのまま速度を掛けると、斜め方向だけ移動量が増えます。

## 入力方向を作る

まず、入力から方向ベクトルを作ります。

```gdscript
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
```

ここでは、キー名を直接見るのではなく、`move_left` や `move_right` のようなInput Mapのアクション名を使っています。

たとえば `move_left` にLeft ArrowとAを両方登録しておけば、コード側は同じ `move_left` だけを見ればよくなります。

## normalizedで長さをそろえる

入力方向がある場合だけ、`normalized()` でベクトルの長さを1にそろえます。

```gdscript
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

    if direction.length() > 0:
        direction = direction.normalized()

    position += direction * SPEED * delta
```

この形にすると、上下左右でも斜めでも、最終的な移動速度が同じ基準になります。

## なぜlengthを確認するのか

`direction` が `Vector2.ZERO` のまま、つまり何も押していない状態では、移動方向がありません。

そのため、実際に入力があるときだけ `normalized()` を呼ぶ形にしています。

```gdscript
if direction.length() > 0:
    direction = direction.normalized()
```

この1段階を入れておくと、入力なし、上下左右、斜め入力のすべてを同じ流れで扱えます。

## 画面外に出ないようにする

小さな2Dゲームでは、移動後に画面内へ収める処理もよく使います。

```gdscript
position += direction * SPEED * delta
position.x = clamp(position.x, 32.0, 768.0)
position.y = clamp(position.y, 32.0, 568.0)
```

800 x 600の画面で、プレイヤーの見た目や当たり判定に少し余白を持たせるなら、このように上下左右の範囲を決めておくと扱いやすくなります。

## よくある詰まりどころ

### normalizedを毎フレーム必ず呼んでいる

入力がないときの `direction` は `Vector2.ZERO` です。実際に入力があるか確認してから正規化すると、読みやすくなります。

### deltaを掛けていない

`position += direction * SPEED` だけにすると、フレームレートによって移動量が変わりやすくなります。

`_process(delta)` の `delta` を掛けて、1秒あたりの移動速度として扱います。

### Input Mapの名前が一致していない

`move_left` と `move-left` は別の名前です。Project SettingsのInput Mapに登録した名前と、コード内の文字列が一致しているか確認します。

## 関連する教材

この移動処理を含めて、空のGodotプロジェクトから2Dコイン収集ゲームを完成させる日本語の実習書としてまとめています。

- Zenn無料版: https://zenn.dev/batstudio/books/godot-2d-coin-collect-ja?utm_source=github_assets&utm_medium=tech_note&utm_campaign=godot_coin_ja_player_move
- 無料デモ: https://batstudio.gumroad.com/l/godot-coin-ja-demo?utm_source=github_assets&utm_medium=tech_note&utm_campaign=godot_coin_ja_player_move
- Gumroad版: https://batstudio.gumroad.com/l/arcxuq?utm_source=github_assets&utm_medium=tech_note&utm_campaign=godot_coin_ja_player_move
