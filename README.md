# じゃんけんゲーム with チートモード

ブラウザで動くじゃんけんゲーム。チェックボックスで **チートモード** を切替できます。

## 使い方
1. `index.html` をブラウザで開く
2. 「グー / チョキ / パー」をクリック
3. 必要なら「チートモードをONにする」にチェック

## 特徴
- ボタンクリックでプレイヤー手を取得
- チートON時：CPUは必ず負ける手を出す
- 勝敗に応じて結果テキストと色（`draw/win/lose`）を切替
- 背景アニメーション（`body.cheat-active .overlay`）

## 技術メモ
- ランダム手：`Math.floor(Math.random() * 3)`
- 勝敗判定：`0=あいこ, 1=勝ち, 2=負け`
- 関数分割：
  - `getPlHandFromBtn(btn)`：ボタン→手(0/1/2)
  - `getCpuHand(plHand, cheatOn)`：CPU手（チート時は (plHand + 1) % 3）
  - `judgeOutcome(plHand, cpuHand)`：勝敗を 0/1/2 で返す

## 開発
- Node 不要 / ビルド不要（静的HTMLのみ）
- 主要ファイル：`index.html`
