# 第4章 Your first 3D game（Squash the Creeps）

3Dの世界を動き回り、ジャンプして敵を踏みつぶすゲームを作ります。第3章の2Dゲームと似た作りなので、前の章の内容を思い出しながら進められます。

**作るもの**: 8方向に動いてジャンプするプレイヤー / ランダムに現れる敵 / 踏みつぶすとスコア加算 / 敵に当たるとゲームオーバー、Enterでリトライ / BGM / 浮遊アニメーション

**学ぶこと**: 3Dの座標、`CharacterBody3D` での移動と衝突、物理レイヤーとグループ、敵の定期的な出現、アニメーション、3Dゲームでのスコア表示

> 前提: 第2章まで終えていること（第3章は未経験でも進められますが、先に2Dをやる方が楽です）。3Dのコードは2Dより少し複雑です。完成版: https://github.com/godotengine/godot-demo-projects/tree/master/3d/squash_the_creeps

> **2Dと3Dの違いの目安**: 距離の単位は**メートル**（2Dはピクセル）。Y軸は**上がプラス**（2Dは下がプラス）。地面は**XZ平面**です。

---

## パート1: ゲーム領域の準備

### 1-1. 素材のプロジェクトを開く
1. 3d_squash_the_creeps_starter.zip をダウンロードして展開する（URLは「はじめに」）
2. プロジェクトマネージャーで **Import** をクリック
3. 展開したフォルダの `project.godot` を選ぶ（Browse ボタンから）
4. **Import** をクリック
5. 「古いバージョンで作られた」という警告が出たら **OK**（現在のバージョンに変換される）。すぐ開かなければ、一覧から開く

✅ FileSystem に `art/`、`fonts/`、`icon.svg` がある。`art/` に `player.glb`、`mob.glb`（3Dモデル）、マテリアル、BGM が入っている。

> `.glb` は、3Dシーンを保存する、オープンな形式（glTF 2.0）のファイルです。

### 1-2. メインシーンの土台
1. Scene ドックの **Add Child Node（＋）** → **Node** をダブルクリックして作成
   - 名前の変更: ノードを右クリック > Rename、または **F2**
2. 名前を `Main` にする
3. **Ctrl+S** で `main.tscn` として保存

### 1-3. 地面
1. Main を選び、**StaticBody3D**（動かない当たり判定）を追加
2. その子に **CollisionShape3D** を追加
3. StaticBody3D の名前を `Ground` に変える

```
Main
└─ Ground (StaticBody3D)
   └─ CollisionShape3D
```

4. CollisionShape3D に警告が出ている（形がない）。CollisionShape3D を選び、Inspector の **Shape** `<empty>` → **New BoxShape3D**
5. BoxShape3D をクリックして展開し、**Size** を **X=60、Y=2、Z=60** にする

当たり判定は見えません。見た目の床を作ります。
6. Ground を選び、子に **MeshInstance3D** を追加
7. Inspector の **Mesh** → **New BoxMesh**
8. BoxMesh をクリックして展開し、**Size** を **60、2、60** にする

✅ 灰色の大きな板が、グリッドを覆う。

地面を1m下げて、グリッドを見えるようにします。
9. 3D 画面で、ツールバーの **Use Snap** ボタン（または **Y** キー）でスナップをオンにする
10. **Ground ノード**を選び、Y軸（緑の矢印）をドラッグして、**1m下げる**
    - ⚠ 動かすのは **Ground**。MeshInstance3D や CollisionShape3D ではありません
    - 矢印（移動ギズモ）が見えないときは、ツールバーの **Transform Mode** を選ぶ
11. Ground の Inspector の **Position > Y** が `-1` になっていればOK

### 1-4. ライト
1. Main を選び、**DirectionalLight3D** を追加
2. ギズモで、上へ動かし、赤い円弧を使って、X軸まわりに回転させ、地面が照らされるようにする
3. Inspector の **Shadow** にチェック（影をオン）

✅ ここまでが出発点。次はプレイヤー。

---

## パート2: プレイヤーのシーンと入力アクション

### 2-1. シーンの構成
1. メニュー **Scene > New Scene**
2. **Other Node** から **CharacterBody3D** を作り、名前を `Player` にする
   - CharacterBody3D は、**自分のコードで動きを決める**キャラクター用の物理ノード（RigidBody は物理エンジン任せ）
3. Player の子に **Node3D** を追加し、名前を `Pivot` にする
   - あとで、アニメーションと独立して、モデルを回転させるための土台
4. FileSystem で `art/` を開き、**`player.glb` を Pivot へドラッグ＆ドロップ**
5. できた子ノードの名前を `Character` にする

### 2-2. 当たり判定
1. Player を選び、**CollisionShape3D** を追加
2. **Shape** → **New SphereShape3D**
3. 球をキャラクターに合わせる: オレンジの点を動かして、半径 約 `0.8` m にする
4. 球の下端が、グリッドの平面に合うよう、上へ動かす
   - 見づらいときは、Pivot または Character の **目のアイコン**で、モデルを一時的に隠す
5. `player.tscn` として保存

```
Player (CharacterBody3D)
├─ Pivot (Node3D)
│  └─ Character (player.glb)
└─ CollisionShape3D
```

### 2-3. 入力アクション（Input Map）
1. メニュー **Project > Project Settings...** → **Input Map** タブ
2. **Show Built-in Actions** は**オフのまま**（ゲームパッドにも対応するため、自分用のアクションを作る）
3. 上の入力欄に、名前を入れて **Add** で、次の5つを作る:
   `move_left`、`move_right`、`move_forward`、`move_back`、`jump`
4. 各アクションの **＋** で、キーを割り当てる

| アクション | キーボード | ゲームパッド |
|---|---|---|
| move_left | ← と A | 左スティックの X 軸のマイナス方向 |
| move_right | → と D | 左スティックの X 軸のプラス方向 |
| move_forward | ↑ と W | 左スティックの Y 軸のマイナス方向 |
| move_back | ↓ と S | 左スティックの Y 軸のプラス方向 |
| jump | Space | A ボタン |

> 操作:「＋」→ キーを押す → **OK**。ゲームパッドは、「＋」のあと、一覧の **Joypad Axes** や **Joypad Buttons** から自分で選ぶ。

---

## パート3: コードでプレイヤーを動かす

### 3-1. スクリプト
1. Player を**右クリック > Attach Script**
2. **Template** を **Empty** にして **Create**

```gdscript
extends CharacterBody3D

# 横の移動速度（メートル／秒）
@export var speed = 14
# 空中での下向きの加速度（重力。メートル／秒²）
@export var fall_acceleration = 75

var target_velocity = Vector3.ZERO
```

> 3Dは単位が**メートル**なので、2Dより数値がずっと小さい（または桁が違う）点に注意。

### 3-2. 入力から方向を求める
`_physics_process` を使います。物理に関わる処理（移動など）に使う、一定間隔で呼ばれる関数です。

```gdscript
func _physics_process(delta):
    var direction = Vector3.ZERO

    if Input.is_action_pressed("move_right"):
        direction.x += 1
    if Input.is_action_pressed("move_left"):
        direction.x -= 1
    if Input.is_action_pressed("move_back"):
        # 3D では、XZ平面が地面。奥（後ろ）が +Z
        direction.z += 1
    if Input.is_action_pressed("move_forward"):
        direction.z -= 1
```

### 3-3. 斜めを速くしない＋向きを変える
`_physics_process` の中、上のコードの続きに:

```gdscript
    if direction != Vector3.ZERO:
        direction = direction.normalized()
        # Pivot の向きを、進む方向に向ける
        $Pivot.basis = Basis.looking_at(direction)
```

### 3-4. 速度と重力
**if の外側**（字下げを1段戻す）に、続けて書きます。

```gdscript
    # 地面での速度
    target_velocity.x = direction.x * speed
    target_velocity.z = direction.z * speed

    # 垂直方向の速度（空中なら落ちる）
    if not is_on_floor():
        target_velocity.y = target_velocity.y - (fall_acceleration * delta)

    # キャラクターを動かす
    velocity = target_velocity
    move_and_slide()
```

- `is_on_floor()`: 床に触れているとき `true`
- `move_and_slide()`: 壁にぶつかっても、滑らせて、なめらかに動かす

### 3-5. Main に置いてカメラをつける
3Dには、**カメラがないと何も映りません。**

1. `player.tscn` を保存し、**Main** のタブに切り替える（閉じていたら FileSystem で `main.tscn` をダブルクリック）
2. Main を**右クリック > Instantiate Child Scene** → `player.tscn` をダブルクリック
3. Main を右クリック > **Add Child Node** → **Marker3D** → 名前を `CameraPivot`
4. CameraPivot の子に **Camera3D** を追加

```
Main
├─ Ground
├─ DirectionalLight3D
├─ Player
└─ CameraPivot (Marker3D)
   └─ Camera3D
```

5. Camera3D を選ぶと、ビューポート左上に **Preview** チェックボックスが出る。これで、ゲーム画面のカメラ視点を見られる
6. 見ながら配置するために、ビューを2分割する。ツールバーの **View > 2 Viewports**（**Ctrl+2**、macOS は Cmd+2）
7. 下のビューで Camera3D を選んで **Preview** をオン
8. 上のビューで、Camera3D を選び、Z軸（青い矢印）方向へ **約 19** 動かす
9. **CameraPivot** を選び、X軸まわり（赤い円）に **-45 度** 回転する（クレーンにカメラが付いているイメージ）
10. **F6** で実行 → 矢印キーで動けることを確認

画面の端に空間が見えるのは、遠近感（パースペクティブ）のためです。この作品では、**平行投影（オルソ）**にして、距離を見やすくします。

11. Camera3D を選び、Inspector の **Projection** を **Orthogonal**、**Size** を `19` にする

> ⚠ 平行投影で、影がぼやけたら、Camera3D の **Far** を `100` 程度に下げてみる（小さくしすぎると、遠くの物が消える）。

✅ **F6** で、8方向に動けて、床を突き抜けない。

---

## パート4: 敵（Mob）のシーン

### 4-1. 構成
1. 新しいシーンを作り、根を **CharacterBody3D**、名前を `Mob` にする
2. 子に **Node3D** を追加して `Pivot` にする
3. `mob.glb` を Pivot へドラッグ＆ドロップし、名前を `Character` にする
4. Mob の子に **CollisionShape3D** を追加し、**Shape** を **New BoxShape3D**
5. ボックスを調整する。**床に接して、モデルより少し細め**にする
   - 当たり判定が大きすぎると、離れて見えるのにやられて、理不尽に感じる
   - 見下ろしカメラなので、高さが少し高くてもよい
6. Mob の子に **VisibleOnScreenNotifier3D** を追加し、ピンクの箱が、モデル全体を覆うように大きさを調整
   - 画面から出たことを知らせるノード。敵が無限に増えないよう、出たら消すために使う

```
Mob (CharacterBody3D)
├─ Pivot (Node3D)
│  └─ Character (mob.glb)
├─ CollisionShape3D
└─ VisibleOnScreenNotifier3D
```

### 4-2. 敵のコード
Mob にスクリプトを付けます。

```gdscript
extends CharacterBody3D

# 最低速度・最高速度（メートル／秒）
@export var min_speed = 10.0
@export var max_speed = 18.0

func _physics_process(_delta):
    move_and_slide()

# Main から呼ばれる関数
func initialize(start_position, player_position):
    # start_position に置き、player_position の方を向かせる
    look_at_from_position(start_position, player_position, Vector3.UP)
    # 正面から外れるように、-45〜+45度でランダムにずらす
    rotate_y(randf_range(-PI / 4, PI / 4))

    # ランダムな速さ
    var random_speed = randf_range(min_speed, max_speed)
    # 前方向の速度
    velocity = Vector3.FORWARD * random_speed
    # 向いている方向に合わせて、回転させる
    velocity = velocity.rotated(Vector3.UP, rotation.y)
```

### 4-3. 画面から出たら消す
1. **VisibleOnScreenNotifier3D** を選び、**Signals** タブを開く
2. **screen_exited()** をダブルクリック → 接続先は **Mob** → **Connect**
3. 作られた関数に、`queue_free()` を書く

```gdscript
func _on_visible_on_screen_notifier_3d_screen_exited():
    queue_free()
```

`mob.tscn` として保存します。

---

## パート5: 敵を出現させる

### 5-1. 画面サイズ
1. Project > Project Settings > **General** タブ > **Display > Window**
2. **Viewport Width** = `720`、**Viewport Height** = `540`

### 5-2. 出現させる経路（目印の円柱を使う）
カメラに映る範囲のすぐ外に経路を作りたいので、目印を置きます。

1. ビューが2分割されていなければ、**Ctrl+2**。下のビューで、Camera3D を選び **Preview** オン
2. Main に **Node3D** を追加して、名前を `Cylinders` にする
3. その子に **MeshInstance3D** を追加し、**Mesh** を **New CylinderMesh** にする
4. 上のビューを「上から見た平行投影」にする（ビューの左上のメニュー、またはテンキーの **7**）
   - グリッドがうるさい場合は、ツールバーの **View > View Grid** で切り替え
5. 円柱を、カメラの映る範囲の**ちょうど外側の左上**へ動かす（スナップをオンにしておくと楽。Y キー）
6. **Ctrl+D** で複製して、下方向（青いZ軸）へ動かし、画面の外の左下へ置く
7. 2つを Shift+クリックで選び、複製して、右方向（赤いX軸）に動かす（右上と右下）
8. 白くて見づらいので、色を付ける。4つの MeshInstance3D をすべて選び（最初をクリック、最後を Shift+クリック）、Inspector の **Material** に **StandardMaterial3D** を作る
9. 球のアイコンを開き、**Albedo** を展開して、背景と違う色（オレンジなど）にする
10. 役目が終わったら、Cylinders を折りたたむ（目のアイコンで、非表示にしてもよい）

### 5-3. 経路
1. Main に **Path3D** を追加すると、ツールバーに4つのアイコンが出る。**Add Point**（緑の＋）を選ぶ
2. 4つの**円柱の中心**を順にクリックして、点を作る
3. **Close Curve** アイコンで経路を閉じる
4. Path3D の子に **PathFollow3D** を追加する
5. 名前を変える: Path3D → `SpawnPath`、PathFollow3D → `SpawnLocation`

### 5-4. Main のスクリプト
Main にスクリプトを付けます。

```gdscript
extends Node

@export var mob_scene: PackedScene
```

**mob_scene に mob.tscn を設定します**（忘れると null になる）: Main を選び、`mob.tscn` を FileSystem から、Inspector の **Mob Scene** へドラッグ＆ドロップ。

### 5-5. タイマー
1. Main に **Timer** を追加し、`MobTimer` と名前を付ける
2. **Wait Time** = `0.5`、**Autostart** = オン
3. MobTimer を選び、**Signals** タブで **timeout** をダブルクリック → 接続先 **Main** → **Connect**

```gdscript
func _on_mob_timer_timeout():
    # Mob の新しいインスタンスを作る
    var mob = mob_scene.instantiate()

    # 経路上のランダムな位置を選ぶ
    var mob_spawn_location = get_node("SpawnPath/SpawnLocation")
    mob_spawn_location.progress_ratio = randf()

    var player_position = $Player.position
    mob.initialize(mob_spawn_location.position, player_position)

    # Main に加えて、出現させる
    add_child(mob)
```

**F6** で実行 ✅ 敵が出てきて、まっすぐ進んで、画面外で消える。敵どうしは、今は、ぶつかる。

> ⚠ Main から `Player` を消すと、`$Player.position` でエラーになります。

---

## パート6: ジャンプと踏みつぶし

### 6-1. 物理レイヤーとマスク
- **Layer**: 自分がどのレイヤーに**いるか**
- **Mask**: どのレイヤーと**衝突を検知するか**
- 初期値は、全員がレイヤー1、マスク1 → 全員が衝突する

レイヤーに名前を付けます。
1. Project > Project Settings > **Layer Names > 3D Physics**
2. 1〜3番に、`player`、`enemies`、`world` と入力

各ノードの設定をします。Inspector の **Collision** セクションで、Layer と Mask のボタンを切り替えます。

| ノード | Layer | Mask |
|---|---|---|
| Ground（main.tscn） | `world`（3番のみ） | なし（全部オフ） |
| Player（player.tscn） | player（初期値のまま） | `enemies` と `world` |
| Mob（mob.tscn） | `enemies` | なし（全部オフ） |

> 敵の Mask を空にしたので、敵どうしはすり抜けます。ぶつけたい場合は、`enemies` の Mask をオンにします。

### 6-2. ジャンプ
Player のスクリプトで、`fall_acceleration` の次に追加:

```gdscript
# ジャンプ時の上向きの勢い（メートル／秒）
@export var jump_impulse = 20
```

`_physics_process` の中、`move_and_slide()` の**前**に追加:

```gdscript
    # ジャンプ
    if is_on_floor() and Input.is_action_just_pressed("jump"):
        target_velocity.y = jump_impulse
```

✅ Space でジャンプする。

### 6-3. 敵をグループにする
1. `mob.tscn` を開き、**Mob** を選ぶ
2. Inspector の隣の **Groups** タブ → **＋** → 名前に `mob` と入力して **OK**

### 6-4. 踏みつぶし
Player のスクリプトの先頭に追加:

```gdscript
# 敵を踏んだときに跳ねる、上向きの勢い
@export var bounce_impulse = 16
```

`_physics_process` の中、ジャンプのコードの後（`move_and_slide()` の前）に追加:

```gdscript
    # このフレームで起きた衝突を、すべて調べる
    for index in range(get_slide_collision_count()):
        var collision = get_slide_collision(index)

        # 同じフレームに同じ敵と2回ぶつかると、2回目は null になるので飛ばす
        if collision.get_collider() == null:
            continue

        # 衝突相手が mob グループなら
        if collision.get_collider().is_in_group("mob"):
            var mob = collision.get_collider()
            # 上から踏んだか、を確かめる
            if Vector3.UP.dot(collision.get_normal()) > 0.1:
                # 踏みつぶして、跳ねる
                mob.squash()
                target_velocity.y = bounce_impulse
                # 二重カウントを防ぐ
                break
```

> `Vector3.UP.dot(collision.get_normal()) > 0.1` は、「衝突面が、ほぼ上を向いているか」の判定。（上から乗っていれば true）

Mob のスクリプトに、シグナルと関数を追加します。

先頭近く:
```gdscript
# プレイヤーに踏まれたときに出す
signal squashed
```

末尾:
```gdscript
func squash():
    squashed.emit()
    queue_free()
```

✅ **F5**（メインシーンに `main.tscn` を選ぶ）で実行 → 敵を踏んで消せる。

---

## パート7: プレイヤーを倒す

踏みつぶしとは別に、「地面で敵に触れたらやられる」判定を、**Area3D** で作ります。

### 7-1. 当たり判定
1. `player.tscn` を開き、Player に **Area3D** を追加して、名前を `MobDetector`
2. その子に **CollisionShape3D** を追加し、**Shape** を **New CylinderShape3D**
3. 円柱の形を調整:
   - **低く**して、キャラクターの**上の方**に置く（ジャンプすると、敵に当たらない高さになる）
   - 球より**少し広く**する（踏む前に、横から触れたら先にやられるように）
4. MobDetector を選び、Inspector で:
   - **Monitorable** を**オフ**（他からは、検知されない）
   - **Collision > Layer** は**すべてオフ**
   - **Collision > Mask** は `enemies` のみ

### 7-2. シグナルとコード
1. MobDetector の **Signals** タブで **body_entered** をダブルクリック → 接続先 **Player** → **Connect**
2. Player のスクリプトの**先頭**に、シグナルを追加:

```gdscript
# 敵に当たったとき
signal hit
```

3. 末尾に追加:

```gdscript
func die():
    hit.emit()
    queue_free()

func _on_mob_detector_body_entered(body):
    die()
```

### 7-3. ゲームを止める
1. `main.tscn` を開き、**Player** を選ぶ
2. **Signals** タブで **hit** をダブルクリック → 接続先 **Main** → **Connect**
3. Main のスクリプトに:

```gdscript
func _on_player_hit():
    $MobTimer.stop()
```

✅ 実行 → 敵に当たるとプレイヤーが消えて、敵の出現が止まる。

> 当たり判定の大きさ・位置は、遊んで気持ちよい形に調整してください。

---

## パート8: スコアとリトライ

### 8-1. スコア表示
1. Main に **Control** を追加し、名前を `UserInterface`（2D画面で作業する）
2. その子に **Label** を追加し、名前を `ScoreLabel`
3. **Text** を `Score: 0`
4. 背景が白いので、文字を黒にする: Inspector の **Theme Overrides > Colors > Font Color** を有効にして黒
5. ビューポートで、文字を動かして、左上の端から離す

### 8-2. テーマ（フォント）
1. `UserInterface` を選び、Inspector の **Theme > Theme** → **New Theme**
2. 作ったテーマをクリックすると、下部パネルにプレビューが出る
3. FileSystem の `fonts/Montserrat-Medium.ttf` を、**Default Font** へドラッグ＆ドロップ
4. **Default Font Size** を `22` にする

### 8-3. スコアのコード
ScoreLabel にスクリプトを付けます。

```gdscript
extends Label

var score = 0

func _on_mob_squashed():
    score += 1
    text = "Score: %s" % score
```

`%s` の部分が、`score` の値に置き換わります。

敵はコードで作るので、エディタではつなげません。**出現のたびに、コードでつなぎます**。Main の `_on_mob_timer_timeout()` の最後に追加:

```gdscript
    # 敵が踏まれたら、スコアを増やす
    mob.squashed.connect($UserInterface/ScoreLabel._on_mob_squashed.bind())
```

✅ 実行 → 敵を踏むと、スコアが増える。

### 8-4. リトライ表示
1. `UserInterface` の子に **ColorRect** を追加し、名前を `Retry`
2. ツールバーの **Anchor Preset** > **Full Rect**
   - ⚠ これだけだと、ほとんど変化がない。**`UserInterface` にも Full Rect を適用**すると、`Retry` が画面全体に広がる（UI部品は、親の大きさが基準のため）
3. Retry の **Color** を、暗くて**半透明**にする（カラーピッカーの **A**（アルファ）を左に下げる）
4. Retry の子に **Label** を追加し、**Text** を `Press Enter to retry.`
5. その Label の **Anchor Preset** を **Center**

### 8-5. リトライのコード
Main のスクリプトに追加・変更します。

```gdscript
func _ready():
    $UserInterface/Retry.hide()
```

`_on_player_hit()` の最後に追加:
```gdscript
    $UserInterface/Retry.show()
```

新しい関数を追加:
```gdscript
func _unhandled_input(event):
    if event.is_action_pressed("ui_accept") and $UserInterface/Retry.visible:
        # 今のシーンを、読み込み直す
        get_tree().reload_current_scene()
```

`ui_accept` は、最初から用意されている、Enter キー等のアクションです。

### 8-6. BGM（オートロード）
普通に Main に音を置くと、リトライでシーンを読み込み直したとき、曲も最初に戻ります。そこで**オートロード**（ゲーム開始時に、シーンの外に自動で読み込まれるもの）を使います。

1. Scene > **New Scene** → **Other Node** → **AudioStreamPlayer** → 名前を `MusicPlayer`
2. FileSystem の `art/House In a Forest Loop.ogg` を、Inspector の **Stream** へドラッグ＆ドロップ
3. **Autoplay** をオン
4. `music_player.tscn` として保存
5. Project > Project Settings > **Globals > Autoload** タブ
6. **Path** の横のフォルダアイコンから `music_player.tscn` を選び、**Add**

✅ 実行すると、BGMが流れる。リトライしても、曲が続く。

> 実行中は Scene ドックに **Remote** と **Local** のタブが出ます。Remote で、動いているゲームのノードの木が見えます（MusicPlayer も、そこに並んでいます）。

---

## パート9: キャラクターのアニメーション

### 9-1. アニメーションエディタ
1. `player.tscn` を開き、**Player** に **AnimationPlayer** を追加
2. 下部に **Animation** パネルが出る
3. **Animation > New** → 名前を `float`
4. **Autoplay**（自動再生）ボタンと、**Loop**（ループ）ボタンを、オンにする
5. パネルの右上の **ピン**アイコンを押すと、パネルが閉じなくなる（ノードの選択を外しても残る）
6. アニメーションの長さを `1.2` 秒にする

### 9-2. 浮遊（float）のキーフレーム
キーフレーム = 「ある時間の、ある値」。時間カーソルの位置に、作られます。

1. **Character** ノードを選び、Inspector の **Transform** を開く
2. **Position** と **Rotation** の横の**鍵アイコン**をクリック
3. 聞かれたら、**RESET トラックを作る**（初期値）を選ぶ
4. 2本のトラックにひし形が出る。ドラッグで、Position のキーを **0.3 秒**、Rotation のキーを **0.1 秒**へ動かす
5. 時間カーソルを **0.5 秒**に動かす（グレーの目盛りをクリック、または数値入力）
6. Inspector で、Position の **Y** を `0.65`、Rotation の **X** を `8` にする
7. もう一度、2つの鍵アイコンをクリックして、キーを打つ
8. Position のキーを、ドラッグで **0.7 秒**に動かす
9. 時間カーソルを**終点（1.2 秒）**に動かす
10. Position の **Y** を `0.35` 前後、Rotation の **X** を `-9` にして、また鍵アイコンでキーを打つ

プレビュー再生は **Shift+D**、停止は **S**。

### 9-3. 動きを自然にする（イージング）
初期状態の動きは機械的です。キーフレームの間の変化のしかたを調整します。

1. 最初の2つのキーを、ドラッグで囲んで、選ぶ
2. Inspector の **Easing** のカーブを、**左に引っ張る**（ease-out: 最初は速く、後半はゆっくり）
3. Rotation トラックの2つ目のキーにも、同じく ease-out をかける
4. Position の2つ目のキーは、逆にカーブを**右へ**引っ張る

再生して違いを確認します。

> アニメーションは、毎フレーム、対象のプロパティを上書きします。そのため、アニメーションさせたのは **Character** で、**Pivot** は触っていません。コードでは、Pivot の向きを、アニメーションの上に重ねて変えられます。

✅ 実行 → 浮かぶように動く。足元が床に近ければ、Pivot を少し上げる。

### 9-4. コードでアニメーションを制御
Player の `_physics_process()` で、`direction` を調べている `if` の中に追加し、`else` も追加します。

```gdscript
    if direction != Vector3.ZERO:
        direction = direction.normalized()
        $Pivot.basis = Basis.looking_at(direction)
        $AnimationPlayer.speed_scale = 4
    else:
        $AnimationPlayer.speed_scale = 1
```

動いているときは、アニメーションを4倍の速さにします。

関数の最後（`move_and_slide()` の後）に、ジャンプで、体を傾ける行を追加:

```gdscript
    $Pivot.rotation.x = PI / 6 * velocity.y / jump_impulse
```

### 9-5. 敵にも同じアニメーション
`Player` と `Mob` は、同じ構成（Pivot と Character）なので、アニメーションを使いまわせます。

1. Player の AnimationPlayer を選び、**Animation > Manage Animations...**
2. `float` の横の **Copy animation to clipboard**（四角が2つのアイコン）をクリック → **OK**
3. `mob.tscn` を開き、**AnimationPlayer** を追加して選ぶ
4. **Animation > Manage Animations** → **New Library**。「Global library will be created.」と出たら、名前は**空のまま** **OK**
5. **Paste**（クリップボードのアイコン）をクリック → 貼り付けられる → **OK**
6. **Autoplay** と **Loop** を、オンにする

Mob のスクリプトの `initialize()` の最後に追加（敵の速さに合わせて、アニメーションを速くする）:

```gdscript
    $AnimationPlayer.speed_scale = random_speed / min_speed
```

### 完成おめでとうございます
3Dゲームができました。次は、公式マニュアルで、スクリプト、3D、物理、入力のセクションを読むのがおすすめです。

---

## 付録: 完成コード（答え合わせ用）

### player.gd
```gdscript
extends CharacterBody3D

signal hit

@export var speed = 14
@export var fall_acceleration = 75
@export var jump_impulse = 20
@export var bounce_impulse = 16

var target_velocity = Vector3.ZERO

func _physics_process(delta):
    var direction = Vector3.ZERO

    if Input.is_action_pressed("move_right"):
        direction.x += 1
    if Input.is_action_pressed("move_left"):
        direction.x -= 1
    if Input.is_action_pressed("move_back"):
        direction.z += 1
    if Input.is_action_pressed("move_forward"):
        direction.z -= 1

    if direction != Vector3.ZERO:
        direction = direction.normalized()
        $Pivot.basis = Basis.looking_at(direction)
        $AnimationPlayer.speed_scale = 4
    else:
        $AnimationPlayer.speed_scale = 1

    target_velocity.x = direction.x * speed
    target_velocity.z = direction.z * speed

    if not is_on_floor():
        target_velocity.y = target_velocity.y - (fall_acceleration * delta)

    if is_on_floor() and Input.is_action_just_pressed("jump"):
        target_velocity.y = jump_impulse

    for index in range(get_slide_collision_count()):
        var collision = get_slide_collision(index)
        if collision.get_collider() == null:
            continue
        if collision.get_collider().is_in_group("mob"):
            var mob = collision.get_collider()
            if Vector3.UP.dot(collision.get_normal()) > 0.1:
                mob.squash()
                target_velocity.y = bounce_impulse
                break

    velocity = target_velocity
    move_and_slide()

    $Pivot.rotation.x = PI / 6 * velocity.y / jump_impulse

func die():
    hit.emit()
    queue_free()

func _on_mob_detector_body_entered(body):
    die()
```

### mob.gd
```gdscript
extends CharacterBody3D

@export var min_speed = 10.0
@export var max_speed = 18.0

signal squashed

func _physics_process(_delta):
    move_and_slide()

func initialize(start_position, player_position):
    look_at_from_position(start_position, player_position, Vector3.UP)
    rotate_y(randf_range(-PI / 4, PI / 4))

    var random_speed = randf_range(min_speed, max_speed)
    velocity = Vector3.FORWARD * random_speed
    velocity = velocity.rotated(Vector3.UP, rotation.y)

    $AnimationPlayer.speed_scale = random_speed / min_speed

func _on_visible_on_screen_notifier_3d_screen_exited():
    queue_free()

func squash():
    squashed.emit()
    queue_free()
```

### main.gd
```gdscript
extends Node

@export var mob_scene: PackedScene

func _ready():
    $UserInterface/Retry.hide()

func _on_mob_timer_timeout():
    var mob = mob_scene.instantiate()

    var mob_spawn_location = get_node("SpawnPath/SpawnLocation")
    mob_spawn_location.progress_ratio = randf()

    var player_position = $Player.position
    mob.initialize(mob_spawn_location.position, player_position)

    add_child(mob)

    mob.squashed.connect($UserInterface/ScoreLabel._on_mob_squashed.bind())

func _on_player_hit():
    $MobTimer.stop()
    $UserInterface/Retry.show()

func _unhandled_input(event):
    if event.is_action_pressed("ui_accept") and $UserInterface/Retry.visible:
        get_tree().reload_current_scene()
```

### score_label.gd
```gdscript
extends Label

var score = 0

func _on_mob_squashed():
    score += 1
    text = "Score: %s" % score
```

---

## 最後に: 各章のおさらい

| 章 | 学んだこと |
|---|---|
| 1 | ノード・シーン・シーンツリー・シグナル、エディタの画面 |
| 2 | シーン保存、インスタンス化、スクリプト、入力、シグナル |
| 3 | 2Dの移動・当たり判定・UI・音 |
| 4 | 3Dの移動・物理レイヤー・グループ・アニメーション・オートロード |
