# DST Mod Tools autocompiler：XML `<Element name="...">` 命名規則まとめ（一次情報統合版）

このドキュメントは、**DST Mod Tools（autocompiler）**が生成する **atlas XML の `<Element name="...">`** が **どの規則で決まるか**、そして **bigportraits / names / selectscreen_portraits** など “要求側の期待名が厳しい” 場所での事故パターンと回避策を、**一次情報（Klei Forums のガイド／テンプレ指示、既知のDSTの仕組み）**を根拠に「実戦向け」に統合したものです。

---

## 0. 先に結論（最重要）

- **autocompiler が生成する `<Element name="...">` は、基本的に `PNGファイル名（拡張子なし） + ".tex"`** で決まる。  
  例：`myitem.png` → `<Element name="myitem.tex" ... />`

- ただし、**“ゲーム側が要求する名前” が autocompiler の命名とズレるフォルダがある**。  
  代表が **`images/names` / `images/names_gold` / `bigportraits`**。  
  これらは **生成後に XML の name を手で直す（または最初からそれに合わせて運用する）**のが定番。

---

## 1. `<Element name>` の決定規則（autocompiler側）

### 1.1 基本ルール
- 入力：PNG（例：`myitem.png`）
- 出力：`.tex` と `.xml`（atlas）
- XMLの中身は概ねこうなる：

```xml
<Atlas>
  <Texture filename="myitem.tex" />
  <Elements>
    <Element name="myitem.tex" u1="0" v1="0" u2="1" v2="1" />
  </Elements>
</Atlas>
```

つまり、**`<Element name>` は「元PNGのベース名に `.tex` を付けた文字列」**。

### 1.2 何が “無関係” か
- **xml ファイル名**（例：`inventoryimages.xml`）は `<Element name>` に影響しない  
- **.tex の実ファイル名**（autocompilerが吐くやつ）は `<Element name>` とセットだが、名前自体の決定因子は元PNGのベース名  
- **Asset("ATLAS", "...") の順番**は `<Element name>` には影響しない（参照可否に影響するだけ）

---

## 2. 事故が起きる理由（ゲーム側＝要求側）

DST側はおおむねこう動きます：

1) autocompiler  
- `png → tex`  
- `png名 → <Element name="png名.tex">`

2) ゲーム側 Lua  
- `atlas.xml` をロード  
- **コードやUI実装が「この name を要求する」と決め打ち**している部分がある

ここで **「生成された `<Element name>`」と「ゲームが要求する name」**が1文字でもズレると、  
**ロード自体は成功しても UI には出ない**（か、シルエット等にフォールバック）という地獄になる。

---

## 3. フォルダ別：要求側の期待名が厳しいところ（典型事故＆対策）

以下は「autocompilerの生成」と「ゲームの期待」がズレやすい所だけを抜粋。

### 3.1 `images/names`（キャラ名の画像）
**よくある状況**  
PNGを `names_<prefab>.png` のように置くと、autocompilerはそのまま  
`<Element name="names_<prefab>.tex">` を生成する。

**でもゲーム側（UI）は** `"<prefab>.tex"` を見に行く（＝`names_` を付けない）運用が一般的。  
→ **なので XML の `<Element name>` を `<prefab>.tex` に修正する必要が出る**。

**典型事故**
- `names_wilson.png` を置いた
- 生成XMLが `name="names_wilson.tex"`
- 期待は `wilson.tex`
- → UIで名前が出ない

**回避策**
- 生成後に `names_` プレフィックスを外して **`<Element name="wilson.tex">` に手修正**する（定番）

---

### 3.2 `images/names_gold`（金色名画像）
`names_gold_<prefab>.png` でも同じ。  
生成：`name="names_gold_<prefab>.tex"`  
期待：`<prefab>.tex`  
→ **`names_gold_` を外して手修正**が定番。

---

### 3.3 `bigportraits`（キャラ選択の大きいポートレート）
ここは「名前ズレ」より **接尾辞が罠**で、ガイド系でも “autocompilerのせい” と言われがち。

代表パターン：`<prefab>_none.png` を作る  
生成：`<Element name="<prefab>_none.tex">`

しかし、UIが探すのが `"<prefab>_none_oval.tex"` であるケースがあり、  
その場合は **XMLを `..._oval.tex` に変える**必要がある。

**典型事故**
- `watson_none.png` を置いた
- 生成XML：`name="watson_none.tex"`
- 期待：`watson_none_oval.tex`
- → big portrait が出ない

**回避策**
- 生成後にXMLで `watson_none.tex` → `watson_none_oval.tex` に修正（定番）

---

### 3.4 `images/selectscreen_portraits`（キャラ選択の小さめポートレート）
ここは “命名” より “変換設定” で事故が多い話として知られています。

**典型事故**
- 変換時の設定（Mipmaps など）で黒くなる／表示崩れ
- ※命名以前に「見える画像」になってない

**回避策（知られている定番）**
- TexTool等で変換する場合、設定に注意（Mipmapsの扱いなど）

---

## 4. 実戦チェックリスト（事故を潰す）

### A. まず「生成された name」を確認する（最強）
- [ ] 該当フォルダの `.xml` を開いて `<Element name="...">` を直接見る  
- [ ] ゲーム側が要求する「期待名」と **1文字ずつ**突き合わせる（大小文字も含む）

### B. 命名（ベース名）チェック
- [ ] **拡張子なしのベース名**が期待通りか
- [ ] **大小文字が一致**しているか（`TeddyHat` ≠ `teddyhat` ≠ `TEDDYHAT`）
- [ ] 余計な接尾辞／プレフィックスを付けてないか（`names_`, `names_gold_`, `_oval` 等）

### C. フォルダ別 “補正が必要” チェック
- [ ] `images/names`：`names_` を外して `<prefab>.tex` にする必要があるか
- [ ] `images/names_gold`：`names_gold_` を外して `<prefab>.tex` にする必要があるか
- [ ] `bigportraits`：`_none_oval.tex` を要求されていないか（`_none.tex` のままだと出ないケース）

### D. 文字種・パスの地雷
- [ ] **半角英数と `_`** で統一（日本語・空白・特殊記号は避ける）

---

## 5. 補足：前回テキストの「png名だけ見る」について（統合＆訂正）
前回の説明で「png名だけ見る」という言い方をした部分は、意図としては  
**“フォルダ名や xml名ではなく、元PNGのベース名が本質”**という意味でした。

ただし、厳密には autocompiler は XML に **`.tex` を付けた name** を出すのが基本です。  
（例：`wilson_bigportrait.png` → `wilson_bigportrait.tex`）

なので、この統合版では一次情報に合わせて、  
**`<Element name> = PNGベース名 + ".tex"`** を正式ルールとして扱っています。

---

## 6. 参考（一次情報リンク：そのまま貼ると長いのでコードブロックに入れる）
```text
Klei Forums: Image files for DST?（names / autocompiler の説明・テンプレ運用）
https://forums.kleientertainment.com/forums/topic/118754-image-files-for-dst/

Klei Forums: The Big Book of DST Character Creation（bigportraits の _oval 罠など）
https://forums.kleientertainment.com/forums/topic/129004-guide-the-big-book-of-dst-character-creation/

Klei Forums: Custom Item（大小文字一致の注意喚起）
https://forums.kleientertainment.com/forums/topic/50003-custom-item/
```

---

## 7. 使い方（おすすめ運用）
- まずは **png を置く → autocompiler で生成 → xml を開いて `<Element name>` を確認**  
- 期待名とズレてたら、**names / names_gold / bigportraits だけ最小限の手修正**  
- あとは **prefab名・skin id・フォルダ優先順位**の話（skinner絡み）に進むと “透明・無表示” 問題の地図が完成します
