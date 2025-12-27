# 凹凸を実装する

「デコボコ感」を出すには、CGの世界で最もよく使われる魔法、 **「ノーマルマップ（法線マップ）」** を使います。

これを使うと、ポリゴンの数はそのままでも、光の当たり方を計算で誤魔化して「あたかも表面がデコボコしているかのように」見せることができます。

![alt text](earth_normal_map.png)

## 1. 準備：紫色の画像
デコボコ情報を記録した、**「全体的に薄紫色」**の画像が必要です。 上の画像をコピーして保存してください。
ファイル名: earth_normal_map.png とします。

配置: assets フォルダの中に保存してください。

## 2. コードの変更
main.rs の setup 関数を少し書き換えます。 やることは単純で、画像をもう一枚読み込み、マテリアルの normal_map_texture という項目にセットするだけです。

ついでに、デコボコがより分かりやすくなるように、惑星の「ツルツル具合（ラフネス）」も調整しましょう。

```rust
fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>,
    asset_server: Res<AssetServer>, 
) {
    let texture_handle = asset_server.load("earth.png");
    let normal_handle = asset_server.load("earth_normal_map.png"); // ノーマルマップ

    let material_handle = materials.add(StandardMaterial {
        base_color: Color::WHITE, 
        base_color_texture: Some(texture_handle), 
        normal_map_texture: Some(normal_handle), // ノーマルマップを追加

        perceptual_roughness: 1.0, // 0.0(ツルツル) ~ 1.0
        
        metallic: 0.1, // 0.0(非金属) ~ 1.0

        ..default()
    });

    commands.spawn((
        Earth,
        Mesh3d(meshes.add(Sphere::new(1.0))), 
        MeshMaterial3d(material_handle),
        Transform::from_xyz(0.0, 0.0, 0.0),
    ));

    commands.spawn((
        Camera3d::default(),
        Transform::from_xyz(2.0, 0.0, 2.0).looking_at(Vec3::ZERO, Vec3::Y), // 少し近づけた
    ));

    // ライト
    commands.spawn((
        PointLight {
            intensity: 2_000_000.0,
            shadows_enabled: true,
            ..default()
        },
        Transform::from_xyz(3.0, 3.0, 5.0) // 見えやすいように修正
    ));
}
```

## 実行
![alt text](image-1.png)

正直あんまりわからない