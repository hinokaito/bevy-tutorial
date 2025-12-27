# テクスチャを張ってみよう
![alt text](../images/earth.png)

この画像をsphereに張って観察してみましょう。

## Step1: 素材フォルダを作る
Bevyには厳格なルールがあります。 「画像や3Dモデルなどの素材は、プロジェクトのルートにある assets というフォルダに入れる」というルールです。
1. Cargo.toml がある場所に、assets という名前の新しいフォルダを作ってください。
2. その中に、惑星の模様にしたい画像(上の画像をコピーしearth.pngで保存してください)を入れてください。

## Step2: コンポーネント定義と関数用意
```rust
use bevy::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup)
        .add_systems(Update, rotate_planet)
        .run();
}

#[derive(Component)]
struct Earth;

fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>,
    asset_server: Res<AssetServer>, // 素材読み込み役
) {

}

// 惑星を自転させるシステム
fn rotate_planet(time: Res<Time>, mut query: Query<&mut Transform, With<Earth>>) {

}
```

## Step3: 画像をロードしてマテリアルを作る
```rust
// setup関数内

    // Bevyは自動的に"assets/"フォルダを探しに行く
    // なので"assets/..."とは書かないで直接書く
    let texture_handle = asset_server.load("earth.png");

    // マテリアルを作る
    // 色(bsse_color)ではなく、画像(bsse_color_texture)を指定する
    let material_handle = materials.add(StandardMaterial {
        base_color: Color::WHITE, // 画像の色をそのまま出すために白にする
        base_color_texture: Some(texture_handle), // 画像セット
        ..default()
    });
```

## Step4: 地球・カメラ・ライトを生成
```rust
// setup関数内

    // 地球を生成
    commands.spawn((
        Earth,
        Mesh3d(meshes.add(Sphere::new(1.0))), // UV座標(画像の貼り位置)を持った球体
        MeshMaterial3d(material_handle),
        Transform::from_xyz(0.0, 0.0, 0.0),
    ));

    // カメラ
    commands.spawn((
        Camera3d::default(),
        Transform::from_xyz(0.0, 1.5, 4.0).looking_at(Vec3::ZERO, Vec3::Y),
    ));

    // ライト
    commands.spawn((
        PointLight {
            intensity: 2_000_000.0,
            shadows_enabled: true,
            ..default()
        },
        Transform::from_xyz(4.0, 8.0, 4.0)
    ));
```

## Step5: 自転させる関数を実装
```rust
// 惑星を自転させるシステム
fn rotate_planet(time: Res<Time>, mut query: Query<&mut Transform, With<Earth>>) {
    let dt = time.delta_secs();
    for mut transform in &mut query {
        transform.rotate_y(0.5 * dt);
    }
}
```

## 実行してみる
![alt text](image.png)

このように、地球が表示がされれば成功です。太平洋にギザギザがありますが、主に画像の問題なようなので放置します。