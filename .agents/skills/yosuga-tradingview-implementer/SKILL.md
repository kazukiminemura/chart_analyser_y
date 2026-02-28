---
name: yosuga-tradingview-implementer
description: Use when implementing or updating TradingView Pine indicators based on Yosuga-style Dow theory, multi-timeframe trend recognition, line-assisted scenario building, and acceleration/deceleration state analysis.
---

# Yosuga TradingView Implementer

このスキルは、このリポジトリ内でよすが式の考え方を TradingView / Pine Script に落とし込むときに使う。

まず [docs/requirement.md](../../docs/requirement.md) を読み、必要に応じて [yosuga_dow.txt](../../yosuga_dow.txt) と既存の Pine 実装を参照する。すべてを一度に読まず、今回の修正に必要な範囲だけ読むこと。

## Read This First

参照優先順位は以下。

1. [docs/requirement.md](../../docs/requirement.md)
2. [yosuga_acceleration_state.pine](../../yosuga_acceleration_state.pine)
3. [trendline_entryzone.pine](../../trendline_entryzone.pine)
4. [yosuga_dow.txt](../../yosuga_dow.txt)

[yosuga_dow.txt](../../yosuga_dow.txt) は文字化けして見えることがあるため、その場合は既存実装と要件書を優先して意味を復元する。

## Core Interpretation

このリポジトリでのよすが式は、以下の原則で扱う。

- トレンド認識はダウ構造が最上位である。
- ライン、ゾーン、チャートパターンは補助根拠であり、主判定にはしない。
- 上位足は環境認識、下位足はタイミングという役割を分ける。
- 実装は売買の断定機ではなく、シナリオ判断を助けるものにする。
- 出力は加速、減速、継続、転換警戒、無効文脈を読み取れるようにする。

## Non-Negotiable Rules

実装時は必ず以下を守る。

1. ラインブレイクだけでトレンド転換とみなさない。先にスイング構造でトレンドを定義する。
2. `HH`, `HL`, `LH`, `LL` をコード上で明示し、構造判定を追跡できるようにする。
3. 環境認識とエントリー条件を分離する。上位足の文脈と下位足のトリガーを混ぜない。
4. `request.security()` を使うときは、上位足状態を独立した変数や状態として露出させる。
5. ピボットや MTF 参照は確定遅延を前提とし、未確定情報を確定済みとして扱わない。
6. ユーザーが明示しない限り `strategy()` ではなく `indicator()` を使う。
7. 構造が曖昧なときは無理に方向を出さず `NEUTRAL` または watch 状態を返す。

## Preferred Workflow

作業順は以下。

1. 現状の Pine コードを読み、既存の命名と状態遷移を把握する。
2. スイング高値・安値の検出方法を確認する。
3. ダウ構造からトレンド状態を定義する。
4. 上位足の文脈を追加または検証する。
5. 加速・減速・継続・転換警戒の条件を設計する。
6. その後にライン、ゾーン、パターン、アラートを接続する。
7. 表示がノイジーになっていないか確認する。

構造の正しさを、見た目やアラートより優先する。

## Required Code Shape

Pine 実装は、できるだけ以下のブロックに分ける。

### Inputs
- ピボット感度
- 上位足
- トレンド・モメンタム関連の長さ
- 加速・減速の閾値
- パターン検出の ON/OFF
- 表示の ON/OFF
- アラートの ON/OFF

### Structure Detection
確認済みピボットまたは同等の確定ロジックで、以下を追跡する。

- 最新の確定スイング高値
- 最新の確定スイング安値
- ひとつ前の確定スイング高値
- ひとつ前の確定スイング安値
- スイング列の方向性

ここから少なくとも以下を導出する。

- 上昇トレンド
- 下降トレンド
- 転換候補
- レンジまたは中立

### Higher Timeframe Context
上位足状態は、少なくとも以下のいずれかとして読めるようにする。

- bullish
- bearish
- neutral
- accelerating
- decelerating

1 つの巨大な真偽値に押し込まず、読める状態に分解する。

### Momentum And State
加速・減速の判定には、必要に応じて以下を使う。

- レッグ長の比較
- ATR 比の拡大縮小
- EMA 傾き
- 出来高拡大
- パターン成立または失敗
- レベル突破の質

マジックナンバー 1 行判定より、加点式または分離された条件を優先する。

### Confirmation Layer
以下は補助根拠としてのみ使う。

- トレンドライン
- サポート/レジスタンス帯
- プルバックゾーン
- ダブルトップ / ダブルボトム
- ヘッドアンドショルダー
- ブレイク確認
- ヒゲや拒否反応の質

これらでダウ構造の判定を上書きしてはいけない。

### Output Layer
出力状態は、少なくとも以下のように解釈可能であること。

- `ACCEL`
- `DECEL`
- `NEUTRAL`
- `BULLISH CONTINUATION`
- `BEARISH CONTINUATION`
- `REVERSAL WATCH`

可能なら理由も露出する。

- structure
- higher timeframe bias
- momentum condition
- line or level condition
- pattern condition

## Coding Rules

- 重要ロジックは意味のある変数名で分解する。
- 繰り返し処理は小さな helper function に切り出す。
- コメントは複雑な意図の説明に限定する。
- 短すぎる式に圧縮して読みづらくしない。
- 確定条件と暫定条件を名前で区別する。
- 非リペイント性をできるだけ維持する。
- ユーザーが理解できない隠れた状態遷移を作らない。

## Visualization Rules

- 背景色やバー色は高レベル状態の表現に使う。
- ラベルは重要な確定イベントに限定する。
- テーブルは状態要約に使う。
- トレンドラインやゾーン表示は ON/OFF 可能にする。
- アラートは意味のある状態遷移に限定する。

以下は避ける。

- 毎バーのノイジーなラベル
- 意味が分からない多色表示
- 弱い暫定状態へのアラート

## Alert Rules

アラート条件は、例えば以下に限定する。

- 加速への確定遷移
- 減速への確定遷移
- 上位足と下位足の整合
- プルバック後またはゾーン内からの有効ブレイク
- 構造悪化後の reversal watch 発火

アラート文には、方向、状態、時間足文脈、主な確認理由を含める。

## Decision Heuristics

- 構造が曖昧なら `NEUTRAL` を優先する。
- ライン根拠とダウ構造が衝突したらダウ構造を優先する。
- 下位足モメンタムが上位足文脈に逆行するなら、逆張りまたは低信頼として扱う。
- 根拠が割れるなら trade signal ではなく watch state を返す。

## Anti-Patterns

以下の実装は避ける。

- ラインブレイクをそのまま転換とみなす実装
- パターン検出だけで buy/sell を出す実装
- 確定ピボットと未確定ピボットを無表示で混在させる実装
- 閾値を増やしすぎて市場ロジックが読めなくなる実装
- 中立局面でも二択シグナルを強制する実装
- 見た目だけ派手で、状態解釈ができない実装

## Repository Context

このリポジトリでは、以下と命名・概念を揃える。

- `yosuga_acceleration_state.pine`
- `trendline_entryzone.pine`
- `docs/requirement.md`
- `yosuga_dow.txt`

ユーザーが明示的に再設計を求めない限り、既存用語と状態名の整合性を維持する。
