# UI構築について学びましょう

## ベースコード（コピペしてください）
```rust
use bevy::prelude::*;
use bevy::core_pipeline::tonemapping::Tonemapping;
use bevy::post_process::bloom::Bloom;
use bevy::render::view::Hdr;

// 太陽
#[derive(Component)]
struct Sun;

// 地球
#[derive(Component)]
struct Earth;

// カメラ
#[derive(Component)]
struct MainCamera;

fn main() {
    App::new()
        // 背景を真っ黒にする
        .insert_resource(ClearColor(Color::BLACK))
        // ウィンドウなど
        .add_plugins(DefaultPlugins)
        // 開始時に一度だけ実行
        .add_systems(Startup, setup)
        // 定期的に実行
        .add_systems(Update, (rotate_planet, move_camera).chain())
        // スタート
        .run();
}

// セットアップ(Startupなので最初に一度だけ)
fn setup(
    // コマンド全体
    mut commands: Commands,
    // メッシュ
    mut meshes: ResMut<Assets<Mesh>>,
    // マテリアル
    mut materials: ResMut<Assets<StandardMaterial>>, 
    // アセット読み込み
    asset_server: Res<AssetServer>, 
) {

    // 太陽のマテリアル(mterials.addでマテリアル作成)
    let sun_mat = materials.add(StandardMaterial {
        // ベースカラー
        base_color: Color::WHITE, // 真っ白
        // 発光させる
        emissive: LinearRgba::rgb(10.0, 2.0, 0.0), 
        // それ以外デフォルト
        ..default()
    });

    // 太陽を生成(commands.spawnで生成)
    commands.spawn((
        // Sun構造体を探す
        Sun,
        // 3Dメッシュで球体
        Mesh3d(meshes.add(Sphere::new(1.0))),
        // 先ほど作ったマテリアルを適用
        MeshMaterial3d(sun_mat),
        // 生成する座標
        // with_scaleでサイズだけを調整
        Transform::from_xyz(100.0, 0.0, -500000.0).with_scale(Vec3::splat(20000.0)),
    ));

    // 地球のテクスチャマップ読み込み
    let texture_handle = asset_server.load("earth.png");

    // 地球のマテリアル
    let material_handle = materials.add(StandardMaterial {
        // 画像をそのまま見せるためにベースカラーは白
        base_color: Color::WHITE,
        // ここにテクスチャマップを入れる
        base_color_texture: Some(texture_handle), 
        ..default()
    });

    // 地球生成
    commands.spawn((
        Earth,
        Mesh3d(meshes.add(Sphere::new(1.0))), // UV座標(画像の貼り位置)を持った球体
        MeshMaterial3d(material_handle),
        Transform::from_xyz(0.0, 0.0, 0.0).with_scale(Vec3::splat(1.0)),
    ));

    // カメラ
    commands.spawn((
    // MainCameraを探す
    MainCamera,
    // カメラを実装
    Camera3d::default(),
    // 設定を変える
    Projection::Perspective(PerspectiveProjection {
        // 最小描画距離？
        near: 1.0,
        // 最大描画距離
        far: 1_000_000.0,
        ..default()
    }),
    // HDRを有効にする
    Hdr,
    // トーンマッピングを適用する
    Tonemapping::TonyMcMapface, 
    // Bloomを設定
    Bloom {
        intensity: 0.2,
        scale: Vec2::new(5.0, 1.0),
        ..Default::default()
    },
    // 位置と見る方向
    Transform::from_xyz(0.0, 0.0, 100.0).looking_at(Vec3::ZERO, Vec3::Y),
    ));


    // ライト
    commands.spawn((
        PointLight {
            intensity: 2_000_000.0,
            shadows_enabled: true,
            ..default()
        },
        Transform::from_xyz(3.0, 3.0, 5.0)
    ));
}

// 移動
fn move_camera(
    time: Res<Time>,
    keyboard_input: Res<ButtonInput<KeyCode>>,
    mut query: Query<&mut Transform, With<MainCamera>>,
) {
    let mut transform = query.single_mut();
    let mut direction = Vec3::ZERO;
    let speed = 30.0;

    if keyboard_input.pressed(KeyCode::KeyW) { direction.z -= 1.2; }
    if keyboard_input.pressed(KeyCode::KeyA) { direction.x -= 1.0; }
    if keyboard_input.pressed(KeyCode::KeyS) { direction.z += 1.2; }
    if keyboard_input.pressed(KeyCode::KeyD) { direction.x += 1.0; }

    if keyboard_input.pressed(KeyCode::ControlLeft) { direction.y -= 1.0; }
    if keyboard_input.pressed(KeyCode::Space) { direction.y += 1.0; }

    if direction.length_squared() > 0.0 {
        let move_delta = direction.normalize() * speed * time.delta_secs();
        transform.unwrap().translation += move_delta;
    }
}

// 惑星を自転させるシステム
fn rotate_planet(time: Res<Time>, mut query: Query<&mut Transform, With<Earth>>) {
    let dt = time.delta_secs();
    // 時間の更新のたびに0.5度回転
    for mut transform in &mut query {
        transform.rotate_y(0.5 * dt);
    }
}
```