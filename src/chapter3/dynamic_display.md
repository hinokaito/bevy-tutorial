# 動的なデータ表示をしてみよう
今回は、「カメラの現在座標 (X, Y, Z)」 をリアルタイムに画面左下のテキストに表示させてみましょう。

### 今回の目標
- UI上の静的なテキスト「Status: In Orbit」を、動的な「Camera - Pos: (x, y, z)」に書き換える。

WASDキーでカメラを動かすと、数値がパラパラと変わるようにする。

## Step1: 準備：更新対象を見つけるためのマーカー
どのテキストを書き換えるのかを特定するために、新しいコンポーネント（タグ）を作ります。 main 関数の手前あたりに追加してください。
```rust
// カメラ座標を表示するテキストにつけるタグ
#[derive(Component)]
struct CameraPositionText;
```

## Step2: UIの修正：マーカーを取り付ける
setup_ui 関数の中にある、左下のテキスト（"Status: In Orbit" の部分）に、先ほどのタグを追加します。

setup_ui の footer の部分を以下のように修正してください。
```rust
// ... (footerのchildren内)

            // 左側の情報: ステータス 改め カメラ座標
            footer.spawn((
                Text::new("Pos: 0.0, 0.0, 0.0"), // 初期値
                TextFont { font_size: 18.0, ..default() },
                TextColor(Color::WHITE),
                // ★ここに追加！これを目印にして後で検索します
                CameraPositionText, 
            ));

            // ... (Speedの方はそのままでOK)
```

## Step3: データを同期させる
「カメラの座標」を取得し、「UIのテキスト」に流し込むシステムを作ります。 コードの一番下に追加してください。

```rust
// UIの数値を更新するシステム
fn update_dashboard(
    // カメラの位置を知りたいので MainCamera の Transform を取得
    // カメラは1つしかないので Single が使える (0.15以降の便利機能)
    camera_query: Single<&Transform, With<MainCamera>>,
    
    // 書き換え対象のテキストを取得 (CameraPositionTextを持つもの)
    mut text_query: Single<&mut Text, With<CameraPositionText>>,
) {
    // カメラの座標を取り出す
    let pos = camera_query.translation;

    // テキストを書き換える
    // {:.1} は「小数点以下1桁まで」という意味
    text_query.0 = format!("Pos: {:.1}, {:.1}, {:.1}", pos.x, pos.y, pos.z);
}
```

## Step4: Main関数への登録
update_dashboard システムを Update スケジュールに追加します。
```rust
fn main() {
    App::new()
        .insert_resource(ClearColor(Color::BLACK))
        .insert_resource(SimulationState { is_running: true })
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, (setup, setup_ui))
        // ★ update_dashboard を追加
        .add_systems(Update, (rotate_planet, move_camera, button_system, update_dashboard).chain())
        .run();
}
```

## 実行
cargo run してみてください。

1. 画面左下に Pos: 0.0, 0.0, 100.0 のような数値が出ていますか？
2. WASDキー や Space/Ctrlキー を押してカメラを動かしてみてください。
3. 移動に合わせて、数値がリアルタイムに変化しますか？