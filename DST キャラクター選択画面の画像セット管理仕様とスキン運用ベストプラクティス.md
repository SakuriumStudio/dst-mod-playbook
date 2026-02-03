# DST キャラクター選択画面の画像セット管理仕様とスキン運用ベストプラクティス

## 背景と前提スコープ

本レポートは、Don’t Starve Together（以下DST）の「キャラクター選択画面（character selection screen）」で主に絡む “人物系UI画像” のうち、モッダーが編集しがちな `images/avatars`・`images/saveslot_portraits`・`images/selectscreen_portraits`・`bigportraits` を中心に、役割（必須/任意）、命名規約、参照経路（どこから読まれるか）、欠落時のフォールバック（黙って代替される挙動）、そしてスキン適用に伴う運用の落とし穴を整理する。根拠は、entity["organization","Klei Entertainment","game studio, vancouver, ca"]公式フォーラムのModdingガイド／チュートリアル、およびDST Luaスクリプト由来のAPIドキュメント（ビルド番号を明示しているもの）に置く。citeturn22view0turn35view0turn21view0

この領域は「ゲーム内スキン（キャラ用コスメ）」と「キャラ選択UI（ポートレート/アバター）」が合流する地点で、同じ“見た目”でも **(A) UI用の画像（`.xml`+`.tex`）** と **(B) プレイヤー実体のアニメ（bank/build）** が別系統で管理されることが多い。混ぜて考えると、欠落ログが出ない／出る、見た目だけ変わる／実体が変わらない、などの事故が起きる。citeturn22view0turn35view0turn37search2

## 画像フォルダの役割と命名規約

ここでは「キャラクター選択画面に関連する画像」を、DSTが提供する `CharacterUtil` とスキン定義側（CreatePrefabSkin等）で確認できる “実際に参照されるファイルパス” を軸に整理する。citeturn22view0turn35view0

### フォルダ別の仕様まとめ

| フォルダ | 目的（何の画像か） | 必須/任意（キャラ選択に対して） | 基本命名・期待される中身 | 主な参照元/関数（確認できた範囲） | 欠落時の挙動（代表例） |
|---|---|---|---|---|---|
| `images/avatars` | “アバター”系の小画像（UIの顔アイコン用途） | **実質必須**（少なくともアバター画像が必要なUI群に影響） | 例：`avatar_wilson.tex` が使われる（atlas+textureの組） | `GetCharacterAvatarTextureLocation(character)` が atlas/texture を返す（例：`"images/avatars.xml", "avatar_wilson.tex"`）citeturn22view0 | 未登録/不明キャラは `"avatar_unknown.tex"` 等へフォールバックする、と説明されているciteturn22view0 |
| `bigportraits` | キャラの “楕円ポートレート（oval portrait）” | **キャラ選択の大ポートレート表示に直結**（ただしフォールバックあり） | `bigportraits/[portrait_name].xml` から読む、と明記（= atlas）citeturn22view0。スキン用では “skin id + `_oval` をxml内nameに設定” が必要、という手順が提示されているciteturn35view0 | `SetSkinnedOvalPortraitTexture(image_widget, character, skin)` が bigportraits を読む／楕円が無い場合は別ポートレートへフォールバックすると説明citeturn22view0 | “楕円が無ければ盾型（shield portrait）へフォールバック” と明記citeturn22view0 |
| `images/saveslot_portraits` | “セーブスロット用ポートレート” という名前の通りの用途が想定される | **DST現行のCharacterUtil API上は参照が確認できないため、キャラ選択に対しては任意扱い**（要検証） | 伝統的テンプレに残ることが多いが、`CharacterUtil` の説明範囲には出現しないciteturn22view0 | 本レポートがアクセスできた一次資料（CharacterUtil/skinsutils/公式フォーラムチュートリアル）では参照箇所を確定できないciteturn22view0turn35view0 | “使われないならエラーも出ない” 可能性が高い（後述の「黙って無視される」パターンに近い）citeturn19search2turn22view0 |
| `images/selectscreen_portraits` | “選択画面用ポートレート” という名前の通りの用途が想定される | **少なくとも現行CharacterUtilでは bigportraits を利用する設計が明示**されているため、キャラ選択の主経路では任意扱い（要検証）citeturn22view0 | テンプレ由来で残りがちだが、現行での必須性は一次資料上確定できないciteturn22view0turn35view0 | 同上 | 同上 |

「saveslot_portraits / selectscreen_portraits は本当に不要なのか？」は、**“現行のキャラ選択（楕円）” が `bigportraits` を読む**という仕様が明確な一方で、当該2フォルダを直接参照する一次資料（閲覧できた範囲）が足りない、というのが正直な結論になる。だからベストプラクティスは **「現行UIの参照経路（avatars + bigportraits）をまず100%正しくする」→「必要が出た時にだけ追加で入れる」** の順が事故りにくい。citeturn22view0turn35view0turn19search2

image_group{"layout":"carousel","aspect_ratio":"16:9","query":["Don't Starve Together character select screen","Don't Starve Together big portrait oval","Don't Starve Together mod character select portrait","Don't Starve Together avatars icon UI"],"num_per_query":1}

### ビルド（スキン）とポートレートの命名がぶつかるポイント

スキン（見た目差分）を導入すると、「キャラの識別子（character/prefab）」と「スキンID（skin id）」が分離する。このとき **ポートレート名が “キャラ名” ではなく “スキンID” 基準になる**ケースが公式フォーラムのチュートリアルで明示されている（“通常キャラのポートレートとほぼ同じだが、キャラ名の代わりに skin id を使う”）。citeturn35view0turn22view0

さらに、楕円ポートレートの `.xml` 内では `name` が **`skin_id .. "_oval"`** になっている必要がある、と具体例と共に説明されている（ファイル名はそのままでも、xml内部のnameが重要）。citeturn35view0turn22view0

## 欠落画像がエラーにならないケースの実態

DSTのこの手のUI画像は、「チェックしてから差し替える」「不明ならunknownへ」「見つからないなら別ルートへ」みたいな **“失敗前提のフォールバック設計”** が多く、結果として「画像を置き忘れたのにログが出ない」ことが起こり得る。citeturn22view0turn19search2

### 代表パターン

**パターンA：softresolvefilepathで“存在確認→なければ別パスへ”**  
クラフトUIのキャラアバター解決ロジックの例では、まず “Mod用のcrafting avatar atlas” を組み立て、`softresolvefilepath(atlas_name) == nil` なら別の場所（一般アバター）へ回す、という分岐が明示されている。ここで重要なのは「`softresolvefilepath` が nil を返した時点で、**例外・クラッシュではなく分岐でリカバリする設計**」になっている点。つまり「ファイルが無い＝即エラー」ではなく、**“無いなら別のを使う”**が先に走る。citeturn19search2

**パターンB：CharacterUtilが unknown/mod に丸める**  
`GetCharacterAvatarTextureLocation(character)` は、バニラキャラだけでなくmodキャラや未知キャラにも対応し、未知の場合は `"images/avatars.xml", "avatar_unknown.tex"` のようにフォールバックする、と例付きで説明されている。つまり “そのキャラ専用のアバターが無い” という状況が、**unknownアイコンに吸収されてログが汚れにくい**。citeturn22view0

**パターンC：楕円ポートレートが無ければ盾型へ**  
`SetSkinnedOvalPortraitTexture` は “楕円があれば楕円を読み、無ければshield portraitへフォールバック” と説明されている。これも「欠落＝即死」ではなく「欠落＝別見た目で続行」になりやすい。citeturn22view0

### 逆に“ログに出やすい欠落”の条件

DSTの基本的なPrefab作成例では、`AnimState:SetBank()` と `AnimState:SetBuild()` を明示的に呼ぶ（＝bank/buildが揃っていることが前提）コードが提示されている。ここを壊すと、UI画像とは違って **ゲーム内実体のアニメ系**は破綻しやすく、警告・エラーになりやすい領域だと読み取れる（少なくとも “bank/buildをセットするのが規範” として書かれている）。citeturn37search2turn37search3

## スキン適用のランタイム挙動

この章は、質問のうち「player_common と skinner の相互作用」「skinschanged」「SetSkinMode/SetSkinName」「build requirement」までを **“一次資料で確定できる部分” と “合理的推定になる部分” に分けて**整理する。理由は単純で、今回アクセスできた一次資料には `components/skinner.lua` の本文（関数本体・イベント発火箇所）が含まれていないため、発火タイミングを断言できない領域が残るから。citeturn30view0turn22view0turn35view0

### 確定できる事実：スキンは「mode」と「build override」を持つ

公式フォーラムのスキン作成チュートリアルでは、`CreatePrefabSkin` の引数として少なくとも次が登場する。

- `skins = { normal_skin = "...", ghost_skin = "..." }` のような **“skin mode表”**  
- `build_name_override = "..."` のような **“どのbuildでスキンするか”**（キャラの場合も同様に例示）citeturn35view0

さらに、DSTのスキンユーティリティ（skinsutils）の説明では、キャラクターには `GetSkinModes(character)` という「そのキャラが持つskin mode一覧（normal/ghost/変身など）」を返す関数がある、と説明されている。ここから “modeを切り替える” という概念自体が、公式スクリプトのユーティリティレイヤで前提になっていることが分かる。citeturn18search0turn35view0

また、Mod Supportの例では `AddModCharacter("mycharacter", ..., { { type = "normal_skin" ... }, { type = "ghost_skin" ... } })` のように、modキャラ登録時点で skin mode（normal_skin/ghost_skin）を宣言する例が提示されている。これは “player_common側の共通処理が、skin modeの概念を前提にキャラを扱う” 方向性を補強する材料になる。citeturn28search8turn30view0

### 推定できる実行順：player_commonで土台→skin modeで上書き→UI更新

DeepWikiの解説では、`MakePlayerCharacter`（player_common）がプレイヤー生成の共通基盤であり、コンポーネント初期化やイベント処理、ネット同期を担うとされる。つまり「最初に“素のプレイヤー”が出来る」のは player_common 側の責務として妥当。citeturn30view0turn29view0

そのうえで、スキン定義は `CreatePrefabSkin` によって **base_prefab（元のprefab）+ build_name_override（差し替えbuild）+ skins（mode別build）** を提供する作りになっているため、ランタイムでは概ね以下のような上書きが起きるのが自然だ（これは “設計からの推定” であり、関数名・発火タイミングを断言しない）。citeturn35view0turn18search0turn30view0

```
(推定フロー)
プレイヤー生成（player_common / MakePlayerCharacter）
  ↓
プレイヤーの見た目初期値（bank/build等）が設定される
  ↓
skin mode（normal_skin / ghost_skin / その他）と skin id から
「どのbuildを使うか」が決まる（build_name_override等）
  ↓
選ばれたbuildで見た目が上書きされる
  ↓
UI側は CharacterUtil で
  - アバター（images/avatars）
  - 楕円ポートレート（bigportraits）
を引き直す（必要ならフォールバック）
```
（この段落のうち “CharacterUtil が avatars と bigportraits を読む” は確定、 “player_common→上書き” は推定）citeturn22view0turn35view0turn30view0

### skinschanged イベントの扱い

質問の “skinschanged はいつ発火するか” は、**イベント名まで含めて一次資料で確定できなかった**。今回参照できた公式フォーラムのチュートリアルは “スキン定義とポートレート命名” に強く、`skinner` の内部イベント発火（event push）までは触れていない。citeturn35view0turn22view0

ただし、DSTのシステムはイベント駆動でコンポーネントを疎結合にしている、とDeepWikiのアーキテクチャ概要が説明しているため、「スキン変更が“何らかのイベント”として通知され、それを受けてUI/見た目が更新される」という設計自体は合理的。citeturn29view0turn30view0

よって、実装上のベストプラクティスとしては **“スキン変更通知（一般に skinschanged と呼ばれることが多い）を受けたら、UIのポートレート再解決と、必要な見た目再適用を行う”** を推奨する。ただし、イベント名・いつ飛ぶか（SetSkinName直後なのか、mode切替後なのか、ネット同期後なのか）は、`components/skinner.lua` の一次ソース確認なしに断言しない。citeturn22view0turn35view0turn29view0

## 安定運用のための最小パターン集

この章は “限界まで短く・事故りにくい” ことを優先して、(A) 画像セット、(B) スキン定義、(C) build requirement回避をまとめる。citeturn22view0turn35view0turn37search2

### 画像セットを壊さないための最小チェックリスト

**アバター（avatars）**  
`GetCharacterAvatarTextureLocation` は “atlas と texture名（例：avatar_wilson.tex）” を返す設計が示されているため、最低限「atlas（.xml）とtexture（.tex）がペアで揃う」ことが重要になる。citeturn22view0turn19search2

**楕円ポートレート（bigportraits）**  
`SetSkinnedOvalPortraitTexture` は `bigportraits/[portrait_name].xml` を読むことが明記され、スキン用ポートレートは「skin id を使う」「xml内nameは skin id + `_oval`」が手順として提示されている。つまり “ファイルを置く” だけでなく **xml内のElement名まで整合**させる必要がある。citeturn22view0turn35view0

### CreatePrefabSkin を軸にした “いちばん壊れにくい” スキン実装

公式フォーラムのチュートリアルが提示する `CreatePrefabSkin` パターンは、DSTのスキン設計（mode/override）と一致しており、**“SetSkinMode/SetSkinNameを手で叩いて事故る” より先に採用すべき基準実装**として妥当。citeturn35view0turn18search0turn28search8

以下は、チュートリアル内容の要点（modeとbuild overrideの関係）を “最小形” に寄せた例。コードは例示であり、文字列やアセットはあなたのmodの実ファイルに合わせる必要がある。citeturn35view0

```lua
-- 例: prefabs/mychar_skins.lua のイメージ
local prefabs = {}

table.insert(prefabs, CreatePrefabSkin("mychar_none", {
    assets = {
        Asset("ANIM", "anim/mychar.zip"),
        Asset("ANIM", "anim/ghost_mychar.zip"),
    },
    skins = {
        normal_skin = "mychar",
        ghost_skin  = "ghost_mychar",
    },
    base_prefab = "mychar",
    build_name_override = "mychar",
    type = "base",
    rarity = "Character",
    skin_tags = { "BASE", "MYCHAR" },
}))

return unpack(prefabs)
```

この方式の利点は、(1) mode別のbuildを **定義で固定**でき、(2) ポートレート命名も skin id を基準に整理でき、(3) 公式チュートリアルに沿うため第三者が読みやすい点にある。citeturn35view0turn22view0

### “prefab name build requirement” を最短で潰す考え方

この問題は、ログ文言の完全一致まで一次資料で確定できていないが、DSTの基本Prefab例が示す通り、**AnimStateを持つ实体は bank/build をセットするのが規範**になっている。ここが揃っていないと、見た目やアニメの前提が崩れ、警告やエラーに繋がりやすい。citeturn37search2turn37search3

最短で効く対策は、次のどれか（または複合）。

- **bank/buildを、存在するアセット名に合わせて必ず設定する**（例：`inst.AnimState:SetBank("mychar")` と `SetBuild("mychar")`）citeturn37search2turn37search3  
- スキンを使うなら、`CreatePrefabSkin` の `build_name_override` が示す build の `.zip`（ANIM資産）を確実に同梱し、normal/ghostの両modeで破綻しないようにする（チュートリアルは “ghostも指定する” 前提で例示）。citeturn35view0  
- 変身や特殊形態があるキャラの場合、skinsutilsが想定するように “skin modeが複数あり得る” ので、モードごとの見た目（build/bank）を欠落させない（GetSkinModesが “normal/ghost/変身” を扱うと説明している）。citeturn18search0turn35view0  

### 方法・ツール・実装方針の比較表

最後に、DSTで「キャラ選択UIとスキン」を扱う現実的な選択肢を比較する（“何を優先するか” で最適解が変わる）。citeturn22view0turn35view0turn18search1turn28search8

| 方針 | 何を実装するか | 強み | 弱み | 向いているケース |
|---|---|---|---|---|
| キャラ追加のみ（実質 `_none` 相当だけ） | 新キャラprefab + 最低限のavatar/bigportrait | 実装が軽い。運用が単純 | スキン差分の拡張が後から面倒 | まずキャラを動かしたい／UI画像だけ整えたい citeturn37search2turn22view0 |
| `CreatePrefabSkin` でスキンを定義 | skin id・mode（normal/ghost）・build override を定義し、ポートレートも skin id 基準で作る | 公式フォーラムのチュートリアル手順に沿い、設計が読みやすい | アセット（ANIM/portrait）数が増えると管理が大変 | スキン追加が主目的。長期運用・共同開発 citeturn35view0turn22view0turn18search0 |
| `PREFAB_SKINS`（公式のスキンDB）と整合する発想で設計 | “base_prefab→skin一覧” という関係性を意識して管理 | 公式のスキン体系に近い整理ができる | 公式DB自体は自動生成で直接編集対象ではない | 既存スキン体系に寄せた設計をしたい citeturn18search1turn37search4 |
| UIは CharacterUtil に寄せて徹底的にフォールバック前提で作る | avatars/bigportraits を CharacterUtil のAPIに合わせて揃える | 欠落があっても “unknown/盾型” で落ちにくい | 逆に欠落に気づきにくい（ログが出にくい） | クラッシュよりUXを優先／リリース後の安定性重視 citeturn22view0turn19search2 |

---

### 付録：ワークフロー図（画像ロードとフォールバック）

この図は `CharacterUtil` と、`softresolvefilepath` を用いた “存在確認→フォールバック” の例から、キャラ選択周辺の画像解決を **実務でデバッグしやすい形**に落としたもの。citeturn22view0turn19search2

```
[キャラ選択UIで必要になる画像]
  ├─ アバター（小）
  │    └─ CharacterUtil:GetCharacterAvatarTextureLocation(character)
  │          ├─ (バニラ) images/avatars.xml + avatar_<character>.tex
  │          └─ (不明/未登録) images/avatars.xml + avatar_unknown.tex 等へ
  │
  └─ 楕円ポートレート（大）
       └─ CharacterUtil:SetSkinnedOvalPortraitTexture(widget, character, skin)
             ├─ bigportraits/<portrait_name>.xml を試す
             └─ 無ければ shield portrait へフォールバック
```

（この図の各矢印は、該当関数の説明・例に基づく）citeturn22view0turn19search2turn35view0