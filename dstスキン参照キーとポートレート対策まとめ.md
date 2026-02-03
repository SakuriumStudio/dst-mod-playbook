# DST：スキン有効時の参照キー切替・必要ファイル・確実にポートレートを出す方針

> 目的：DSTのMODキャラで skinner（SetSkinMode/SetSkinName 等）＋スキンシステムを使うとき、**UI（ポートレート/アバター/名前画像/アイコン）参照が prefab 名から skin key（例：`mychar_none` / `mychar_<skinID>`）へ切り替わる条件**と、必要ファイル・対処方針を“実装に落ちる形”で固定する。

---

## 1) 参照キーが prefab → skin key に切り替わる条件（実務で効くやつ）

### A. “スキンシステムに乗ったキャラ”と判定される条件
- `MakeModCharacterSkinnable("<prefab>")` を modmain.lua で実行している（AddModCharacter の後）
- その上で `AddModCharacterSkin(...)` などによりスキン情報が登録される

この状態になると、キャラは「スキン対応キャラ」として扱われ、**UIや各種参照が `"<prefab>"` ではなく `"<prefab>_none"` を起点に探されやすくなる**。

### B. “いま選ばれている見た目”が skin key になる条件
- `skinner` コンポーネントが存在する
- `skinner.skin_name` が `nil` ではなく、`"<prefab>_none"` または `"<prefab>_<skinID>"` になっている
  - ※最近のAPI/一部ケースではスポーン直後に `skin_name` が `nil` になって事故る事があるため、**スポーン時に `SetSkinName("<prefab>_none")` を明示する対策が推奨**

### C. どの場面で “prefab名” が残りうるか
- スキン非対応キャラ（Aを満たしていない）
- スキン対応でも `skin_name` が `nil` / `"none"` 扱いになっている（＝事故）

> 実務的には「**スキン対応にした瞬間、デフォルトでも `<prefab>_none` がキーになる**」と思って設計するのが安全。

---

## 2) 必要ファイル：どのフォルダに何を置くべきか

DSTのUI画像は基本「`*.xml`（atlas）＋ `*.tex`」のペア。

### 最優先（キャラ選択画面の大判ポートレート）
- `bigportraits/<prefab>_none.xml` + `bigportraits/<prefab>_none.tex`
- atlas 内で参照される楕円用テクスチャ：`<prefab>_none_oval.tex`

スキン別に出すなら（強い・公式準拠）：
- `bigportraits/<prefab>_<skinID>.xml` + `bigportraits/<prefab>_<skinID>.tex`
- atlas 内で：`<prefab>_<skinID>_oval.tex`

### HUDのアバター・自己確認（APIの avatarset オプション）
- `images/avatar_<name>.xml` + `images/avatar_<name>.tex`
- `images/self_inspect_<name>.xml` + `images/self_inspect_<name>.tex`

ここで `<name>` は、
- 通常：`<prefab>`（もしくはエンジンが期待する標準名）
- スキンで差し替える：`avatarset` を使って **`<avatarset>`** に置換される

### 選択画面の名前プレート（APIの uiname オプション）
- `images/names_<name>.xml` + `images/names_<name>.tex`
- `uiname` を使うと **`names_<uiname>`** が使われる

### インベントリアイコン（スキンで ChangeImageName する場合）
- `images/inventoryimages/<atlas>.xml` + `images/inventoryimages/<atlas>.tex`
- `inventoryitem:ChangeImageName(inst:GetSkinName())` をやるなら、
  - 参照キーが `"<prefab>_none"` / `"<prefab>_<skinID>"` になる
  - それに対応する icon/atlas の登録が必要

---

## 3) 既存の対処パターン（MODがやってる“勝ち筋”）

### パターンA：公式準拠（推奨・一番堅い）
- **UI画像もスキンごとに用意**する
- つまり `bigportraits/<prefab>_<skinID>.*` を全部作る
- `modmain.lua` の Assets に atlas/tex を追加し、確実にロードさせる

メリット：
- UIがスキンに完全追従し、事故が激減
- 将来の仕様変化にも強い

デメリット：
- 画像ファイル数が増える

### パターンB：フォールバック固定（軽量・現実的）
- スキンはアニメ/見た目だけ切替
- **ポートレートは常に同じ画像を使う**（`<prefab>_none` のみ用意）
- ただし参照キーは `"<prefab>_<skinID>"` で来る可能性があるため、
  - 「skin key でも同じatlasを返す」ように登録する

メリット：
- 画像が1枚で済む
- 実装が軽い

デメリット：
- スキン別のポートレート差分は出せない

---

## 4) 「スキンを使いつつポートレートを確実に出す」実装方針

ここは **“二段ロケット”**。

### 4-1. 最低限の事故防止（必須）
1) `bigportraits/<prefab>_none.xml/.tex`（＋ `_none_oval.tex`）を必ず用意
2) スポーン時に `skin_name` が `nil` にならないよう、
   - `AddPlayerPostInit` などで `inst.components.skinner:SetSkinName("<prefab>_none")` を実行

### 4-2. ポートレートの作り分け方針（A or B）

#### 方針A（公式準拠：スキン別ポートレート）
- `bigportraits/<prefab>_<skinID>.xml/.tex`（＋`_<skinID>_oval.tex`）をスキンごとに用意
- `modmain.lua` の `Assets` にそれらを列挙してロード漏れを防ぐ

#### 方針B（軽量：ポートレートは固定）
- `bigportraits/<prefab>_none` だけ用意
- 参照が `<prefab>_<skinID>` で来ても表示できるように、
  - スキン側のUI参照を同じ画像に向ける（登録で吸収）

---

## 5) modmain.lua での扱い（実務テンプレ）

### A) スキン対応宣言
- `AddModCharacter("<prefab>", ...)` の後に
- `MakeModCharacterSkinnable("<prefab>")`

### B) “デフォルトスキン名がnil事故”を潰す（超重要）
- `AddPlayerPostInit(function(inst) ... end)` 等で
  - `if inst.components.skinner and inst.components.skinner:GetSkinName() == nil then ... end`
  - `inst.components.skinner:SetSkinName("<prefab>_none", true)`

### C) Assets（ロード漏れ防止）
- 画像は「参照される可能性があるもの」を Assets に入れる
- 方針Aならスキン別bigportraitも全部
- 方針Bでも最低 `bigportraits/<prefab>_none` は入れる

---

## 6) まずこれだけチェックすれば原因特定できる（デバッグ観点）

1) 実行時に `print(inst.components.skinner and inst.components.skinner:GetSkinName())`
   - `nil` なら “nil事故”
   - `"<prefab>_none"` なら正常
   - `"<prefab>_<skinID>"` ならスキン参照になってる

2) `bigportraits/` に `<prefab>_none.xml` が存在するか

3) `<prefab>_none.xml` の中で参照している tex 名が本当に存在するか（特に `_oval`）

---

## 7) 推奨の結論（兄弟向けにズバッと）

- **スキン対応にする＝キーは `prefab` じゃなく `prefab_none / prefab_<skinID>` が主戦場**
- まず `bigportraits/<prefab>_none` を絶対に揃える（これ無しだと透明になりやすい）
- スポーン時に `SetSkinName("<prefab>_none")` を入れて `nil` 事故を潰す
- その上で、
  - こだわるなら「方針A：スキン別bigportrait」
  - まず安定優先なら「方針B：ポートレート固定＋登録で吸収」

---

## 付記：このまとめの出どころ（一次情報の方向性）
- Klei公式フォーラムの Modded Skins API スレッド群
- 実際に bigportraits の `*_none.xml` と `*_none_oval.tex` を参照している報告
- スポーン直後に skin_name が nil になり得るため `SetSkinName("*_none")` を推奨する投稿

（※キャンバスは“読む用”に寄せているので、引用形式のURL/引用番号は省略してある。必要なら本文側で引用付き版も出せる）

