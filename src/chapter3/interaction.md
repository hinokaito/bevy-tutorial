# インタラクションを実装してみよう

今回は、UIを見るだけでなく「触れる」ようにします。 具体的には、ステータスバーに「停止/再開」ボタンを追加し、地球の自転をコントロールできるようにします。

Bevyでボタンを作るには、以下の2つが必要です。

見た目: Button バンドル（自動的にクリック判定機能 Interaction がつきます）。

ロジック: Interaction コンポーネントの状態（クリックされたか？マウスが乗ったか？）を監視するシステム。

## Step1: 準備
まず、シミュレーションが動いているかどうかを管理するデータ（Resource）と、ボタンの文字を書き換えるための目印（Component）を定義します。

main 関数の手前あたりに以下を追加してください。
```rust
// シミュレーションの状態管理
#[derive(Resource)]
struct SimulationState {
    is_running: bool,
}

// ボタンのテキストを後で書き換えるための目印
#[derive(Component)]
struct PauseButtonText;
```

## Step2: 
既存の rotate_planet 関数を、「SimulationState が true のときだけ回る」ように書き換えます。
```rust
// 引数に Res<SimulationState> を追加
fn rotate_planet(
    time: Res<Time>, 
    mut query: Query<&mut Transform, With<Earth>>,
    sim_state: Res<SimulationState> // 追加
) {
    // 停止中なら何もしない
    if !sim_state.is_running {
        return;
    }

    let dt = time.delta_secs();
    for mut transform in &mut query {
        transform.rotate_y(0.5 * dt);
    }
}
```

## Step3: UIの追加：ボタンを配置
setup_ui の中の、下部エリア（footer）にボタンを追加します。 前回の footer.spawn(...).with_children(|footer| { ... }); の中（StatusやSpeedの下あたり）に、以下のコードを追加してください。

```rust
// ... (Status, Speed の定義のあとに追加)

            // ボタン本体
            footer.spawn((
                // Buttonバンドルを入れるだけで、クリック判定(Interaction)などがつく
                Button,
                Node {
                    width: Val::Px(100.0),
                    height: Val::Px(35.0),
                    // ボタン内の文字を中央揃え
                    justify_content: JustifyContent::Center,
                    align_items: AlignItems::Center,
                    border: UiRect::all(Val::Px(2.0)),
                    ..default()
                },
                // ボタンの背景色
                BackgroundColor(Color::srgb(0.2, 0.2, 0.2)),
                // 枠線の色
                BorderColor::all(Color::WHITE),
                // 角丸にする
                BorderRadius::all(Val::Px(8.0)), 
            ))
            .with_children(|button| {
                // ボタンの中のテキスト
                button.spawn((
                    Text::new("Pause"), // 初期値
                    TextFont { font_size: 16.0, ..default() },
                    TextColor(Color::WHITE),
                    PauseButtonText, // 後で文字を変えるための目印
                ));
            });
```

## Step4: ボタンのクリック処理を実装
これが今回のキモです。「ボタンがクリックされたら状態を反転し、色と文字を変える」システムを作ります。 コードの一番下に追加してください。
```rust
// ボタンのインタラクション処理
fn button_system(
    // Interactionが「変更された」エンティティだけを抽出（負荷軽減）
    mut interaction_query: Query<
        (&Interaction, &mut BackgroundColor, &Children),
        (Changed<Interaction>, With<Button>),
    >,
    // テキストを書き換えるために必要
    mut text_query: Query<&mut Text, With<PauseButtonText>>,
    // シミュレーションの状態
    mut sim_state: ResMut<SimulationState>,
) {
    for (interaction, mut bg_color, children) in &mut interaction_query {
        // ボタンの子要素（テキスト）を取得
        // children[0]がテキストのエンティティ
        let mut text = text_query.get_mut(children[0]).unwrap();

        match *interaction {
            // クリックされた瞬間
            Interaction::Pressed => {
                // 状態を反転
                sim_state.is_running = !sim_state.is_running;
                
                // 色とテキストを更新
                if sim_state.is_running {
                    *bg_color = Color::srgb(0.2, 0.2, 0.2).into(); // 通常色
                    text.0 = "Pause".to_string();
                } else {
                    *bg_color = Color::srgb(0.7, 0.1, 0.1).into(); // 赤色（停止中）
                    text.0 = "Resume".to_string();
                }
            }
            // マウスが乗った時
            Interaction::Hovered => {
                // 少し明るくして反応を示す
                if sim_state.is_running {
                    *bg_color = Color::srgb(0.3, 0.3, 0.3).into();
                }
            }
            // マウスが離れた時
            Interaction::None => {
                 // 元の色に戻す（停止中なら赤のまま）
                 if sim_state.is_running {
                    *bg_color = Color::srgb(0.2, 0.2, 0.2).into();
                 } else {
                    *bg_color = Color::srgb(0.7, 0.1, 0.1).into();
                 }
            }
        }
    }
}
```

## Step5: Appに追加
最後に main 関数で、新しいリソースとシステムを登録します。
```rust
fn main() {
    App::new()
        .insert_resource(ClearColor(Color::BLACK))
        // ★初期状態は「動いている(true)」
        .insert_resource(SimulationState { is_running: true }) 
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, (setup, setup_ui))
        // ★ button_system を追加
        .add_systems(Update, (rotate_planet, move_camera, button_system).chain()) 
        .run();
}
```

## 実行
cargo run してみてください。

1. 画面右下に「Pause」ボタンが表示されていますか？
2. ボタンにマウスを乗せると色が少し変わりますか？（Hover）
3. ボタンをクリックすると、地球の自転が止まり、ボタンが赤くなって「Resume」に変わりますか？
4. もう一度押すと再開しますか？