# Yosuga Accel/Decel State — 要件定義書（TradingView実装・単一ファイル版）

**名称**：Yosuga Accel/Decel State (for TradingView)  
**目的**：よすが式ダウ手法に基づき、相場の“勢い”を「加速中（ACCEL）／減速中（DECEL）／中立（NEUTRAL）」として可視化・通知し、初動～追随のトレンドフォローを補助する。理論（トレンド定義、ライン、MTF）と実践（環境認識→シナリオ→PA→運用）の考え方を実装要件に落とし込む:contentReference[oaicite:0]{index=0}:contentReference[oaicite:1]{index=1}。

---

## 1. コンポーネント（TradingView / Pine v5）
- **インジケーター**（`indicator()`）  
  - メインチャートの**色帯（bgcolor）**、**状態ラベル**、**上位足方向**表示  
  - サブパネルに**合成スコア**（0基準オシレーター）  
  - **アラート**：状態遷移、帯接近、HTF矛盾
- **（任意）戦略**：別ファイルでの`strategy()`化も可能（本ファイルはインジ専用）

---

## 2. 入出力（I/O）
### 入力（ユーザ設定）
- **時間足**：`tf_htf`（上位足）:contentReference[oaicite:2]{index=2}  
- **スイング検出**：`pivotLeft/right`（ダウ高安抽出の基礎）:contentReference[oaicite:3]{index=3}  
- **勢いスコア**：`emaLen`, `atrLen`, `volLen`  
- **しきい値**：`legRatioUp`（>1.10）, `legRatioDn`（<0.90）, `T_accel`, `T_decel`, `minHold`  
- **プライスアクション検出**：`detectZZ`（推進）、`detectDT`（ダブルトップ/ボトム）、`detectHS`（H&S）:contentReference[oaicite:4]{index=4}  
- **ライン帯**：`autoLevels`（簡易水平抽出ON/OFF）, `bandMerge`（近接統合幅%）, `levelExpiry`（2回抜けて無効化）:contentReference[oaicite:5]{index=5}  
- **表示/アラート**：色濃度、サブパネル表示、各アラートON/OFF

### 出力
- **状態**：`ACCEL` / `DECEL` / `NEUTRAL`  
- **メタ**：合成スコア、素点内訳（脚長・ATR・出来高・EMA傾き・PA）、HTF一致  
- **描画**：状態色帯・スコア・簡易水平帯  
- **アラート**：遷移、帯±X%内、HTF矛盾

---

## 3. ロジック定義
### 3.1 トレンド方向（ダウ理論）
- `ta.pivothigh/low`で**高値/安値**を抽出 → **高値更新/安値更新**で方向判定:contentReference[oaicite:6]{index=6}  
- `request.security(tf_htf, ...)`で**上位足方向**を取得・加点:contentReference[oaicite:7]{index=7}

### 3.2 ライン/帯（有利位置の文脈）
- 直近スイング群の**目立つ高値/安値**を簡易抽出→近接は**帯**に統合、**2回明確に抜ければ無効化**:contentReference[oaicite:8]{index=8}

### 3.3 勢いの素点（加点/減点）
- **脚長比**：`leg_ratio = |last_leg| / |prev_leg|`（拡大→＋、縮小→−）  
- **ATR変化**：`ΔATR`（増→＋、減→−）  
- **出来高拡張度**：`volExp = vol / SMA(vol) - 1`（増→＋、減→−）  
- **EMA傾き**：`slope = EMA - EMA[1]`（方向一致→＋、不一致→−）  
- **PA**：推進ZZ→＋、DT/H&S→−:contentReference[oaicite:9]{index=9}

### 3.4 合成スコアと状態
- `Score = Σ(素点×重み) + HTF一致ボーナス`  
- `Score ≥ T_accel → ACCEL`／`Score ≤ -T_decel → DECEL`／他→`NEUTRAL`  
- **ヒステリシス**：`minHold`本は状態維持。上位足帯至近では**逆方向ACCELを抑制**:contentReference[oaicite:10]{index=10}

---

## 4. 描画・UI・アラート
- **色帯**：ACCEL=緑、DECEL=赤、NEUTRAL=灰（濃度=|Score|正規化）  
- **サブパネル**：Score棒グラフ／内訳ライン  
- **アラート**：状態遷移／帯±X%内でのACCEL/DECEL／HTF矛盾

---

## 5. 初期値（推奨）
- `emaLen=20`, `atrLen=14`, `volLen=20`  
- `legRatioUp=1.10`, `legRatioDn=0.90`  
- `w_leg=1.0`, `w_ATR=1.0`, `w_vol=0.5`, `w_slope=0.5`, `w_pa=1.0`, `w_htf=0.5`  
- `T_accel=+2`, `T_decel=−2`, `minHold=2`  
- `autoLevels=ON`, `bandMerge=0.15%`, `levelExpiry=2`

---

## 6. 制約・例外
- FXの出来高は**ティック出来高**（相対評価）:contentReference[oaicite:11]{index=11}  
- 上位足の確定値のみ使用（リペイント回避）  
- 自動ラインは**補助**であり、手動ラインを邪魔しない:contentReference[oaicite:12]{index=12}

---

## 7. 受入れ基準（TradingView上）
1) 初動～推進局面でACCELが継続し、上位足帯逆らい抑制が発動:contentReference[oaicite:13]{index=13}  
2) 弱化/転換前にDECELが増え、DT/H&Sと整合:contentReference[oaicite:14]{index=14}  
3) レンジではNEUTRAL優勢:contentReference[oaicite:15]{index=15}  
4) アラートが定義通りに発火し、誤発火が少ない  
5) 遅延/スクリプト制限超過がない

---

## 8. 参照
- **理論編**：トレンド定義・ライン・MTF・決済基準:contentReference[oaicite:16]{index=16}  
- **実践編**：環境認識→シナリオ→PA→運用、時間足ペア、ライン6ステップ:contentReference[oaicite:17]{index=17}
