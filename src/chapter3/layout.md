# レイアウトの基礎
画面下部にステータスバーを作ってみましょう。

前回、ルートノード（親）に `flex_direction: Column`（縦並び）を指定しました。 今回は、親ノードの中での配置ルール（`justify_content`）を `SpaceBetween` に変更します。

SpaceBetween: 最初の子要素を「上端」、最後の子要素を「下端」に配置し、その間を等間隔に空けます。

しかし、前回のままだと「タイトル」と「サブタイトル」が別々の子要素になっているため、 `[タイトル] --- (空き) --- [サブタイトル] --- (空き) --- [ステータスバー]` のようにバラバラになってしまいます。

そこで、「タイトルとサブタイトル」を一つの 透明な箱（Node） にまとめて、それを「上部エリア」として扱います。

## `setup_ui`の書き換え
```rust
fn setup_ui(mut commands: Commands) {
    commands.spawn((
        Node {
            width: Val::Percent(100.0),
            height: Val::Percent(100.0),
            flex_direction: FlexDirection::Column, // 縦並び
            // ★重要: 中身を「両端（上と下）」に振り分ける設定
            justify_content: JustifyContent::SpaceBetween, 
            ..default()
        },
    ))
    .with_children(|parent| {
        // ==================================================
        // 1. 上部エリア（ヘッダー）のグループ
        // ==================================================
        parent.spawn((
            Node {
                // 縦並び
                flex_direction: FlexDirection::Column,
                // 左上に余白をつける
                padding: UiRect::all(Val::Px(20.0)),
                ..default()
            },
        )).with_children(|header| {
            // タイトル
            header.spawn((
                Text::new("Earth Orbit Simulator"),
                TextFont { font_size: 40.0, ..default() },
                TextColor(Color::WHITE),
            ));
            // サブタイトル
            header.spawn((
                Text::new("Press WASD to move..."),
                TextFont { font_size: 20.0, ..default() },
                TextColor(Color::srgb(0.8, 0.8, 0.8)),
                Node {
                    margin: UiRect::top(Val::Px(5.0)), // タイトルとの間隔
                    ..default()
                }
            ));
        });

        // ==================================================
        // 2. 下部エリア（ステータスバー）
        // ==================================================
        parent.spawn((
            Node {
                width: Val::Percent(100.0),
                height: Val::Px(50.0), // 高さを固定
                // 横並びにする
                flex_direction: FlexDirection::Row, 
                // 垂直方向の中央揃え
                align_items: AlignItems::Center,
                // 水平方向の左寄せ（要素同士の間隔設定も可能）
                justify_content: JustifyContent::FlexStart,
                // 内側の余白
                padding: UiRect::horizontal(Val::Px(20.0)),
                // 子要素同士の隙間（モダンなCSSのgapプロパティと同じ）
                column_gap: Val::Px(30.0), 
                ..default()
            },
            // ★背景色をつけるコンポーネント
            BackgroundColor(Color::srgba(0.1, 0.1, 0.1, 0.8)), 
        )).with_children(|footer| {
            // 左側の情報: ステータス
            footer.spawn((
                Text::new("Status: In Orbit"),
                TextFont { font_size: 18.0, ..default() },
                TextColor(Color::WHITE),
            ));

            // 右側の情報: 速度（仮の値）
            footer.spawn((
                Text::new("Speed: 29.78 km/s"),
                TextFont { font_size: 18.0, ..default() },
                TextColor(Color::WHITE),
            ));
        });
    });
}
```