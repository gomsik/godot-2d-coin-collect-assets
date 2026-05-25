# Godot 4でゲームオーバー表示とRキーのリトライを作る最小構成

プレイヤー移動、コイン収集、スコア表示まで作れたら、次に足したいのは「ゲームとして終わる」処理です。

このメモでは、`Timer` で制限時間を管理し、時間切れになったらHUDにゲームオーバーを表示し、Rキーで現在のシーンを読み直す最小構成を整理します。

## 想定するノード構成

```text
Main (Node2D)
├─ Player (Area2D)
├─ Coin (Area2D)
├─ GameTimer (Timer)
└─ HUD (CanvasLayer)
   ├─ ScoreLabel (Label)
   ├─ TimeLabel (Label)
   └─ GameOverLabel (Label)
```

`GameOverLabel` は最初は非表示にします。Inspectorで `Visible` をオフにするか、`HUD.gd` の初期化処理で非表示にします。

## Input Mapにrestartを追加する

Rキーでリトライしたい場合は、Project SettingsのInput Mapで `restart` を追加します。

| Action | 割り当てるキー |
| --- | --- |
| `restart` | R |

コード内でキー名を直接見るより、`restart` というアクション名を見るほうが後から変更しやすくなります。

## HUD側に表示用の関数を用意する

`HUD (CanvasLayer)` に `HUD.gd` を付けて、スコア、時間、ゲームオーバー表示を更新する関数を用意します。

```gdscript
extends CanvasLayer

@onready var score_label: Label = $ScoreLabel
@onready var time_label: Label = $TimeLabel
@onready var game_over_label: Label = $GameOverLabel

func _ready() -> void:
    game_over_label.visible = false

func set_score(score: int) -> void:
    score_label.text = "Score: %d" % score

func set_time_left(time_left: int) -> void:
    time_label.text = "Time: %d" % time_left

func show_game_over(final_score: int) -> void:
    game_over_label.text = "Game Over\nScore: %d\nPress R to Retry" % final_score
    game_over_label.visible = true
```

HUDは見た目を担当し、ゲームの状態は `Main` 側で管理する形にすると整理しやすくなります。

## Main側でゲーム状態を持つ

`Main (Node2D)` に `Main.gd` を付け、ゲーム中かどうかを `is_game_over` で管理します。

```gdscript
extends Node2D

const START_TIME := 30

@onready var player: Area2D = $Player
@onready var game_timer: Timer = $GameTimer
@onready var hud = $HUD

var score := 0
var time_left := START_TIME
var is_game_over := false

func _ready() -> void:
    hud.set_score(score)
    hud.set_time_left(time_left)
    game_timer.wait_time = 1.0
    game_timer.start()
```

`GameTimer` は1秒ごとに残り時間を減らすために使います。Inspectorで `One Shot` はオフにしておくと、繰り返しtimeoutを受け取れます。

## Timerのtimeoutで時間を減らす

`GameTimer (Timer)` の `timeout` Signalを `Main.gd` に接続します。

```gdscript
func _on_game_timer_timeout() -> void:
    if is_game_over:
        return

    time_left -= 1
    hud.set_time_left(time_left)

    if time_left <= 0:
        finish_game()
```

時間が0になったら、ゲーム終了用の関数を呼びます。

```gdscript
func finish_game() -> void:
    is_game_over = true
    game_timer.stop()
    player.set_process(false)
    hud.show_game_over(score)
```

この例では、ゲーム終了後にPlayerの `_process()` を止めています。実際のプロジェクトでは、コインを非表示にする、効果音を鳴らす、入力を制限するなどを追加できます。

## Rキーで現在のシーンを読み直す

`_process()` で `restart` アクションを確認します。

```gdscript
func _process(_delta: float) -> void:
    if is_game_over and Input.is_action_just_pressed("restart"):
        get_tree().reload_current_scene()
```

`reload_current_scene()` は、現在開いているシーンを読み直します。小さな練習ゲームでは、ゲーム状態を最初から作り直すリトライ処理として使いやすいです。

## よくある詰まりどころ

### Timerが1回しか動かない

`GameTimer (Timer)` のInspectorで `One Shot` がオンになっていると、1回だけtimeoutして止まります。1秒ごとに残り時間を減らすなら、まずオフで確認します。

### Rキーを押しても反応しない

Input Mapに `restart` があるか、コード内の文字列が `restart` と一致しているか確認します。

### GameOverLabelが最初から表示される

`GameOverLabel` の `Visible` をオフにするか、`HUD.gd` の `_ready()` で `game_over_label.visible = false` にします。

### Signal接続先が違う

`GameTimer` の `timeout` は `Main.gd` に接続します。NodeタブのSignal一覧で、接続先ノードと関数名を確認します。

## 関連する教材

このゲームオーバーとリトライ処理を含めて、空のGodotプロジェクトから2Dコイン収集ゲームを完成させる日本語の実習書としてまとめています。

- Zenn無料版: https://zenn.dev/batstudio/books/godot-2d-coin-collect-ja?utm_source=github_assets&utm_medium=tech_note&utm_campaign=godot_coin_ja_game_over_retry
- Gumroad版: https://batstudio.gumroad.com/l/arcxuq?utm_source=github_assets&utm_medium=tech_note&utm_campaign=godot_coin_ja_game_over_retry
