# Godot 4.7 チュートリアル（初心者向け・日本語版）

Godot 4.7 の公式ドキュメント（Getting started）を元に、4.7の画面・初心者の操作を前提に書き直したものです。
言語は **GDScript**（Godot専用の、初心者向けの言語）で統一しています。

## 構成（この順に進めてください）

| 章 | 内容 | 所要時間の目安 |
|---|---|---|
| 1 Introduction | Godotとは何か、基本概念、エディタの画面 | 30分 |
| 2 Step by step | ノードとシーン、インスタンス化、スクリプト、シグナル | 2〜3時間 |
| 3 Your first 2D game | 2Dゲーム「Dodge the Creeps!」を作る | 3〜4時間 |
| 4 Your first 3D game | 3Dゲーム「Squash the Creeps」を作る | 4〜6時間 |

## 記号の意味

- ✅ 確認: ここまでの操作で、こうなっていれば成功
- ⚠ つまずき: 初心者がよくハマるところと対処
- ※ 補足: 公式の4.7ドキュメントには書かれていない、補足説明（Godotの一般的な仕様に基づく）

## このチュートリアルの正確さについて

- 手順・数値・コードは、4.7の公式ドキュメントのソース（godot-docs の 4.7 ブランチ）に基づいています。
- 実機の4.7エディタで1行ずつ確認したものではありません。画面が違う箇所があれば、メニュー名やボタン名（英語表記）を手がかりに探してください。
- 元ドキュメントは CC BY 3.0 ライセンスです。
- 4.7.2（macOS）の実機で、一部の画面と手順を確認済みです（プロジェクト作成、ノードの追加、実行、メインシーン、Project Settings、Signals タブ）。確認の結果、Asset Store の名称、メインシーンのダイアログ、Stretch の初期値、Autoload の画面、サンプルのシーン名（`Main.tscn` / `Ball.tscn`）、Sprite Frames の位置、3D素材の `.blend` ダイアログ、折りたたまれた Inspector の項目（Transform / Theme Overrides / Layout など）、FPS 欄、Groups タブの位置、Layer Names の場所、Path2D のツールバー、Shortcut / Theme の折りたたみ、Animation パネルのアイコンを直しました。付録の完成コード（2D と 3D）は、4.7.2 で実際に動かして確認しました。
- 画像（`images/` フォルダ）は Godot 公式ドキュメント（4.7 ブランチ）のものを、CC BY 3.0 のもとで利用しています。© Juan Linietsky, Ariel Manzur and the Godot community。一部は 4.4〜4.6 世代の画面で、タブ名などが 4.7 と少し違うことがあります。文章の記述を優先してください。

## 事前準備

1. Godot 4.7 を https://godotengine.org/download から入手します。
   - **標準版（Standard）** を選んでください。「.NET」版はC#用なので不要です。
   - ※インストーラーは不要です。ダウンロードした実行ファイルをそのまま起動します（macOSは .app をアプリケーションフォルダに移すと便利）。
2. 各章で使う素材（先にダウンロードしておくと楽です）

| 章 | 素材 | URL |
|---|---|---|
| 2章（インスタンス化） | instancing_starter.zip | https://github.com/godotengine/godot-docs-project-starters/releases/download/latest-4.x/instancing_starter.zip |
| 3章（2Dゲーム） | dodge_the_creeps_2d_assets.zip | https://github.com/godotengine/godot-docs-project-starters/releases/download/latest-4.x/dodge_the_creeps_2d_assets.zip |
| 4章（3Dゲーム） | 3d_squash_the_creeps_starter.zip | https://github.com/godotengine/godot-docs-project-starters/releases/download/latest-4.x/3d_squash_the_creeps_starter.zip |

   - 2章の素材は Godot 4.2 で作られたため、4.7で開くと「古いバージョンのプロジェクト」という警告が出ます。問題ありません。

## よく使うショートカット

| 操作 | Windows / Linux | macOS |
|---|---|---|
| プロジェクトを実行（メインシーン） | F5 | Cmd + B |
| 今開いているシーンを実行 | F6 | Cmd + R |
| 実行を停止 | F8 | Cmd + . |
| 保存 | Ctrl + S | Cmd + S |
| ノードを複製 | Ctrl + D | Cmd + D |
| 子ノードを追加 | Ctrl + A | Cmd + A |
| ノード名の変更 | F2 | F2 |
| ヘルプ検索（クラスリファレンス） | F1 | Opt + Space |
| プロジェクト一覧に戻る | Ctrl + Shift + Q | Ctrl + Option + Cmd + Q |
| 3Dビューを2分割 | Ctrl + 2 | Cmd + 2 |
| 2D画面へ切り替え | Ctrl + F1 | Ctrl + Cmd + 1 |

## コードを貼り付けるときの注意（重要）

GDScriptは **インデント（行頭の字下げ）が文法** です。
Godotのコードエディタは標準でタブ文字を使います。スペースとタブが混ざると
「Mixed use of tabs and spaces」というエラーになります。

- このチュートリアルのコードブロックは、タブでインデントしてあります。
- コードを**手で打つ**とき: コードエディタは字下げと閉じカッコを自動で入れ、補完の候補を出します。字下げは足さず、候補が出ているときの Enter は「候補の確定」になるので、Esc で閉じてから押してください。**貼り付ける**ほうが確実です。
- 貼り付け後にエラーが出たら、スクリプトエディタのメニュー Edit > Convert Indent to Tabs を実行してください（※）。
- 「Indented block expected」というエラーは、字下げが足りない行があるという意味です。
