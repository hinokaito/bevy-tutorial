# 物理演算を実装してみよう
今回は、「太陽(重力の発生源)」と「惑星(影響を受ける側)」を明確に分けて実装します。

やることは3ステップです。
1. データ（Component）: 「重さ」を表すデータを作る。
2. ロジック（System）: 重力によって「速度」を変化させる関数を作る。
3. 配置（Spawn）: 重い太陽と、回る惑星を配置する。


## Step1: 重さを定義する
まずは、重力の源となる物体につけるデータを作ります。 main.rs のコンポーネント定義エリア（main関数の上あたり）に追加してください。
```rust
// 重力の強さ（質量のようなもの）
#[derive(Component)]
struct Mass {
    value: f32,
}
```
ついでに、太陽であることを区別するためのタグも作っておきましょう。
```rust
#[derive(Component)]
struct Sun;
```

## Step2: 引力の計算ロジック（System）を作る
ここが今回の山場です。 ECSにおいて**「ある物体（太陽）の情報を使って、別の物体（惑星）を動かす」**にはどうすればいいでしょうか？

答えは、Queryを2つ使うことです。
1. 重力源（太陽）の位置と重さを取得するQuery
2. 動かしたい惑星の位置と速度を取得するQuery

これらを組み合わせて、「惑星の速度（Velocity）」を書き換えます。 move_objects 関数の下あたりに、新しい関数 apply_gravity を追加しましょう。

```rust
fn apply_gravity(
    time: Res<Time>,
    // 1. 重力源（太陽）の情報を取得：位置と重さが欲しい（書き換えないので & ）
    //    With<Sun> で「太陽タグがついているもの」だけに絞り込みます
    sun_query: Query<(&Transform, &Mass), With<Sun>>,

    // 2. 惑星の情報を取得：位置と、書き換えたい速度を取得
    //    With<Planet> で「惑星タグがついているもの」だけに絞り込みます
    mut planet_query: Query<(&Transform, &mut Velocity), With<Planet>>,
) {
    // 太陽は1つと仮定して、最初の1個を取得します（なければ何もしない）
    // .single() は「ちょうど1個だけある」ときに成功する便利なメソッドです
    let Ok((sun_transform, sun_mass)) = sun_query.single() else {
        return;
    };

    let dt = time.delta_secs();

    // 全ての惑星に対して処理を行う
    for (planet_transform, mut planet_velocity) in &mut planet_query {
        // --- ここからベクトル計算 ---
        
        // 1. 太陽への方向ベクトルと距離を求める
        //    (相手の位置 - 自分の位置)
        let diff = sun_transform.translation - planet_transform.translation;
        let distance_sq = diff.length_squared(); // 距離の2乗（計算負荷を減らすため）

        // ゼロ除算や、近づきすぎた時の荒ぶりを防ぐガード
        if distance_sq < 0.001 {
            continue;
        }

        // 2. 万有引力の法則っぽい計算
        //    力 F = G * M / r^2 
        //    ここでは定数Gなどは適当に省いて、Mass / 距離の2乗 とします
        let force_magnitude = sun_mass.value / distance_sq;

        // 3. 力をベクトル（向き）にする
        //    diff.normalize() で「長さ1の方向ベクトル」を作り、それに大きさを掛ける
        let force_vector = diff.normalize() * force_magnitude;

        // 4. 速度に加算する（加速度の適用）
        planet_velocity.x += force_vector.x * dt;
        planet_velocity.y += force_vector.y * dt;
        planet_velocity.z += force_vector.z * dt;
    }
}
```

## Step3: 配置と登録
最後に、太陽を配置し、システムを登録します。

### システムの登録 (main関数)
apply_gravity を追加します。 順番が大事です。「力を加えてから(Gravity) → 移動する(Move)」のが自然なので、chain()を使って順序を固定しましょう。

```rust
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup)
        // updateシステムを連結(chain)して順番を保証
        .add_systems(Update, (apply_gravity, move_objects).chain())
        .run();
}
```

### セットアップ
前回のコードを書き換えて、太陽と、公転しそうな惑星を作ります。
```rust
fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>,
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
    // 共通のメッシュとマテリアルを先に作っておく（メモリ節約のテクニック）
    let sphere_mesh = meshes.add(Sphere::new(1.0));
    let sun_mat = materials.add(StandardMaterial {
        base_color: Color::WHITE,
        emissive: LinearRgba::rgb(10.0, 10.0, 0.0), // 発光させる
        ..default()
    });
    let planet_mat = materials.add(Color::srgb(0.3, 0.5, 0.9));

    // 1. 太陽（Sun）を生成
    commands.spawn((
        Sun,
        Mass { value: 500.0 }, // 重さを適当に設定（大きくすると引力が強くなる）
        Mesh3d(sphere_mesh.clone()),
        MeshMaterial3d(sun_mat),
        Transform::from_xyz(0.0, 0.0, 0.0),
    ));

    // 2. 惑星（Planet）を生成
    commands.spawn((
        Planet,
        // 初速を与える：
        // Z軸方向（手前）にずらして、X軸プラス方向（右）へ初速を与えると、
        // 太陽の周りをぐるっと回り始めます。
        Velocity { x: 6.0, y: 0.0, z: 0.0 }, 
        
        Mesh3d(sphere_mesh.clone()),
        MeshMaterial3d(planet_mat),
        Transform::from_xyz(0.0, 0.0, 15.0).with_scale(Vec3::splat(0.5)), // 少し小さくして、手前に配置
    ));

    // カメラ
    commands.spawn((
        Camera3d::default(),
        // 上から見下ろす位置に配置（軌道が見やすいように）
        Transform::from_xyz(0.0, 40.0, 0.0).looking_at(Vec3::ZERO, Vec3::Z), // Z軸を上に設定して見下ろす
    ));
}
```

## 実行してみよう
![alt text](image-2.png)

このように青い惑星が太陽の周りをまわっていれば成功です。
値を好きにいじって遊んでみてください。


_もしうまくいかなかった場合以下のコード全体を参考にしてください。_
```rust
use bevy::prelude::*;

// --- コンポーネント ---

// 位置と速度を定義(Transform自作)
#[derive(Component)]
struct Velocity {
    x: f32,
    y: f32,
    z: f32,
}

// 重力の強さ(質量)
#[derive(Component)]
struct Mass {
    value: f32,
}

// 惑星であることを示すタグ
#[derive(Component)]
struct Planet;

// 太陽であることを示すタグ
#[derive(Component)]
struct Sun;

// --- メイン ---
fn main() {
    App::new()
        .add_plugins(DefaultPlugins)// ウィンドウ、入力、レンダリングなどの基本機能を全部盛り
        .add_systems(Startup, setup) // 開始時に一度だけ走るシステム
        .add_systems(Update, (apply_gravity, move_objects).chain())
        .run();
}

// --- システム ---

// 初期化システム: コマンドを使ってEntity生成
fn setup(
    mut commands:   Commands,
    mut meshes:     ResMut<Assets<Mesh>>,               // メッシュの倉庫
    mut materials:  ResMut<Assets<StandardMaterial>>,   // マテリアルの倉庫
) {
    // 共通のメッシュとマテリアルを先に作っておく
    let sphere_mesh = meshes.add(Sphere::new(1.0));
    let sun_mat = materials.add(StandardMaterial {
        base_color: Color::WHITE,
        emissive: LinearRgba::rgb(10.0, 10.0, 0.0), // 発光させる
        ..default()
    });
    let planet_mat = materials.add(Color::srgb(0.3, 0.5, 0.9));

    // 太陽
    commands.spawn((
        Sun,
        Mass { value: 500.0 }, // 質量
        Mesh3d(sphere_mesh.clone()),
        MeshMaterial3d(sun_mat),
        Transform::from_xyz(0.0, 0.0, 0.0),
    ));

    // 惑星
    commands.spawn((
        Planet,
        // 初速を与える；
        // Z軸方向(手前)にずらして、X軸プラス方向(右)へ初速を与えると、
        // 太陽の周りをぐるっと回り始める。
        Velocity { x: 6.0, y: 0.0, z: 0.0 },

        Mesh3d(sphere_mesh.clone()),
        MeshMaterial3d(planet_mat),
        Transform::from_xyz(0.0, 0.0, 15.0).with_scale(Vec3::splat(0.5)), // 少し小さくして、手前に配置
    ));

    // カメラを配置
    commands.spawn((
        Camera3d::default(),
        Transform::from_xyz(0.0, 40.0, 0.0).looking_at(Vec3::ZERO, Vec3::Z), // Z軸を上に設定して見下ろす
    ));

    // 光源
    commands.spawn((
        PointLight::default(),
        Transform::from_xyz(4.0, 8.0, 4.0),
    ));
}

// Query<...>型を使って、ほしいコンポーネントを指定
// mut Transform : 位置は書き換えるので mut
// &Velocity     : 速度を読むだけなので &
fn move_objects(time: Res<Time>, mut query: Query<(&mut Transform, &Velocity)>) {
    // query.iter_mut() は条件に合う全Entityを返す。
    for (mut transform, velocity) in &mut query {
        // デルタタイム (前のフレームからの経過時間)を使って移動
        let dt = time.delta_secs();

        transform.translation.x += velocity.x * dt;
        transform.translation.y += velocity.y * dt;
        transform.translation.z += velocity.z * dt;
    }
}

fn apply_gravity(
    time: Res<Time>,
    // 重力源(太陽)の情報を取得：位置と重さが欲しい(書き換えないので & )
    // With<Sun> で「太陽タグが付いているもの」だけに絞り込む
    sun_query: Query<(&Transform, &Mass), With<Sun>>,

    // 惑星の情報を取得：位置と書き換えたい速度を取得
    // With<Planet> で「惑星タグがついているもの」だけに絞り込む
    mut planet_query: Query<(&Transform, &mut Velocity), With<Planet>>,
) {
    // 太陽は1つと仮定して、最初の1個を取得する(なければ何もしない)
    // .single()は「ちょうど1個だけある」時に成功するメソッド
    let Ok((sun_transform, sun_mass)) = sun_query.single() else {
        return;
    };

    let dt = time.delta_secs();

    // 全ての惑星に対して処理を行う
    for (planet_transform, mut planet_velocity) in &mut planet_query {
        // --- ここからベクトル計算 ---

        // 太陽への方向ベクトルと距離を求める (相手の位置 - 自分の位置)
        let diff = sun_transform.translation - planet_transform.translation;
        let distance_sq = diff.length_squared(); // 距離の2乗

        // ゼロ助産や、近づきすぎた時のあらぶりを防ぐガード
        if distance_sq < 0.001 {
            continue;
        }

        // 万有引力の法則っぽい計算
        //    力 F = G * M / r^2 
        //    ここでは定数Gなどは適当に省いて、Mass / 距離の2乗 とする
        let force_magnitude = sun_mass.value / distance_sq;

        // 力をベクトルにする
        // diff.normalize() で「長さ1の方向ベクトル」を作り、それに大きさを掛ける
        let force_vector = diff.normalize() * force_magnitude;

        // 速度に加算する（加速度の適用）
        planet_velocity.x += force_vector.x * dt;
        planet_velocity.y += force_vector.y * dt;
        planet_velocity.z += force_vector.z * dt;
    }
}
```