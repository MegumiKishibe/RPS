# Changelog

## [2025-08-30] じゃんけんゲーム with チートモード v1.0
Since: 2025-08-26

### Step 1 — 基本実装
#### Added
- クリックイベントの実装（`.button-group` にデリゲーション）。  
  `e.target.closest('button')` で実ボタンを特定。
- 押下ボタンのテキストを結果エリア（`#result`）に表示。
- CPU のランダム手（`Math.floor(Math.random() * 3)`）を実装。
- 勝敗表示（あいこ／勝ち／負け）をベタ書き条件分岐で実装。
- 開発用ログ（`console.log`）でプレイヤー手・CPU手を確認。

#### Changed
- プレイヤー手：固定（グー=0）→ ボタンラベル判定で `plHand` を決定。  
  （`includes("グー"|"チョキ"|"パー")`）

#### Fixed
- 非ボタン領域クリック時の `btn === null` による  
  `Cannot read properties of null (reading 'textContent')` を防止。  
  `if (!btn) return;` をイベントハンドラー先頭に追加。

---

### Step 2 — チートモード
#### Added
- チェックボックス `#cheatMode` の取得と `change` イベントを実装。  
  `checked` に応じてログ出力（ON/OFF）。
- `cheatOn` フラグを導入（初期値 `false`）。  
  ON のときは CPU が必ず負ける手を選択：`cpuHand = (plHand + 1) % 3`。

#### Changed
- 「モード切替＝状態フラグに基づく条件分岐」という設計に整理。  
  クリック時の処理内で `cheatOn` を参照して CPU 手を決定。

#### Rationale
- `cheatOn` を未初期化にしない（`undefined` 回避）。  
  実行順序上の予期せぬバグを避けるため `let cheatOn = false;` を明示。

---

### Step 3 — UI 演出（アドバンスド）
#### Added
- 勝敗に応じて `#result` に `.win / .lose / .draw` を付与し色変更（ネオン表現）。  
  CSS 例：  
  ```css
  .win { color: #57aaff; }
  .lose { color: #ff5555; }
  .draw { color: #00ff99; }
  ```

- チートモード ON で背景アニメーション（`body.cheat-active .overlay` に `cheatGlow`）。  
  `document.body.classList.add/remove('cheat-active')` で切替。

#### Changed
- 色判定：`result.textContent.includes(...)` 依存 → `outcome` 数値（0/1/2）へ移行。  
  文言依存を解消し安定化。  

  ```js
  // outcome: 0=あいこ, 1=勝ち, 2=負け
  if (outcome === 0) textcolor = "draw";
  else if (outcome === 1) textcolor = "win";
  else textcolor = "lose";
  ```
- クラス更新は毎回クリアしてから付与：

  ```js
  result.classList.remove("draw","win","lose");
  result.classList.add(textcolor);
  ```
  #### Notes
- 背景アニメーション（チートモード時）：

  ```css
  /* body に .cheat-active が付くと .overlay にアニメ適用 */
  .cheat-active .overlay {
    animation: cheatGlow 1s infinite alternate;
  }

  @keyframes cheatGlow {
    0%   { background: #2e003e; }
    100% { background: #5e00a0; }
  }
  ```

#### Housekeeping
- **Removed**: デバッグ用 `console.log` の大半／検証用の旧コード（コメントアウト）を整理。  
  旧実装は Git 履歴で追跡する方針に変更。

- **Docs**: ブログ記事の実装ログをもとに CHANGELOG を整備。  
  `btn === null` によるエラー原因と対策も明文化。

## [2025-08-31] じゃんけんゲーム 関数化リファクタ v1.1
Since: 2025-08-26

### Refactor
- ロジックを小さな関数へ分割し責務を分離：
  - `getPlHandFromBtn(btn)`：ボタンテキスト → 0/1/2
  - `getCpuHand(plHand, cheatOn)`：チートONで必ず負ける手／OFFでランダム
  - `judgeOutcome(plHand, cpuHand)`：0=あいこ / 1=勝ち / 2=負け を返す
- 関数定義 → イベントハンドラーの順に**配置換え**（宣言が先、呼び出しが後）。
- 勝敗後に色更新を行うヘルパーへ処理を集約（`classList.remove → add`）。

### Added
- 表示用テンプレ配列を導入して文言を一本化：
  - `['グー','チョキ','パー']`、`['→あいこ！','→あなたの勝ち！','→あなたの負け！']`
- `cheat` 切替時のクラス操作に `classList.toggle('cheat-active', cheatOn)`（第2引数で冪等適用）。

### Changed
- クリック時の処理フローを **(1)手の取得 → (2)CPU決定 → (3)勝敗判定 → (4)結果表示 → (5)色更新** に整理。
- 文字色は `outcome` 数値ベースで決定（`includes` 依存を排除）。

### Fixed
- ドキュメント上の表記ゆれを修正：`judgeCpuHand` → **`judgeOutcome`**（関数名）。

### Notes
- 可読性重視で `getCpuHand` は初心者向けに if/else 版で記述：
  ```js
  function getCpuHand(plHand, cheatOn) {
    if (!cheatOn) {
      return Math.floor(Math.random() * 3); // 0,1,2 を等確率
    }
    return (plHand + 1) % 3; // プレイヤーに必ず負ける手
  }
  ```

// 結果テキスト
```js
const textHands   = ['グー','チョキ','パー'];
const textOutcome = ['→あいこ！','→あなたの勝ち！','→あなたの負け！'];

result.textContent =
  'あなた：' + textHands[plHand] + '　相手：' + textHands[cpuHand] + '　' + textOutcome[outcome];

// 文字色（outcome: 0/1/2 → draw/win/lose）
let textcolor;
if (outcome === 0)      textcolor = 'draw';
else if (outcome === 1) textcolor = 'win';
else                    textcolor = 'lose';

result.classList.remove('draw','win','lose');
result.classList.add(textcolor);
```