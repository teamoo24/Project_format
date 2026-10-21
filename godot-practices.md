# Godot 4 ベストプラクティス（報告式テンプレート用）

公式マニュアル（stable, 2026-10-21 時点）を読んで、エージェントが守るべきことだけを抜き出した。
各節の末尾に出典を書く。日本語版は https://docs.godotengine.org/ja/4.x/ の同じパス。

## 1. シーン構成
- シーンはできるだけ依存なしで作る。必要なものは自分の中に持たせる。
- 外とやり取りが要るときは、親が子に渡す（依存性注入）。子から外を探しに行かない。
- 子から親への連絡はシグナル。シグナルは「起きたことへの反応」に使い、処理の開始には使わない。
- シグナル名は過去形の動詞にする（例：`entered`, `item_collected`）。
- 処理を始めさせたいときは、親が子のメソッドを呼ぶ。
- 兄弟ノード同士は互いを知らない。共通の親（所有者）が両者をつなぐ。
- データはなるべくシーンの内側に置く。外部に依存すると壊れやすい。
- 後から枝を別シーンに切り出すと、NodePath とエディタのシグナル接続が外れる。最初から分ける。
出典: https://docs.godotengine.org/en/stable/tutorials/best_practices/scene_organization.html

## 2. シーンとスクリプトの使い分け
- 見た目や構成を持つもの（ゲーム固有の物）はシーンにする。シーンは編集も追跡もしやすい。
- シーン（PackedScene）は直列化データからまとめて生成されるので、コードで同じ物を組むより速い。
- いろいろなプロジェクトで使い回す道具や汎用部品は、`class_name` 付きスクリプトにする。
出典: https://docs.godotengine.org/en/stable/tutorials/best_practices/scenes_versus_scripts.html

## 3. Autoload と普通のノード
- マネージャー的な Autoload を安易に作らない。ノードツリーとシグナルでグローバル状態は避けられる。
- Autoload のデータは全シーンから触れるので、バグが出たとき原因を絞りにくい。
- 補助関数だけなら `class_name` ＋ `static func`、共有値なら `static var` で足りる。
- Autoload が向くのは、自分のデータだけを管理する広域システム（クエスト、会話など）。
出典: https://docs.godotengine.org/en/stable/tutorials/best_practices/autoloads_versus_regular_nodes.html

## 4. ノードを使わない選択
- 大量の単純データにノードを使わない。Object / RefCounted / Resource の方が軽い。
- 通常のデータクラスは RefCounted で十分（自動で解放される）。
- Inspector で編集したい・保存したいデータは Resource にする。
- 素の Object は手動解放。参照が知らないうちに無効になることがある。
出典: https://docs.godotengine.org/en/stable/tutorials/best_practices/node_alternatives.html

## 5. 通知とコールバック
- フレームごとの見た目の更新は `_process()`。
- 移動や物理的な変換は `_physics_process()`（一定間隔で呼ばれる）。
- 入力は `_input` 系で受け、毎フレーム入力を調べるのは避ける。
- シーンツリーに依存しない初期化は `_init()`。子ノードに触る初期化は `_ready()`。
- 毎フレーム不要な処理は Timer などで間引く。
出典: https://docs.godotengine.org/en/stable/tutorials/best_practices/godot_notifications.html

## 6. データとロジックの選び方
- 検索が多いなら Array より Dictionary（キー検索は定数時間）。
- 列挙は enum（整数）を使う。表示用の文字列が要るときだけ文字列を検討する。
- 単純なコマ送りは AnimatedSprite2D、複数プロパティやタイミング付き演出は AnimationPlayer、遷移が複雑なら AnimationTree。
- `@export var x: PackedScene = preload(...)` は無駄。export の初期値は null にして Inspector で割り当てる。
- `preload` は定数パスでロード時に読み込む。`load` は実行時。巨大なものや条件付きは `load`。
出典: https://docs.godotengine.org/en/stable/tutorials/best_practices/data_preferences.html / https://docs.godotengine.org/en/stable/tutorials/best_practices/logic_preferences.html

## 7. プロジェクトの整理
- 素材はそれを使うシーンの近くに置く（例：`characters/player/` に画像とシーンとスクリプト）。
- フォルダ名・ファイル名は snake_case。Windows でのエクスポート後の大文字小文字問題を避ける。
- ノード名は PascalCase。
- 外部の素材やプラグインは最上位の `addons/` に置く。
- 取り込みたくないフォルダ（資料など）には空の `.gdignore` を置く。
出典: https://docs.godotengine.org/en/stable/tutorials/best_practices/project_organization.html

## 8. バージョン管理
- `.godot/` は git に入れない（キャッシュ）。`*.translation` も除外。
- `.gitignore` と `.gitattributes` は「プロジェクト > バージョン管理 > メタデータを生成」で作る（LF を強制）。
- Windows で自前設定なら `core.autocrlf input`。
- 大きな画像・音を多く扱うなら、最初のコミット前に Git LFS を設定する。
出典: https://docs.godotengine.org/en/stable/tutorials/best_practices/version_control_systems.html

## 9. GDScript スタイル
- インデントはタブ、改行は LF、UTF-8（BOM なし）、末尾に改行1つ。
- 複数行の配列・辞書・enum は最後にもカンマを付ける（差分がきれい）。
- 1行に1文。条件の不要な括弧は付けない。`&&` ではなく `and`。
- 長い式の折り返しは括弧で行い、継続行はインデント2段。
- 命名：ファイル・関数・変数・シグナルは snake_case、クラス・ノードは PascalCase、定数・enum 値は CONSTANT_CASE。
- コードの並び順：`class_name` → `extends` → `##` 説明 → signal → enum → const → static var → `@export` → 通常変数 → `@onready` → `_init` → `_enter_tree` → `_ready` → `_process` → `_physics_process` → 他の仮想関数 → 自作メソッド。
- 公開メンバーを先、`_` 始まりの非公開を後に。
出典: https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html

## 10. 静的型付け
- 変数・引数・戻り値に型を書く（`var hp: int`, `func hit(dmg: int) -> void`）。補完が効き、バグを早く見つけられる。
- 型推論は `:=` で書ける。
- 自作の型は `class_name` で登録して使う。
- `as` キャストは失敗すると null になる。型の判定は `is` を使う方が安全。
- 型付けを徹底するなら、プロジェクト設定で未型付けの警告を有効にする。
出典: https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/static_typing.html

## 11. @export
- export 変数には型を付けるか定数で初期化する。
- 調整値は `@export_range(min, max, step)` で範囲を付け、Inspector で触りやすくする。
- 項目が多いときは `@export_group` / `@export_subgroup` でまとめる。
- ノード参照は NodePath ではなく `@export var target: Node2D` のようにノード型で export できる。
- `##` のドキュメントコメントは Inspector のツールチップになる。
出典: https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_exports.html

## 12. シーン固有名（%名前）とグループ
- スクリプトから使うノードは「%としてアクセス」を付け、`%Name` で取る。ノードを移動してもパスを直さなくてよい。
- 固有名はそのシーンの中だけで有効。別シーンのノードには使えない。
- 種類でまとめて扱う物（敵、保存対象など）はグループに入れ、`get_tree().get_nodes_in_group()` や `call_group()` で扱う。
- グループ名は snake_case を推奨。
出典: https://docs.godotengine.org/en/stable/tutorials/scripting/scene_unique_nodes.html / https://docs.godotengine.org/en/stable/tutorials/scripting/groups.html

## 13. シグナル
- ノード同士の連絡はシグナルで行い、結合を弱くする。
- エディタの「ノード > シグナル」から接続できる。コードでは `signal_name.connect(callable)` を使う（文字列でなく補完が効く）。
- 自作シグナルは `signal health_changed(old_value: int, new_value: int)` のように型付き引数で宣言し、`emit()` で発火する。
- 定期処理には Timer ノードの `timeout` シグナルを使う。
出典: https://docs.godotengine.org/en/stable/getting_started/step_by_step/signals.html

## 14. Resource（データ）
- 敵やブロックのステータスなど、データは `class_name` 付きの Resource にして `.tres` で保存する。
- `.tres` はテキストで git 向き。エクスポート時はバイナリ化される。
- Resource は読み込みがキャッシュされ共有される。インスタンスごとに値を変えるなら `duplicate()` する。
- `.tres` はスクリプトのパスを持つので、スクリプトを動かすと壊れることがある。
出典: https://docs.godotengine.org/en/stable/tutorials/scripting/resources.html

## 15. 2D 物理
- StaticBody2D：動かない壁や床。Area2D：重なり検知とシグナル（`body_entered` など）。RigidBody2D：物理シミュレーション（力を加えて動かす）。CharacterBody2D：コードで動かす。
- CharacterBody2D の `position` を直接いじらない。`move_and_collide()` か `move_and_slide()` を使う。
- 物理の移動は `_physics_process()` で行う。
- 跳ね返りは `move_and_collide()` の衝突の法線で `velocity.bounce(normal)` する。
- `collision_layer` は自分がいる層、`collision_mask` は自分が見る層。既定はどちらも層1。
- 層には役割を決めて名前を付け（プロジェクト設定）、Inspector で設定する。
出典: https://docs.godotengine.org/en/stable/tutorials/physics/physics_introduction.html / https://docs.godotengine.org/en/stable/tutorials/physics/using_character_body_2d.html

## 16. 入力
- キーを直接見ず、プロジェクト設定の InputMap にアクションを作って `is_action_pressed("name")` で読む。
- 一度きりの操作はイベント（`_unhandled_input` など）、押しっぱなしは `Input.is_action_pressed()`。
出典: https://docs.godotengine.org/en/stable/tutorials/inputs/input_examples.html

## 17. UI
- 画面サイズに合わせる配置はアンカー、並べる配置はコンテナ（HBox/VBox/Grid/Margin など）。
- コンテナの子は自分で位置を決められない。位置を手で直さず、コンテナの設定で調整する。
- 見た目は Theme リソースでまとめる。テーマは親から子へ継承されるので、UI の根に1つ付ける。
出典: https://docs.godotengine.org/en/stable/tutorials/ui/gui_containers.html / https://docs.godotengine.org/en/stable/tutorials/ui/gui_skinning.html

## 18. セーブ
- セーブは `user://` に書く（`res://` はエクスポート後に書けない）。
- 保存対象のノードをグループ（例：`persist`）に入れ、各ノードに保存用データを返させる。
- 設定（音量など）は ConfigFile を使う。
- JSON は読みやすいが型が落ちる。小さく型を保ちたいならバイナリ直列化。
- 読む前に `FileAccess.file_exists()` で確認する。
出典: https://docs.godotengine.org/en/stable/tutorials/io/saving_games.html

## 19. パフォーマンス
- 推測で最適化しない。プロファイラと計測で一番のボトルネックを見つけてから直す。
- 直したら毎回計測し直す。
- まず設計（アルゴリズムとデータ構造）を正しくする。細かい最適化は後回し。
出典: https://docs.godotengine.org/en/stable/tutorials/performance/general_optimization.html

## 20. エクスポート
- エクスポートにはテンプレートのインストールとエクスポートプリセットが必要。
- `.` で始まるファイルやフォルダはエクスポートに含まれない。
- `export_presets.cfg`（4.1以降）は git に入れてよいが、パスワード等の秘密は `.godot/export_credentials.cfg` 側に入る。
- 形式は基本 PCK を使う。
出典: https://docs.godotengine.org/en/stable/tutorials/export/exporting_projects.html

---

## project-formatに入れるべき要点（既存ルールと重ならないもの中心）
1. シーンは依存なしで作り、外の物は親が渡す。子が外を探しに行かない。
2. 子→親はシグナル（過去形の名前）、親→子はメソッド呼び出し。兄弟は親がつなぐ。
3. Autoload は自分のデータだけを持つ広域システムに限る。補助関数は `class_name`＋`static func`。
4. 全スクリプトに静的型を付ける。シグナルの引数も型付き。`as` より `is`。
5. コードの並び順と命名はスタイルガイドに従う（ファイル snake_case、ノード PascalCase、定数 CONSTANT_CASE、タブ・LF）。
6. ゲームのデータ（ブロックHP、強化の値段など）は `class_name` 付き Resource の `.tres` にする。個体ごとに変えるなら `duplicate()`。
7. 調整値は `@export_range` と `@export_group` で Inspector から触りやすくする（本人が数値を決めやすい）。
8. export の初期値に `preload` を入れない。null にして Inspector で割り当てる。
9. 移動は `_physics_process()`、CharacterBody2D は `move_and_collide`/`move_and_slide` で動かし、`position` を直接いじらない。
10. collision layer/mask は役割ごとに層を決めて名前を付け、報告に書く。
11. 入力は InputMap のアクション経由。キーを直接見ない。
12. UI はコンテナとアンカーで組み、手で位置を合わせない。見た目は UI の根の Theme で統一。
13. セーブは `user://`、保存対象はグループで集める。設定は ConfigFile。
14. `.godot/` を git に入れない。.gitignore/.gitattributes はエディタで生成する。
15. 最適化は計測してから。素材はシーンの近くに置き、外部物は `addons/`。
