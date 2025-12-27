# Bevyとは？
ざっくり、ECSアーキテクチャを採用したRust製のOSSゲームエンジン。UEやUnityのように専用エディタは存在せず、ひたすらRustコーディングをすることで開発ができる。

## ECSとは？
データとロジックを分離することで高いパフォーマンス、並列処理の容易さ、高い拡張性と保守性などが特徴。<br>
多数の類似したエンティティを扱うのが得意で、例えば粒子シミュレーションの最適解<br>
一方で、OOPと比較すると難易度は上がる。<br>
|ECS|意味|説明|例|
|---|---|---|---|
|E|Entity: ID(行)|ただのIDで、中身は空っぽ|ID: 1(惑星), ID: 2(宇宙船)　|
|C|Component: データ(列)|Entityにくっつけるデータで、structやenumそのもの|Position { x: 0.0, y: 0.0, z: 0.0 }, RenderMesh(見た目の3Dモデル) |
|S|System: ロジック(関数)|全データをスキャンして更新する関数。|fn move_objects(...): 「Position」「Velocity」の両方を持っている全Entityを探し出し、位置を更新するなど|

## インストール
```bash
cargo new space_sim
cd space_sim
cargo add bevy
```

## examples
![alt text](../images/sample1.gif)
![alt text](../images/sample2.gif)
![alt text](../images/sample3.gif)