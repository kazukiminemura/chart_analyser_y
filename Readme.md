# Yosuga Acceleration State Analyzer

Yosuga Acceleration State Analyzer は yosuga_acceleration_state.pine に実装された Pine Script v5 のインジケーターで、加速/減速状態を多面的に判定してチャート上へ可視化します。

## 主な機能
- スイング高値/安値の更新状況からトレンド構造を解析
- ATR・出来高・EMA 乖離をスコア化し、ACCEL/DECEL/NEUTRAL 状態を判定
- 状態変化時にシナリオ矢印と利確目標を描画
- 最新状態を日本語ラベル（加速中/減速中/中立）で表示し、テーブルでコンポーネントの内訳を確認
- スイング高値/安値ライン、アラート条件、マルチタイムフレーム設定をサポート

## 使い方
1. TradingView で Pine Editor を開き、yosuga_acceleration_state.pine の内容を貼り付けます。
2. チャートへ適用すると、状態ラベル・背景色・シナリオ矢印が表示されます。
3. 入力パラメーターから以下を調整できます。
   - スイング検出 (Pivot Length, Minimum Bars To Hold State)
   - トレンド判定タイムフレーム (Higher Timeframe, Signal Timeframe, MTF Alignment Weight)
   - コンポーネント重み (Swing Leg Weight, ATR Weight, Volume Weight, Price Action Weight など)
   - シナリオ描画 (Show Scenario Arrows, Take Profit ATR Multiple, Arrow Projection Bars ほか)
   - 表示制御（日本語テキスト、ライン描画、アラートなど）

## 注意事項
- 出来高データが存在しない銘柄では Use Volume Component をオフにしてください。
- TradingView のライン・ラベル数制限に達しないよう、Max Scenario Marks や Max Historical Lines Per Side を環境に合わせて調整してください。
- 状態スコア（score）はチャートへプロットしていません。必要に応じて plot(score, ...) を追加してください。

## アラート設定
State: ACCEL / State: DECEL / State: NEUTRAL の 3 種類のアラート条件を用意しています。状態変化に合わせて通知を受け取りたい場合は、チャート上のインジケーター設定から該当アラートを追加してください。
