# DST Autocompiler における PNG → TEX 命名規則と例外整理

本ドキュメントは、Don't Starve / Don't Starve Together の  
**autocompiler（Mod Tools）** が PNG を TEX に変換する際の  
**命名規則・例外・再現条件・回避策**を、  
テンプレート流用（esctemplate 等）を前提に **実務向け**に整理したもの。

---

## 1. 基本規則（原則）

autocompiler は以下の思想で動作する。

- atlas / xml / build 内に存在する  
  `<Element name="foo.tex">`
- に対応する PNG（通常は `foo.png`）を探し
- それを **foo.tex** としてパックする

つまり、

> **Element name = 最終的に生成される .tex 名**

PNG のファイル名は本来ただの素材名であり、  
**最終決定権は Element name にある**。

通常は以下の対応になる。

```
foo.png → foo.tex
<Element name="foo.tex">
```

ここまでは一般に知られている基本挙動。

---

## 2. 例外が起きる理由（核心）

例外の正体はほぼこれに尽きる。

> **「Element name が autocompiler によって上書きされなかったケース」**

autocompiler は万能ではなく、  
**既存定義が存在する場合は「人間が決めた」と判断して介入しない**。

以下の条件が揃うと、Element name の更新がスキップされる。

---

## 3. 例外①：esctemplate / 既存 template 由来の Element が残る

### 再現条件

- esctemplate（Extended Sample Character Template）  
  または他 mod の template をコピー
- xml / scml / build.xml 内に  
  既存の `<Element name="xxx.tex">` が残っている
- PNG 名だけを変更した

### autocompiler の内部判断（挙動）

```
「Element はすでに定義済みだな。
  じゃあこの PNG は、その Element 用の差し替え素材だ」
```

### 結果

- PNG 名は無視される
- **古い Element name が最終 tex 名になる**

```
newicon.png → oldicon.tex
```

### esctemplate 実例

- `names_MyChar.png` を用意
- Spriter / xml 内の Element が `esctemplate.tex` のまま
- 出力される XML 例：

```xml
<Texture filename="names_MyChar.tex"/>
<Element name="esctemplate.tex"/>
```

Texture と Element の名前が食い違う、典型的な事故。

---

## 4. 例外②：同名 PNG が複数ディレクトリに存在

### 再現条件

- template 由来の PNG が削除されていない
- 新旧 PNG がどちらも atlas 対象に含まれている
- Element name は 1 つだけ存在

### 挙動

autocompiler は

- 最初に見つけた PNG
- または template 側の相対パス

を掴んだ時点で  
**対応関係を確定し、再評価を行わない**。

### 結果

- tex 名は Element name 依存
- PNG 名は完全に無関係になる

---

## 5. 例外③：build.xml / atlas.xml を手動編集している

### 再現条件

- build.xml / atlas.xml を直接編集
- autocompiler 実行時に  
  「既存ファイルを尊重する」分岐に入る

### 挙動

```
「これは人間が決めた名前だな。触らないでおこう」
```

### 結果

- PNG 名を変更しても tex 名は変わらない
- 手動編集が常に優先される

---

## 6. 補足例外：buildanimation.py の既知バグ

Klei 公式フォーラムで報告されている既知問題。

### 内容

- `buildanimation.py` 内で
- SCML のレイヤー名取得に誤った属性を参照していた時期がある

本来参照すべき：

```
layer.name
```

誤って参照されていた例：

```
layer.layername
```

### 影響

- `timeline_xxx` などの意味不明な symbol / Element 名が生成される
- PNG 名・Element name と一致しない tex が出力される
- テンプレ問題と誤認しやすい

### 対策

- 修正済みの Mod Tools を使用
- 公式フォーラムで案内されている修正版 `buildanimation.py` を使用

---

## 7. まとめ：名前が残るケースの再現条件

以下のいずれかを満たすと **名前残留が発生する**。

- 既存 Element name が xml / build に存在
- template 由来の定義が削除されていない
- autocompiler が「上書き不要」と判断した
- buildanimation.py の既知バグを踏んでいる

要点はこれ。

> **PNG 名が主導権を握れるのは  
> 「Element が未定義のときだけ」**

---

## 8. 実務での回避策（妥協なし）

### 回避策①：template 流用時は「名前リセット」を最初に行う

- xml / scml / atlas / build をすべて確認
- `<Element name="...">` を一度すべて削除
- PNG 名を最終 tex 名に揃える
- その状態で autocompiler を実行

最も確実な方法。

---

### 回避策②：Element name を明示的に正解へ書き換える

- Element name を PNG 名と一致させる
- autocompiler に名前決定を期待しない

テンプレ流用では最短ルート。

---

### 回避策③：template 素材を atlas 対象から完全に除外

- 使わない PNG を物理的に削除
- build 対象ディレクトリに残さない

**見えなければ掴まれない**。

---

### 回避策④：最終確認は tex 名を見る

- atlas.xml の `<Element name="xxx.tex">`
- build 出力された tex ファイル名

ここが一致していなければ、まだ事故る。

---

## 9. 結論

autocompiler は

> **新規作成には優しいが、流用には容赦がない**

テンプレートは便利だが、

> **名前は信用するな。消してから作れ。**

この原則を守れば、  
PNG 名と TEX 名の不一致問題は確実に回避できる。

この地雷を一度理解した人間は、  
二度と同じ事故を起こさない。
