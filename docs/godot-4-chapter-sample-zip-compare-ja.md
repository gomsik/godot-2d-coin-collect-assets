# Godot 4で章別サンプルZIPと自分のプロジェクトを比較する方法

Godotのチュートリアルで途中まで進めたあと、コードは同じように見えるのに動かないことがあります。

その場合、原因はスクリプト本文だけではなく、ノード名、シーン構成、Inspectorの値、Input Map、Signal接続、ファイルパスのどこかにあることが多いです。

このメモでは、章ごとの完成サンプルZIPを使って、自分のプロジェクトとの差分を確認する順番を整理します。

## 先に確認すること

まず、自分のプロジェクトをすぐに上書きしないでください。

おすすめは、次のように別フォルダで開くことです。

```text
my-coin-game/              自分が作業しているプロジェクト
chapter-sample-04-hud/     章別サンプルZIPを展開したプロジェクト
```

両方をGodotで開ける状態にして、画面を切り替えながら見比べます。

## 比較する順番

### 1. Sceneツリー

最初にSceneツリーを見ます。

確認するもの:

- ルートノードの種類
- 子ノードの名前
- 子ノードの階層
- `HUD/ScoreLabel` のようなパスが本文と一致しているか
- `Player`、`Coin`、`Timer` などの名前がコード内の参照と一致しているか

Godotでは、ノード名が1文字違うだけでも `$HUD/ScoreLabel` のようなパスは別物になります。

### 2. Scriptの付け先

次に、どのノードにどのスクリプトが付いているかを確認します。

例:

```text
Player  -> player.gd
Coin    -> coin.gd
Main    -> main.gd
HUD     -> hud.gd
```

コードが正しくても、別のノードに付いていると期待したタイミングで実行されません。

### 3. Inspectorの値

Inspectorで設定した値も見比べます。

特に確認するもの:

- CollisionShape2DにShapeが入っているか
- Area2DのMonitoringが有効か
- TimerのWait Time、One Shot、Autostart
- AudioStreamPlayerに音声ファイルが設定されているか
- Labelの表示位置とテキスト

本文では短く説明される設定でも、実際のプロジェクトでは1つ抜けるだけで動作が変わります。

### 4. Input Map

プレイヤーが動かない場合は、Project SettingsのInput Mapを見ます。

確認するアクション:

```text
move_left
move_right
move_up
move_down
```

コード側で `Input.is_action_pressed("move_left")` と書いているなら、Input Map側にも同じ名前が必要です。

### 5. Signal接続

コインを拾ってもスコアが増えない、Timerが終わってもゲームオーバーにならない場合は、Signal接続を確認します。

確認するもの:

- どのノードのSignalを使っているか
- 接続先ノードはどこか
- 接続先メソッド名がコード内に存在するか
- エディタ接続とコード接続が重複していないか

Signal周りで迷う場合は、次のチェックリストも使えます。

- NodeとSignalの確認チェックリスト: https://github.com/gomsik/godot-2d-coin-collect-assets/blob/main/docs/godot-4-node-signal-troubleshooting-ja.md?utm_source=github_assets&utm_medium=sample_zip_note&utm_campaign=godot_coin_ja_samples

## 差分を見るときの考え方

サンプルZIPは、答えを丸ごと置き換えるためだけのものではありません。

次のように、原因を切り分けるために使うと学習しやすくなります。

```text
動かない場所を決める
-> 該当する章のサンプルを開く
-> Sceneツリーを見る
-> Scriptの付け先を見る
-> Inspectorを見る
-> Signalを見る
-> 最後にコードを見比べる
```

コードを先に全部見比べるより、Godotエディタ上の構成から確認した方が早く原因に近づけることがあります。

## 章別サンプルZIPが役に立つ場面

- チュートリアルの途中で画面が動かなくなった
- エラーは出ていないのにスコアだけ更新されない
- コインとの当たり判定が反応しない
- HUDが表示されない、または表示位置がおかしい
- Timer後のゲームオーバー処理だけ動かない
- 自分のプロジェクトと完成状態の違いを確認したい

## 関連リンク

本文はZennで無料公開しています。

- Zenn無料版: https://zenn.dev/batstudio/books/godot-2d-coin-collect-ja?utm_source=github_assets&utm_medium=sample_zip_note&utm_campaign=godot_coin_ja_samples


PDF/EPUBと章ごとの完成サンプルZIPをまとめて受け取りたい方には、Gumroad版があります。

- Gumroad版: https://batstudio.gumroad.com/l/arcxuq?utm_source=github_assets&utm_medium=sample_zip_note&utm_campaign=godot_coin_ja_samples

この教材はGodot公式資料ではなく、BatStudioによる個人制作の日本語実習教材です。
