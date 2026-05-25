# Godot 4でNodeとSignalのつながりを確認するチェックリスト

Godot 4で小さな2Dゲームを作っていると、コードは正しそうなのに「動かない」「Signalが呼ばれない」「HUDが更新されない」という状態になりやすいです。

このページでは、コイン収集ゲームを作るときに確認しやすい順番で、Scene、Node、NodePath、Signalのチェックポイントをまとめます。

## 1. Sceneツリーの名前を確認する

Godotでは、コードからNodeを参照するときにSceneツリー上の名前を使うことがあります。

たとえば `$ScoreLabel` と書いている場合、同じScene内に `ScoreLabel` という名前のNodeが必要です。表示名が `Label` のままだったり、親Nodeの下に移動していたりすると、参照できません。

確認すること:

- Node名が本文やコード例と同じか
- 対象Nodeが同じScene内にあるか
- 親子関係が変わっていないか
- 大文字と小文字が一致しているか

## 2. Scriptが正しいNodeに付いているか確認する

コードが正しくても、Scriptを別のNodeに付けていると想定した動きになりません。

プレイヤー移動のコードは `Player`、コインの処理は `Coin`、ゲーム全体の進行は `Main` のように、役割に合ったNodeへScriptを付けます。

確認すること:

- InspectorのScript欄に正しい `.gd` ファイルが入っているか
- 同じScriptを誤って別Nodeに付けていないか
- Sceneを保存したあとに実行しているか

## 3. NodePathを短く決めつけない

`$HUD/ScoreLabel` のようなNodePathは、Sceneツリーの親子関係に依存します。

HUDを `CanvasLayer` に分けた場合や、LabelをPanelの中に入れた場合は、NodePathも変わります。

うまく参照できない場合は、Godotエディタで対象Nodeを右クリックし、NodePathをコピーして確認すると原因を見つけやすくなります。

## 4. Signalの接続先を確認する

`body_entered` や `timeout` などのSignalは、接続先Nodeと関数名が一致していないと呼ばれません。

確認すること:

- Signalタブで接続が残っているか
- 接続先Nodeが正しいか
- 生成された関数名をあとから変更していないか
- `Timer` の `timeout` がMainなど想定したNodeに接続されているか

コイン収集では、当たり判定そのものの設定も重要です。Area2D側の設定で迷った場合は、次のチェックリストも確認できます。

- [Area2Dの当たり判定が反応しない時のチェックリスト](godot-4-area2d-collision-checklist-ja.md)

## 5. HUD更新は表示Nodeと値の更新を分けて考える

スコアが増えているのに画面表示が変わらない場合、ゲーム内の値は更新されていても、Labelの `text` を更新していないことがあります。

確認すること:

- スコア用の変数は増えているか
- `ScoreLabel.text` のような表示更新も呼んでいるか
- HUDのNodePathが正しいか
- CanvasLayer内のLabelを参照しているか

## 無料で確認できるもの

本文はZennで無料公開しています。PDF/EPUBと章別サンプルZIPが必要な場合はGumroad版を利用できます。

- Zenn無料版: https://zenn.dev/batstudio/books/godot-2d-coin-collect-ja?utm_source=github_assets&utm_medium=node_signal_note&utm_campaign=godot_coin_ja_troubleshooting
- 共通アセット: https://github.com/gomsik/godot-2d-coin-collect-assets/releases/latest/download/coin_collect_assets.zip

## PDF/EPUBと章別サンプルが必要な場合

Zenn本文を読んだあと、PDF/EPUBで手元に置きたい場合や、途中で詰まったときに章ごとの完成サンプルZIPと比較したい場合はGumroad版があります。

- Zenn無料版とGumroad版の違い: [zenn-free-vs-gumroad-package-ja.md](zenn-free-vs-gumroad-package-ja.md)
- Gumroad版: https://batstudio.gumroad.com/l/arcxuq?utm_source=github_assets&utm_medium=node_signal_note&utm_campaign=godot_coin_ja_troubleshooting
