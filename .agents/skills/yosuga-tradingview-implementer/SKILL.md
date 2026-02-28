---
name: yosuga-tradingview-implementer
description: Use when implementing or updating TradingView Pine indicators based on Yosuga-style Dow theory, multi-timeframe trend recognition, and acceleration/deceleration state analysis.
---

# Yosuga TradingView Implementer

このスキルは、よすが式の考え方を TradingView / Pine Script に実装するときに使う。

## Core Idea

よすが式では、トレンド認識を最優先に扱う。

- ダウ構造が主判断
- ラインやゾーンは補助根拠
- 上位足は環境認識
- 下位足はタイミング
- 出力はシグナル断定ではなく状態把握

## Must Follow

1. ラインブレイクだけで転換と判定しない。
2. `HH`, `HL`, `LH`, `LL` を明示してトレンドを定義する。
3. 上位足の文脈と下位足のトリガーを分離する。
4. 未確定ピボットを確定済みとして扱わない。
5. 構造が曖昧なら `NEUTRAL` か watch 状態を返す。
6. ユーザー指定がない限り `indicator()` を使う。

## Implementation Order

1. スイング高値・安値を検出する。
2. ダウ構造から上昇、下降、中立、転換候補を定義する。
3. 上位足の方向と勢いを判定する。
4. 下位足で加速、減速、継続、失速を判定する。
5. 最後にライン、ゾーン、パターンを補助条件として追加する。

構造の正しさを、見た目やアラートより優先する。

## Code Shape

Pine コードは、できるだけ以下の形に分ける。

### Inputs
- ピボット感度
- 上位足
- EMA / ATR などの長さ
- 加速 / 減速の閾値
- 表示とアラートの ON/OFF

### Structure
- 確定スイング高値
- 確定スイング安値
- 直近と一つ前の比較
- `HH`, `HL`, `LH`, `LL` の判定

### Higher Timeframe
- bullish
- bearish
- neutral
- accelerating
- decelerating

### Momentum
- レッグ長の比較
- ATR 比の拡大縮小
- EMA 傾き
- ブレイクの質

### Confirmation
- トレンドライン
- 水平帯
- プルバックゾーン
- ダブルトップ / ダブルボトム
- ヘッドアンドショルダー

補助根拠でダウ構造の判定を上書きしてはいけない。

### Output

出力状態は少なくとも以下を含める。

- `ACCEL`
- `DECEL`
- `NEUTRAL`
- `BULLISH CONTINUATION`
- `BEARISH CONTINUATION`
- `REVERSAL WATCH`

可能なら、状態の理由も見えるようにする。

- structure
- higher timeframe bias
- momentum condition
- line or zone condition

## Coding Rules

- 重要ロジックは意味のある変数名に分解する。
- 繰り返し処理は helper function に切り出す。
- 複雑な意図だけ短いコメントを付ける。
- 短すぎる式に圧縮しない。
- 確定条件と暫定条件を名前で分ける。
- 非リペイント性をできるだけ保つ。

## Visualization

- 背景色やバー色は高レベル状態に使う。
- ラベルは重要な確定イベントだけに使う。
- テーブルは状態要約に使う。
- アラートは意味のある状態遷移だけに使う。

## Avoid

- ラインブレイクをそのまま転換とみなすこと
- パターンだけで売買を決めること
- 中立局面で無理に方向シグナルを出すこと
- 閾値を増やしすぎてロジックを不透明にすること
- 見た目だけ派手で状態解釈できない実装
