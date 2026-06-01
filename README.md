# MTA H1 EMA Filter Lab

## 概要

このリポジトリは、15分MTAセットアップ + 1分Triggerの既存Strategyに対して、1時間足EMA Position Filterを追加し、上位足環境フィルターとして有効かを検証するための研究ログです。

対象は以下の3系統です。

- B5.2系：Setup前の直近PivotをPre Break基準にする
- B1系：Setup後、最初に確定したPivotをBreak基準にする
- B2.1系：Setup後、有利方向に更新されるPivotだけをBreak基準にする

まずはB5.2に1H EMA Position Filterを追加し、結果が良ければB1 / B2.1へ横展開します。

---

## 目的

1時間足EMA Position Filterを追加することで、以下が改善するかを確認します。

- 勝率
- PF
- 最大DD
- トレード回数
- 平均損益
- Entry品質

特に、15分MTA Setup + 1分Triggerのうち、1時間足の大きな流れに逆らうEntryを除外できるかを検証します。

---

## 1H EMA Position Filter

### Long条件

text 確定済み1時間足終値 > 確定済み1時間足EMA 

### Short条件

text 確定済み1時間足終値 < 確定済み1時間足EMA 

1分足でTriggerが発生した時点で、上記条件を満たす場合のみEntryします。

---

## 確定済み1時間足を使う理由

1時間足EMA Filterでは、進行中の1時間足ではなく、確定済みの1時間足データを使います。

理由：

- 未確定の1時間足closeやEMAを使うと、リアルタイムと過去表示で挙動がズレる可能性がある
- ルックアヘッドバイアスを避ける
- バックテストと実運用の整合性を高める

Pine Script上では以下を基本にします。

pine h1CloseConfirmed = request.security(syminfo.tickerid, "60", close[1], gaps = barmerge.gaps_off, lookahead = barmerge.lookahead_off) h1EmaConfirmed = request.security(syminfo.tickerid, "60", ta.ema(close, h1EmaLength)[1], gaps = barmerge.gaps_off, lookahead = barmerge.lookahead_off) 

---

## 初期検証対象

### Step 1

text B5.2 Strategy + 1H EMA Position Filter 

### Step 2

EMA Lengthを変更して比較します。

text 20 / 50 / 100 / 200 

### Step 3

B5.2で改善が見られた場合、同じFilterをB1 / B2.1へ横展開します。

---

## 比較ベース

| Base Trigger | H1 EMA Filter | EMA Length | Trades | Win Rate | PF | Max DD | Avg P/L | Memo |
|---|---|---:|---:|---:|---:|---:|---:|---|
| B1 | none | none | 902 | 30.93% | 0.293 | 1.49% | -0.08 | Setup後最初Pivot固定 |
| B2.1 | none | none | 898 | 30.73% | 0.287 | 1.51% | -0.08 | 有利方向Update Pivot |
| B5.2 | none | none | 922 | 30.37% | 0.259 | 1.58% | -0.09 | Setup前直近Pivot |

---

## 検証テンプレート

| Base Trigger | H1 EMA Filter | EMA Length | Trades | Win Rate | PF | Max DD | Avg P/L | Memo |
|---|---|---:|---:|---:|---:|---:|---:|---|
| B5.2 | Position | 20 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B5.2 | Position | 50 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B5.2 | Position | 100 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B5.2 | Position | 200 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B1 | Position | 20 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B1 | Position | 50 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B1 | Position | 100 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B1 | Position | 200 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B2.1 | Position | 20 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B2.1 | Position | 50 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B2.1 | Position | 100 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B2.1 | Position | 200 |  |  |  |  |  | H1 confirmed close >/< EMA |

---

## Strategy基本設定

pine initial_capital = 1000000 default_qty_type = strategy.percent_of_equity default_qty_value = 2 commission_type = strategy.commission.percent commission_value = 0.04 slippage = 2 pyramiding = 0 process_orders_on_close = false calc_on_every_tick = false 

---

## Backtest方針

- Triggerは確定足で判定
- 注文は次足始値約定想定
- TP/SLはTrigger確定足の終値基準で計算
- RR初期値は1.0
- SLはMTA Zone反対側
- strategy.entry と strategy.exit は同じEntryブロック内で発注
- request.security は gaps_off / lookahead_off
- 1H EMA Filterは確定済み1時間足データのみを使う
- TradingView上で挙動確認する
- コンパイルだけでなくruntimeエラーも確認する

---

## 初期ロードマップ

### 1. B5.2 + 1H EMA Position Filter

B5.2 Strategyに1H EMA Position Filterを追加する。

確認内容：

- 元のB5.2 Trigger位置が維持されるか
- 1H EMA Filter OK/NGが正しく表示されるか
- Filter NGではEntryしないか
- Filter OKのみEntryするか

### 2. EMA Length比較

text 20 / 50 / 100 / 200 

### 3. B1 / B2.1へ横展開

B5.2で改善が見られた場合、同じFilterをB1 / B2.1へ追加する。

### 4. 3系統比較

B1 / B2.1 / B5.2のどれに1H EMA Filterが最も効くか比較する。

---

## 現時点の仮説

1H EMA Position Filterによって、上位足の大きな流れに逆らうEntryを減らせる可能性がある。

ただし、Filterが強すぎると取引回数が減りすぎる可能性がある。

見るべきポイント：

text PF改善 勝率改善 最大DD低下 取引回数の維持 平均損益の改善 
