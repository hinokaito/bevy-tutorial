# カメラ
太陽系全体を一気に描画させようとする方向性はやめましょう。
代わりに、各惑星にワンクリックで飛べるようにします。

## 実装
```rust
use bevy::prelude::*;

// カメラ
#[derive(Component)]
struct MainCamera;

#[derive(Component)]
struct Sun;

#[derive(Component)]
struct Mercury;

#[derive(Component)]
struct Venus;

#[derive(Component)]
struct Earth;

#[derive(Component)]
struct Mars;

#[derive(Component)]
struct Jupiter;

#[derive(Component)]
struct Saturn;

#[derive(Component)]
struct Uranus;

#[derive(Component)] 
struct Neptune;

// match用
#[derive(Component, Clone, Copy, Debug)]
enum Planet {
    SUN, MERCURY, VENUS, EARTH, MARS, JUPITER, SATURN, URANUS, NEPTUNE 
}

#[derive(Component)]
struct PlanetButton(Planet);

// 天文学単位
const ASTRONOMICAL_UNIT: f32 = 23455.0; 

// [相対サイズ, x, y, z]
const SUN:     [f32; 4] =   [109.0, 0.0 * ASTRONOMICAL_UNIT,   0.0, 0.0];
const MERCURY: [f32; 4] =   [0.38,  0.32 * ASTRONOMICAL_UNIT,  0.0, 0.0];
const VENUS:   [f32; 4] =   [0.95,  0.72 * ASTRONOMICAL_UNIT,  0.0, 0.0];
const EARTH:   [f32; 4] =   [1.00,  1.00 * ASTRONOMICAL_UNIT,  0.0, 0.0];
const MARS:    [f32; 4] =   [0.53,  1.52 * ASTRONOMICAL_UNIT,  0.0, 0.0];
const JUPITER: [f32; 4] =   [11.21, 5.20 * ASTRONOMICAL_UNIT,  0.0, 0.0];
const SATURN:  [f32; 4] =   [9.45,  9.54 * ASTRONOMICAL_UNIT,  0.0, 0.0];
const URANUS:  [f32; 4] =   [4.01,  19.19 * ASTRONOMICAL_UNIT, 0.0, 0.0];
const NEPTUNE: [f32; 4] =   [3.88,  30.07 * ASTRONOMICAL_UNIT, 0.0, 0.0];

fn main() {
    App::new()
        .insert_resource(ClearColor(Color::BLACK))
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, (setup, setup_ui))
        .add_systems(Update, planet_button_system)
        .run();
}

fn setup(
    mut commands: Commands,
    mut meshes: ResMut<Assets<Mesh>>, 
    mut materials: ResMut<Assets<StandardMaterial>>,
) {
    // 太陽
    commands.spawn((
        Sun,
        Mesh3d(meshes.add(Sphere::new(1.0))),
        MeshMaterial3d(materials.add(StandardMaterial {
            base_color: Color::WHITE,
            emissive: LinearRgba::rgb(256.0, 256.0, 256.0),
            ..default()
        })),
        Transform:: from_xyz(SUN[1], SUN[2], SUN[3]).with_scale(Vec3::splat(SUN[0]))
    ));

    // 水星
    commands.spawn((
        Mercury,
        Mesh3d(meshes.add(Sphere::new(1.0))),
        MeshMaterial3d(materials.add(StandardMaterial {
            base_color: Color::WHITE,
            emissive: LinearRgba::rgb(256.0, 256.0, 256.0),
            ..default()
        })),
        Transform:: from_xyz(MERCURY[1], MERCURY[2], MERCURY[3]).with_scale(Vec3::splat(MERCURY[0]))
    ));

    // 金星
    commands.spawn((
        Venus,
        Mesh3d(meshes.add(Sphere::new(1.0))),
        MeshMaterial3d(materials.add(StandardMaterial {
            base_color: Color::WHITE,
            emissive: LinearRgba::rgb(256.0, 256.0, 256.0),
            ..default()
        })),
        Transform:: from_xyz(VENUS[1], VENUS[2], VENUS[3]).with_scale(Vec3::splat(VENUS[0]))
    ));

    // 地球
    commands.spawn((
        Earth,
        Mesh3d(meshes.add(Sphere::new(1.0))),
        MeshMaterial3d(materials.add(StandardMaterial {
            base_color: Color::WHITE,
            emissive: LinearRgba::rgb(256.0, 256.0, 256.0),
            ..default()
        })),
        Transform:: from_xyz(EARTH[1], EARTH[2], EARTH[3]).with_scale(Vec3::splat(EARTH[0]))
    ));

    // 火星
    commands.spawn((
        Mars,
        Mesh3d(meshes.add(Sphere::new(1.0))),
        MeshMaterial3d(materials.add(StandardMaterial {
            base_color: Color::WHITE,
            emissive: LinearRgba::rgb(256.0, 256.0, 256.0),
            ..default()
        })),
        Transform:: from_xyz(MARS[1], MARS[2], MARS[3]).with_scale(Vec3::splat(MARS[0]))
    ));

    // 木星
    commands.spawn((
        Jupiter,
        Mesh3d(meshes.add(Sphere::new(1.0))),
        MeshMaterial3d(materials.add(StandardMaterial {
            base_color: Color::WHITE,
            emissive: LinearRgba::rgb(256.0, 256.0, 256.0),
            ..default()
        })),
        Transform:: from_xyz(JUPITER[1], JUPITER[2], JUPITER[3]).with_scale(Vec3::splat(JUPITER[0]))
    ));

    // 土星
    commands.spawn((
        Saturn,
        Mesh3d(meshes.add(Sphere::new(1.0))),
        MeshMaterial3d(materials.add(StandardMaterial {
            base_color: Color::WHITE,
            emissive: LinearRgba::rgb(256.0, 256.0, 256.0),
            ..default()
        })),
        Transform:: from_xyz(SATURN[1], SATURN[2], SATURN[3]).with_scale(Vec3::splat(SATURN[0]))
    ));

    // 天王星
    commands.spawn((
        Uranus,
        Mesh3d(meshes.add(Sphere::new(1.0))),
        MeshMaterial3d(materials.add(StandardMaterial {
            base_color: Color::WHITE,
            emissive: LinearRgba::rgb(256.0, 256.0, 256.0),
            ..default()
        })),
        Transform:: from_xyz(URANUS[1], URANUS[2], URANUS[3]).with_scale(Vec3::splat(URANUS[0]))
    ));

    // 海王星
    commands.spawn((
        Neptune,
        Mesh3d(meshes.add(Sphere::new(1.0))),
        MeshMaterial3d(materials.add(StandardMaterial {
            base_color: Color::WHITE,
            emissive: LinearRgba::rgb(256.0, 256.0, 256.0),
            ..default()
        })),
        Transform:: from_xyz(NEPTUNE[1], NEPTUNE[2], NEPTUNE[3]).with_scale(Vec3::splat(NEPTUNE[0]))
    ));

    // カメラ
    commands.spawn((
        MainCamera,
        Camera3d::default(),
        Projection::Perspective(PerspectiveProjection {
            far: 100_000_000.0,
            ..default()
        }),
        Transform::from_xyz(1000.0, 0.0, 0.0).looking_at(Vec3::ZERO, Vec3::Y),
    ));
}

// UIセットアップ用システム
fn setup_ui(mut commands: Commands) {
    commands.spawn((
        Node {
            width: Val::Percent(100.0),
            height: Val::Percent(100.0),
            flex_direction: FlexDirection::Column,
            justify_content: JustifyContent::End, 
            ..default()
        },
    ))
    .with_children(|parent| {
        let planets = [
            (Planet::SUN,     "Sun"),
            (Planet::MERCURY, "Mercury"),
            (Planet::VENUS,   "Venus"),
            (Planet::EARTH,   "Earth"),
            (Planet::MARS,    "Mars"),
            (Planet::JUPITER, "Jupiter"),
            (Planet::SATURN,  "Saturn"),
            (Planet::URANUS,  "Uranus"),
            (Planet::NEPTUNE, "Neptune")
        ];

        parent.spawn((
            Node {
                width: Val::Percent(100.0),
                height: Val::Px(50.0), 
                flex_direction: FlexDirection::Row, 
                align_items: AlignItems::Center,
                justify_content: JustifyContent::FlexStart,
                padding: UiRect::horizontal(Val::Px(20.0)),
                column_gap: Val::Px(30.0), 
                overflow: Overflow::clip_x(),
                ..default()
            },
            BackgroundColor(Color::srgba(0.1, 0.1, 0.1, 0.5)),
        ))
        .with_children(|footer| {
            for (planet_type, label) in planets {
                footer.spawn((
                    Button,
                    Node {
                        padding: UiRect::all(Val::Px(10.0)),
                        border: UiRect::all(Val::Px(2.0)),
                        ..default()
                    },
                    BackgroundColor(Color::srgb(0.2, 0.2, 0.2)),
                    PlanetButton(planet_type), 
                ))
                .with_children(|button| {
                    button.spawn((
                        Text::new(label),
                        TextFont { font_size: 16.0, ..default() },
                        TextColor(Color::WHITE),
                    ));
                });
            }
        });
    });
}

// ボタンの見た目とカメラ位置などを処理
fn planet_button_system(
    mut interaction_query: Query<
        (&Interaction, &mut BackgroundColor, &PlanetButton),
        (Changed<Interaction>, With<Button>),
    >,
    mut camera_query: Query<&mut Transform, With<MainCamera>>,
) {
    let Ok(mut camera_transform) = camera_query.single_mut() else { return };

    for (interaction, mut bg_color, planet_button) in &mut interaction_query {
        match *interaction {
            Interaction::Pressed => {
                *bg_color = Color::srgb(0.1, 0.5, 0.1).into();
                let target_pos = get_planet_position(planet_button.0);

                // 太陽の時は引き目にする
                let offset = if target_pos == Vec3::ZERO {
                    Vec3::new(1000.0, 0.0, 0.0)
                } else {
                    Vec3::new(100.0, 0.0, 0.0)
                };

                // 対象の惑星に視線を合わせる
                *camera_transform = Transform::from_translation(target_pos + offset)
                    .looking_at(target_pos, Vec3::Y);
            }
            Interaction::Hovered => {
                *bg_color = Color::srgb(0.3, 0.3, 0.3).into();
            }
            Interaction::None => {
                *bg_color = Color::srgb(0.2, 0.2, 0.2).into();
            }
        }
    }
}

// 引数に渡された惑星の座標を返す
fn get_planet_position(planet: Planet) -> Vec3 {
    let data = match planet {
        Planet::SUN     => SUN,
        Planet::MERCURY => MERCURY,
        Planet::VENUS   => VENUS,
        Planet::EARTH   => EARTH,
        Planet::MARS    => MARS,
        Planet::JUPITER => JUPITER,
        Planet::SATURN  => SATURN,
        Planet::URANUS  => URANUS,
        Planet::NEPTUNE => NEPTUNE,
    };

    Vec3::new(data[1], data[2], data[3])
}
```

## 確認
画面下部のボタンを押すと移動するのが分かると思います