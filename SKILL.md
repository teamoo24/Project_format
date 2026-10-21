---
name: report-game
description: Report-style game building (Godot or similar). The agent builds one change at a time and reports it; the user plays, judges, and decides direction. Prefer editor features (signals, export, AnimationPlayer, groups) over code-only. Use when starting a new game, filling templates from an idea (アイデアから型), writing AGENTS.md / PLAN.md / README.md, or when the user says 次 / 次の手 / OK.
---

# 報告式ゲーム制作

エージェントが作って報告する。本人は F5 で遊んで判断し、方向を決める。

新しいゲームを始めるときは、作る前に型を埋める。聞き方と埋め方は [README.md](README.md) の「アイデアから型」。ゲームのレポに `AGENTS.md` があるときは、そちらを優先する。

- [AGENTS.md の型](agents-template.md)
- [PLAN.md の型](plan-template.md)
- [README.md の型](readme-template.md)

## 3ファイルの役割

| ファイル | 役割 |
|---|---|
| `AGENTS.md` | エージェント向け。作り方、事故を防ぐ、報告の形、今作だけのこと |
| `PLAN.md` | 全体の地図。「次の手」は小さな変更の列 |
| `README.md` | 本人向けの今のメモ。「できたこと」「いまの手」 |

## 1回の流れ

1. `PLAN.md` の「次の手」の先頭を1つ選ぶ（本人が別の手を指定したらそちら）
2. 作る。F5 で見える変化が1つになる大きさ
3. 報告する（下の3点セット）
4. 本人の OK を待つ。OK なら README と PLAN を更新し、commit / push

## 報告の3点セット

1. **変えたこと**: ファイル、ノード、Inspector の値
2. **F5 で見るところ**: 1文
3. **本人の残作業**: エディタでやること。なければ「なし」

エラー時に疑う点を1行。クリック場所は図で。短く。

## 本人が決めること

手触り・方向性、数値の最終決定、素材の採用、ノード構成の大きな変更（提案→了承）、投稿やタグ付け。

## 作り方と事故

- エディタの機能を先に使う。シーン内のシグナルはエディタで。`.connect()` は実行時に増える相手だけ
- `preload` 直書きより `@export` と Inspector
- 親クラスは同じ処理が3箇所以上になってから
- シーンは最初から役割ごとに分ける。同じものが2つ以上、UI がまとまった、の時点で別シーンにするよう先に提案する。切り出したら `@export`・NodePath・シグナルが外れていないか確かめる
- 編集前に `.tscn` で実際の親子関係を確かめる
- `@export` を足したら割り当てまで済ませるか、残作業に書く
- 素材は `assets/made/`（画像）、`assets/audio/`（音）

## しないこと

- 複数の機能をまとめて入れて「全部できた」にしない
- 本人の OK を待たずに次へ進まない
- 数値・素材を勝手に確定しない
- 参考を丸ごとコピーして今のシーンを捨てない
- 後回し（タイトル、セーブ、ギャラリーなど）を核より先に足さない