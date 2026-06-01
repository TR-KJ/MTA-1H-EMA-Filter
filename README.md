# MTA Trigger with 1H EMA Filter Lab

## 概要

このリポジトリは、15分MTAセットアップに対して、1分足TriggerでEntryする既存手法に、1時間足EMAフィルターを追加検証するための研究・開発ログです。

これまで、15分MTAセットアップに対して以下のTrigger系を検証してきました。

- B1系：Setup後、最初に確定したPivotをBreak基準にする
- B2系 / B2.1系：Setup後、有利方向に更新されるPivotをBreak基準にする
- B5.2系：Setup前の直近PivotをPre Break基準にする
- B9系：2点CTLを使ったTrigger

B9系では、2点CTLの機械化に取り組みましたが、2点CTLは裁量で見ているCTLとは別物になりやすく、EMA方向・EMA位置・EMA Break確認を加えても、B1/B2.1/B5.2を明確に上回る改善は確認できませんでした。

そのため、本リポジトリではB9系ではなく、比較的安定していたB1系・B2.1系・B5.2系に対して、1時間足EMAによる上位足環境フィルターを追加し、改善余地を検証します。

---

## 検証の目的

目的は、15分MTAセットアップ + 1分Triggerの既存ロジックに対して、1時間足EMAフィルターを追加することで、以下が改善するかを確認することです。

- 勝率
- PF
- 最大DD
- トレード回数
- 平均損益
- エントリーの質

特に、1時間足EMAフィルターによって、上位足の大きな流れに逆らうEntryを除外できるかを検証します。

---

## 対象Trigger

### B1系

Setup後、最初に確定したPivotをBreak基準にするTrigger。

text 15分MTA Setup ↓ Setup後、最初の1分Pivot H/Lを確定 ↓ Long：Pivot Hを実体上抜け Short：Pivot Lを実体下抜け ↓ Entry 

特徴：

- シンプル
- Trigger基準が固定される
- 比較対象として扱いやすい

---

### B2.1系

Setup後、有利方向に更新されるPivotだけをBreak基準にするTrigger。

text Long： Setup後、より高いPivot Hが出た場合のみBreak基準を更新  Short： Setup後、より低いPivot Lが出た場合のみBreak基準を更新 

特徴：

- B1より柔軟
- 不利方向のPivot更新を採用しない
- B1とB5.2の中間的な位置づけ

---

### B5.2系

Setup前の直近PivotをPre Break基準にするTrigger。

text 15分MTA Setup ↓ Setup前に存在していた直近Pivot H/LをBreak基準にする ↓ Long：Pre Break Hを実体上抜け Short：Pre Break Lを実体下抜け ↓ Entry 

特徴：

- 既存検証で主力候補として扱ってきた系統
- Setup前の構造を使う
- B1/B2.1と比較しやすい

---

## 追加する1時間足EMAフィルター

本リポジトリでは、まず1時間足EMAの「位置フィルター」を検証します。

### Position Filter

Long条件：

text 確定済み1時間足終値 > 確定済み1時間足EMA 

Short条件：

text 確定済み1時間足終値 < 確定済み1時間足EMA 

1分足でTriggerが発生した時に、上記条件を満たす場合のみEntryします。

---

## 重要な方針：確定済み1時間足だけを使う

1時間足EMAフィルターでは、進行中の1時間足ではなく、確定済みの1時間足データを使います。

理由：

- 未確定の1時間足closeやEMAを使うと、リアルタイムと過去表示で挙動がズレる可能性がある
- ルックアヘッドバイアスを避ける
- バックテストと実運用の整合性を高める

Pine Script上では、以下のような取得を基本とします。

pine h1CloseConfirmed = request.security(syminfo.tickerid, "60", close[1], gaps = barmerge.gaps_off, lookahead = barmerge.lookahead_off) h1EmaConfirmed = request.security(syminfo.tickerid, "60", ta.ema(close, h1EmaLength)[1], gaps = barmerge.gaps_off, lookahead = barmerge.lookahead_off) 

Entry条件への追加イメージ：

pine longH1Ok = not useH1EmaFilter or h1CloseConfirmed > h1EmaConfirmed shortH1Ok = not useH1EmaFilter or h1CloseConfirmed < h1EmaConfirmed 

---

## 初期検証パラメータ

まずは1H EMA Position Filterのみを検証します。

| Parameter | Values |
|---|---|
| H1 EMA Length | 20 / 50 / 100 / 200 |
| Filter Type | Position |
| Trigger Base | B1 / B2.1 / B5.2 |
| RR | 1.0 |
| SL | MTA Zone反対側 |
| Time Filter | All |
| Backtest Period | 2021/01/01〜2025/12/31 |

---

## 比較ベース

既存の比較値は以下。

| Base Trigger | H1 EMA Filter | EMA Length | Trades | Win Rate | PF | Max DD | Avg P/L | Memo |
|---|---|---:|---:|---:|---:|---:|---:|---|
| B1 | none | none | 902 | 30.93% | 0.293 | 1.49% | -0.08 | Setup後最初Pivot固定 |
| B2.1 | none | none | 898 | 30.73% | 0.287 | 1.51% | -0.08 | 有利方向Update Pivot |
| B5.2 | none | none | 922 | 30.37% | 0.259 | 1.58% | -0.09 | Setup前直近Pivot |

---

## 検証記録テンプレート

| Base Trigger | H1 EMA Filter | EMA Length | Trades | Win Rate | PF | Max DD | Avg P/L | Memo |
|---|---|---:|---:|---:|---:|---:|---:|---|
| B1 | none | none | 902 | 30.93% | 0.293 | 1.49% | -0.08 | Setup後最初Pivot固定 |
| B2.1 | none | none | 898 | 30.73% | 0.287 | 1.51% | -0.08 | 有利方向Update Pivot |
| B5.2 | none | none | 922 | 30.37% | 0.259 | 1.58% | -0.09 | Setup前直近Pivot |
| B1 | Position | 20 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B1 | Position | 50 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B1 | Position | 100 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B1 | Position | 200 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B2.1 | Position | 20 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B2.1 | Position | 50 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B2.1 | Position | 100 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B2.1 | Position | 200 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B5.2 | Position | 20 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B5.2 | Position | 50 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B5.2 | Position | 100 |  |  |  |  |  | H1 confirmed close >/< EMA |
| B5.2 | Position | 200 |  |  |  |  |  | H1 confirmed close >/< EMA |

---

## Strategy基本設定

Strategy化する場合は、以下を基本設定とします。

pine strategy(      "Strategy Name",      overlay = true,      initial_capital = 1000000,      default_qty_type = strategy.percent_of_equity,      default_qty_value = 2,      commission_type = strategy.commission.percent,      commission_value = 0.04,      slippage = 2,      pyramiding = 0,      process_orders_on_close = false,      calc_on_every_tick = false ) 

---

## Backtest思想

- Triggerは確定足で判定
- 注文は次足始値約定想定
- TP/SLはTrigger確定足の終値基準で計算
- RR初期値は1.0
- SLはまずMTA Zone反対側
- strategy.entry と strategy.exit は同じEntryブロック内で発注
- 未決済トレードが変に残らないようにする
- request.security() は必ず gaps_off / lookahead_off
- 1時間足フィルターは確定済み1時間足データを使う
- TradingView上での挙動確認を重視する
- コンパイルが通っても、実チャート上のruntime挙動を必ず確認する
- 長期検証ではTradingViewのバー数上限や表示制限に注意する

---

## Pine Script v6 コーディングルール

- Pine Script v6
- 1行1文
- セミコロンは使わない
- request.security() は gaps = barmerge.gaps_off / lookahead = barmerge.lookahead_off
- 未来参照・ルックアヘッドバイアスを避ける
- barstate.isconfirmed を重視する
- 配列やUDTを使う場合は必ず空チェック・サイズチェックを行う
- Pineの and / or の短絡評価に依存しない
- line.delete() / label.delete() 後は変数を na に戻す
- forループは範囲反転に注意し、必要ならガードする
- strategy化では、entryとexitを同じEntryブロック内で発注する

---

## 初期ロードマップ

### Step 1：B5.2 + 1H EMA Position Filter

まずは既存主力候補のB5.2に対して、1H EMA Position Filterを追加する。

確認項目：

- B5.2の元Trigger位置と一致するか
- 1H EMA Filter条件OK/NGが正しく判定されるか
- Entry対象がFilter OKのみに絞られるか
- 通常表示とリプレイで挙動が安定するか

### Step 2：B5.2 Strategy検証

EMA Length：

text 20 / 50 / 100 / 200 

を検証する。

### Step 3：B1 / B2.1へ横展開

B5.2で挙動が確認できたら、同じ1H EMA Position FilterをB1 / B2.1にも追加する。

### Step 4：3系統比較

B1 / B2.1 / B5.2のどれに1H EMA Filterが最も効くか比較する。

---

## 現時点の仮説

1時間足EMA Position Filterを追加することで、15分MTA Setup + 1分Triggerのうち、上位足の大きな流れに逆らうEntryを減らせる可能性がある。

ただし、フィルターが強すぎる場合、取引回数が減りすぎる可能性がある。

見るべきポイントは、単なるPF改善ではなく、以下のバランス。

text PF改善 勝率改善 最大DD低下 取引回数の維持 平均損益の改善 

特にB9.3では、PFは改善したが取引回数が大きく減ったため、今回の1H EMA Filterでは「取引数をある程度保ちながら改善できるか」を重視する。
