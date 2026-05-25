# Godot 4で小さな2Dコイン収集ゲームを完成させる学習順

Godot 4で最初の2Dゲームを作るときは、いきなり大きな機能を入れるより、小さな完成形まで順番に積み上げるほうが進めやすいです。

このページでは、2Dコイン収集ゲームを作るときの学習順と、このリポジトリにある短い技術メモをまとめます。

## 1. 入力を決める

最初にInput Mapを作ります。

方向キーとWASDを同じアクションにしておくと、コード側は `move_left` や `move_right` だけを見ればよくなります。

- [Input MapでWASDと方向キーを同じ移動処理にする](godot-4-input-map-wasd-ja.md)

## 2. プレイヤーを動かす

入力から `Vector2` の方向を作り、`SPEED * delta` で移動します。

斜め移動が速くなる場合は、入力方向を `normalized()` でそろえます。

- [2Dプレイヤーの斜め移動が速くなる問題を直す](godot-4-diagonal-speed-ja.md)

## 3. コインとの当たり判定を作る

`Player (Area2D)` と `Coin (Area2D)` を使う場合、コードだけでなく `CollisionShape2D`、`Shape`、Layer/Mask、Signal接続も重要です。

当たり判定が反応しないときは、順番に設定を確認します。

- [Area2Dの当たり判定が反応しない時のチェックリスト](godot-4-area2d-collision-checklist-ja.md)

## 4. スコアとHUDを表示する

スコアや残り時間は、ゲーム画面の上に表示するHUDとして分けると扱いやすくなります。

Godotでは `CanvasLayer` を使うと、ゲーム内の座標とUIの表示を分けやすくなります。

- Zenn記事: https://zenn.dev/batstudio/articles/godot-4-canvaslayer-hud-score?utm_source=github_assets&utm_medium=learning_path&utm_campaign=godot_coin_ja_learning_path

## 5. ゲームオーバーとリトライを入れる

最後に、`Timer` で制限時間を作り、時間切れになったらゲームオーバー表示を出します。

Rキーで `reload_current_scene()` を呼ぶと、小さな練習ゲームではリトライ処理を短く作れます。

- [ゲームオーバー表示とRキーのリトライを作る最小構成](godot-4-game-over-retry-ja.md)

## 5.5. コインをランダムな位置に出す

スコア加算まで動いたら、同じコインを画面内の別の位置へ移動させると、ゲームらしい流れになります。

最初はインスタンス生成よりも、既存の `Coin` ノードの `position` を変える形で確認すると扱いやすいです。

- [コインを画面内のランダムな位置に出す基本形](godot-4-random-coin-spawn-ja.md)

## 6. 動かない時はNodeとSignalを確認する

最後に、Sceneツリー上のNode名、Scriptを付けたNode、NodePath、Signal接続を確認します。

コードは正しそうなのにHUDが更新されない、Signalが呼ばれない、Nodeが見つからない場合は、次のチェックリストから見ると原因を分けやすくなります。

- [NodeとSignalのつながりを確認するチェックリスト](godot-4-node-signal-troubleshooting-ja.md)

## 無料で確認できるもの

本文はZennで無料公開しています。PDF/EPUBと章別サンプルZIPが必要な場合はGumroad版を利用できます。

- Zenn無料版: https://zenn.dev/batstudio/books/godot-2d-coin-collect-ja?utm_source=github_assets&utm_medium=learning_path&utm_campaign=godot_coin_ja_learning_path
- 共通アセット: https://github.com/gomsik/godot-2d-coin-collect-assets/releases/latest/download/coin_collect_assets.zip

## PDF/EPUBと章別サンプルが必要な場合

Zenn本文を読んだあと、PDF/EPUBで手元に置きたい場合や、途中で詰まったときに章ごとの完成サンプルZIPと比較したい場合はGumroad版があります。

- Zenn無料版とGumroad版の違い: [zenn-free-vs-gumroad-package-ja.md](zenn-free-vs-gumroad-package-ja.md)
- Gumroad版: https://batstudio.gumroad.com/l/arcxuq?utm_source=github_assets&utm_medium=learning_path&utm_campaign=godot_coin_ja_learning_path
