# カメラ操作を実装してみよう
「カメラ操作」 を追加しましょう。 やることは以下の3ステップです。

1. タグ付け: どれが「操作したいカメラ」なのか分かるようにする。
2. 入力の検知: キーボードの状態（押されているか？）を確認する。
3. 移動: カメラの座標 (Transform) を書き換える。
4. 登録

## Step1: カメラにタグをつける
Planet や Sun と同じように、カメラにも名札（Component）を付けます。これがないと、システム側で「どの Entity がメインカメラなのか」を探し出せません。

main 関数の上あたりにあるコンポーネント定義に追加してください。
```rust
#[derive(Component)]
struct MainCamera;
```
そして、setup 関数内のカメラ生成部分に、このタグを追加します。
```rust
    // カメラを配置
    commands.spawn((
        MainCamera, // <--- これを追加！
        Camera3d::default(),
        // ... (Transformなどはそのまま)
    ));
```

## Step2 & 3：操作システムを作る
新しいシステム move_camera を作ります。
```rust
// Res<Time>: 時間を知るため
// Res<ButtonInput<KeyCode>>: キーボードの入力状態を知るため
// Query: カメラの Transform を書き換えるため
fn move_camera(
    time: Res<Time>,
    keyboard_input: Res<ButtonInput<KeyCode>>, // キーボード入力のリソース
    // MainCameraタグが付いているEntityのTransformだけを持ってくる
    mut query: Query<&mut Transform, With<MainCamera>>,
) {
    // カメラは1つしかないので、single_mut()で取得
    // もしカメラが0や複数あるとエラーで止まる
    let mut transform = query.single_mut();

    let mut direction = Vec3::ZERO;
    let speed = 20.0; // カメラの移動速度

    // キー入力をチェック(.pressed は「押されている間ずっと」)
    // 現在のカメラは「真下」を見ているので、画面上の動きに合わせて座標を操作する
    // ※LookingAtの設定により通常と変わる。

    // Wキー: 画面上へ
    if keyboard_input.pressed(KeyCode::KeyW) { direction.z += 1.0; }
    // Aキー: 画面左へ
    if keyboard_input.pressed(KeyCode::KeyA) { direction.x += 1.0; }
    // Sキー: 画面下へ
    if keyboard_input.pressed(KeyCode::KeyS) { direction.z -= 1.0; }
    // Dキー: 画面右へ
    if keyboard_input.pressed(KeyCode::KeyD) { direction.x -= 1.0; }

    // Q/Eキーでズーム
    if keyboard_input.pressed(KeyCode::KeyE) { direction.y -= 1.0; }
    if keyboard_input.pressed(KeyCode::KeyQ) { direction.y += 1.0; }

    // 移動量が0より大きければ、正規化して速度と時間を掛ける
    if direction.length_squared() > 0.0 {
        let move_delta = direction.normalize() * speed * time.delta_secs();
        transform.unwrap().translation += move_delta;
    }
}
```
補足： direction.normalize() をするのは、斜め移動の速度を一定にするためです。これがないと、斜め移動だけ $\sqrt{2}$ 倍速くなってしまいます。

## Step4: 登録
Appのプラグインに追加します。
```rust
.add_systems(Update, (apply_gravity, move_objects, move_camera).chain())
```

## 起動してみよう
WASD/QEで操作ができれば成功です。

