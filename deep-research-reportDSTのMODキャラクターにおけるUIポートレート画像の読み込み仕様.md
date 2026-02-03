# DSTのMODキャラクターにおけるUIポートレート画像の読み込み仕様

## 研究方法と一次情報の範囲

本レポートは、entity["company","Klei Entertainment","game developer"]の公式フォーラム／公式バグトラッカーに投稿された仕様説明・開発者コメント・代表的なテンプレ運用ガイド、そしてそれらに含まれる**ゲーム同梱Luaスクリプトのファイルパス／関数名／挙動説明**を一次情報として突き合わせ、DSTのMODキャラクターがUIで参照するポートレート系画像（`bigportraits` / `images/selectscreen_portraits` / `images/saveslot_portraits` / `images/avatars` / `self_inspect`相当）について「どのUIが、どの名前で、どの形式を読むか」をフォルダ単位に体系化したものです。citeturn10view0turn28view0turn31view0turn33view0turn29view0  

なお、DSTのLuaスクリプト本体は、ローカルのゲームインストール内（例：`.../Don't Starve Together/data/databundles/scripts.zip`）に格納されていることが公式フォーラムでも明示されており、フォーラム投稿内のスタックトレースに登場する`.../scripts/...`の参照先は、この同梱スクリプト群と整合します。citeturn25search17turn31view0turn32view0  

## 画像アセットの基本仕様

### pngが「直接参照」されない理由（tex/xmlが前提になる根拠）

DSTのUI（Widget）は、ログ上も`Image:SetTexture(atlas, tex)`の形で「**atlas（.xml）＋texture（.tex）**」を前提に描画しており、atlas内に指定した“region名（例：`avatar_ika.tex`）”が無いと警告・エラーになります。実例として、`images/avatars.xml`に`avatar_ika.tex`が見つからないケースでは、`scripts/widgets/image.lua`の`SetTexture`から`playerbadge.lua`→`playerstatusscreen.lua`→`playerhud.lua`へとスタックが伸び、最終的にエラー画面に至っています。citeturn31view0  

また、公式サンプルMOD配布（Klei公式として“Sample Character”を含むサンプル群）でも、テンプレ画像は`.png`で提供される一方、「`.tex`を作るにはTEXツールが必要」と明記されています。これは**ゲームが最終的に読むのが`.tex`である**ことの一次根拠になります。citeturn10view0  

### tex/xml生成が「必須」か（開発時と配布時の整理）

代表的なテンプレ運用（Extended Sample Character系）では、作業手順として「`.png`を編集→コンパイルで`.tex`/`.xml`が生成される」「生成済み`.tex`/`.xml`は一旦削除しても再生成される」と説明されます。つまり**開発フロー上はpngから自動生成できる**が、**参照の最終形はtex/xml**で、この前提が外れるとロード失敗（`Could not find an asset matching ... .tex`等）になります。citeturn9view0turn32view0  

（結論）  
- **UI参照の“入口”は常に`.tex`＋`.xml`**（png直読みの設計ではない）citeturn10view0turn31view0  
- pngは「コンパイラ（オートコンパイル／TEXツール）でtex化してから」実体として利用されるciteturn9view0turn10view0  

### atlas(.xml)の“region名”がファイル名と一致しないケースがある

`.xml`側のregion名（`name="..."`）が、単純に「その.texファイル名」と一致していないと、ログに「Could not find region 'X' from atlas 'Y'. Is the region specified in the atlas?」が出て、代替テクスチャ探索（`Looking for default texture '' ...`）に進みます。citeturn30search9turn31view0  

Extended Sample Character運用では、まさにこの“region名ズレ”を手修正する具体例として、`bigportraits/<prefab>_none.xml`の`Element name`を`<prefab>_none_oval.tex`へ直す手順が提示されています（textureファイル名自体は変えない）。citeturn9view0  

## フォルダ別の読み込み仕様

ここでは各フォルダについて、(1)ファイル名規則 (2)tex/xml要否 (3)参照元Lua・テーブル・関数 (4)失敗時フォールバック (5)ログ例、の順で整理します。

image_group{"layout":"carousel","aspect_ratio":"16:9","query":["Don't Starve Together character select screen","Don't Starve Together player status screen tab menu","Don't Starve Together crafting menu character filter avatars"],"num_per_query":1}

### images/avatars

#### ファイル名規則

公式開発者コメントにより、MODキャラのavatarはデフォルトで次を期待します。citeturn31view0  

- 生存者（通常）  
  - `images/avatars/avatar_<prefabname>.xml`  
  - `images/avatars/avatar_<prefabname>.tex`  
- ゴースト  
  - `images/avatars/avatar_ghost_<prefabname>.xml`  
  - `images/avatars/avatar_ghost_<prefabname>.tex`

ここでのキーは**prefab名**で、skin idではありません（少なくとも“デフォルト期待値”はprefab名）。citeturn31view0  

さらに、状態差分の早期サポートとして、`chr_state_1`タグ＋`USERFLAGS.CHARACTER_STATE_1`を立てた場合に、`avatar_<prefabname>_1.xml/.tex`を使う、というsuffix規則も明示されています。citeturn31view0  

#### tex/xml生成は必須か（png不可の根拠）

`SetTexture(atlas, tex)`で参照されるのは`.xml`と`.tex`であり、`.png`がそのまま指定される前提ではありません。`.tex`が無い／`.xml`にregionが無いと、`Could not find region ... from atlas ...`→`Image:SetTexture`起点のエラーになります。citeturn31view0turn10view0  

#### 参照元となるLua・テーブル・関数

一次情報として、少なくとも以下が確認できます。  

- `scripts/widgets/image.lua` の `SetTexture` が実描画の入口（ログに明示）citeturn31view0  
- Tabのプレイヤーステータス画面では `scripts/widgets/playerbadge.lua` がavatar参照に関与し、そこから `scripts/screens/playerstatusscreen.lua` → `scripts/screens/playerhud.lua`へ到達する（スタックトレースに明示）citeturn31view0  
- MOD側でatlasの置き場所を変える場合は、グローバルテーブル `GLOBAL.MOD_AVATAR_LOCATIONS["prefabname"] = "images/..."` を設定する、という拡張点が開発者コメントとして提示されています。citeturn31view0  

> 重要：この`MOD_AVATAR_LOCATIONS`は「ディレクトリ変更」のための公式に言及された仕組みで、**“images/avatars固定”でない運用**の根拠になります。citeturn31view0  

#### 読み込み失敗時のフォールバック挙動

- atlasにregionが無い場合、まず警告が出て（`Could not find region...`）、その後“default texture”を探しに行くログが出ます。UI上は、状況により**クラッシュ**（少なくとも当該ログ例ではエラー画面）に至り得ます。citeturn31view0turn30search9  
- 2014当時は`playerbadge.lua`側のatlas指定が硬く、MODキャラがTabメニューで落ちる事象があり、ホットフィックスで「prefabに`inst.avatar_tex`/`inst.avatar_atlas`を持たせる」方向へ進んだ、という経緯も記録されています（つまり“参照先をインスタンス側で差し替える”のが公式推奨ルートになった）。citeturn31view0  

#### client_logの典型メッセージ例

- `WARNING! Could not find region 'avatar_<name>.tex' from atlas 'images/avatars.xml'. Is the region specified in the atlas?`citeturn31view0  
- `scripts/widgets/image.lua(...) in function 'SetTexture'`（以降、playerbadge等へスタック）citeturn31view0  

### images/avatars の self_inspect 系（質問文の「self_inspectなど」）

#### ファイル名規則

コミュニティの運用・テンプレMODの資産列挙では、self inspectは次の命名が一般形として扱われています。citeturn12view0turn27search3  

- `images/avatars/self_inspect_<prefab>.xml`  
- `images/avatars/self_inspect_<prefab>.tex`

#### tex/xml生成は必須か、失敗時の見え方

self_inspectが壊れると「黒い四角になる」という症状報告があり、再コンパイルで直らない場合もあるが、最終的に“TEX Creatorで手作成した.texに差し替えたら直った”という事例があります。これは、**見た目が出ない原因がpngではなく.tex生成品質にある**ケースが現実にある、という一次情報です。citeturn12view0  

（つまり対策としては「ファイルパス／命名」だけでなく「tex生成条件」を疑う価値がある。）citeturn12view0turn13search1  

### images/crafting_menu_avatars（※指定外だが、近年の“キャラ頭アイコン”で重要）

2022年の公式バグトラッカー対応で、開発者が**配置ルールとフォールバック**を明文化しています。citeturn28view0  

- クラフトメニュー（フィルター等）用のMODキャラアイコン：  
  - `/images/crafting_menu_avatars/avatar_<name>.xml` と `avatar_<name>.tex` を置くciteturn28view0  
- もしクラフトメニュー専用アイコンが無い場合：  
  - `/images/avatars/avatar_<name>.xml` と `avatar_<name>.tex` にフォールバックciteturn28view0  

ここでの命名は**`avatar_<name>`（prefix固定）**で、これが`images/avatars`の“Tab／バッジ用途”と同じ資産を再利用できる設計になっています。citeturn28view0turn31view0  

### images/saveslot_portraits

#### ファイル名規則

フォーラムの技術的分析では、サーバー一覧／作成（resume含む）画面側が、キャラ名をもとに`<character>.tex`（region名）を組み立てて`SetTexture(atlas, character..".tex")`していることが示されています。citeturn33view0  

また、実運用としては次の配置が想定される（“saveslot_portraitsフォルダに`<characterName>.xml`と`.tex`”）という提案がされています。citeturn33view0  

- `images/saveslot_portraits/<prefab>.xml`  
- `images/saveslot_portraits/<prefab>.tex`

#### tex/xml生成は必須か

ここもウィジェットが`SetTexture(.xml, .tex)`または`SetTexture(atlas, tex)`で扱うため、最終的に必要なのは`.xml`と`.tex`です。citeturn33view0turn31view0  

#### 参照元Lua・テーブル・関数

サーバー一覧画面側について、一次情報として次が挙がっています。citeturn33view0  

- `scripts/screens/serverlistingscreen.lua`（例：行番号1045付近）  
- `scripts/screens/redux/serverlistingscreen.lua`（例：行番号1109付近）  
- 参照判定に `DST_CHARACTERLIST` / `MODCHARACTERLIST` を使う  
- `MODCHARACTERLIST` は `AddModCharacter` が投入し、`AddModCharacter` の定義は `modutil.lua` にある、という“追跡”が提示されているciteturn33view0  

つまり、**画像参照以前の段階で「そのキャラがMODキャラとしてリスト化されているか」**が経路分岐に効きます。citeturn33view0  

#### 読み込み失敗時のフォールバック挙動

serverlistingscreen系の分岐例では、条件によってキャラ名を`mod_small`または`unknown`に差し替える（=汎用アイコンへ退避）処理が示されています。citeturn33view0  

また、2025年時点の別スレでは、saveslot_portraits自体は「特定のMODを入れている場合に使われる／それ以外だと使われない」とも言及されており、**UI側が常用していない（=実装・時期・画面によっては参照されない）**可能性があります。citeturn29view0  

### images/selectscreen_portraits

#### ファイル名規則（legacy含む）

テンプレ・既存MODでは一般に以下が並びます。  
- `images/selectscreen_portraits/<prefab>.xml/.tex`  
- `images/selectscreen_portraits/<prefab>_silho.xml/.tex`（silhouette、suffix=`_silho`）citeturn32view0turn31view0  

#### 参照状況（“今”使われているか）

2025年のフォーラム回答では、`selectscreen_portraits`は「もう使われていない（古いバージョンやソロDONT STARVEで使われた可能性がある）」とされ、レガシー目的で残っている、と言及されています。citeturn29view0  

ただし、**MOD側の`Assets`にこのパスを列挙しているのに、実ファイル（.tex）が無い**場合、ロード時点で即クラッシュ級のエラーになります（`resolvefilepath`が`assert`で落ちる）。この意味で「現行UIで使われない」ことと「MODのAssetsから外して良い」ことは別で、テンプレを流用するなら整合性が必要です。citeturn32view0  

#### tex/xml生成は必須か（png不可の根拠）

不足すると `Could not find an asset matching images/selectscreen_portraits/<name>_silho.tex in any of the search paths` が出て、`scripts/util.lua`の`resolvefilepath`から`RegisterPrefabs`等を経由してロード失敗します。ここで探しているのが`.tex`であること自体が、png直読みでない根拠になります。citeturn32view0  

### bigportraits

#### ファイル名規則

テンプレ運用・一般的な資産列挙は次を前提にしています。citeturn9view0turn27search3  

- `bigportraits/<prefab>.xml`  
- `bigportraits/<prefab>.tex`

加えてDST特有要素として、DSへ移植する際に「不要」とされる資産リストに`Bigportrait/character_none`が挙がっており、DST側では`<prefab>_none`（ロック時・未解放時・シルエット用途など）の系列が想定されていることが読み取れます。citeturn34view0  

そしてExtended Sample Character系では、`bigportraits/<prefab>_none.xml`の**Element名を`<prefab>_none_oval.tex`にする必要がある**、という具体的な調整が提示されています（textureファイル名は変えない）。これはbigportraitsが「oval」前提で参照される経路が存在することを示します。citeturn9view0  

#### tex/xml生成は必須か（png不可の根拠）

公式サンプルMODやテンプレ運用で、bigportrait等を含むUI画像について`.tex`生成が前提とされ、`.tex`が無いと「assetが見つからない」エラーで止まる事例が報告されています。citeturn10view0turn30search0turn32view0  

#### ロード失敗の代表パターン（寸法制約）

bigportrait系は、圧縮テクスチャとして特定条件を満たさないとエラーになります。実例として、`bigportraits/<name>.tex is 520x650 but compressed textures must have power of 2 dimensions`（2の冪サイズ制約）というログが出ています。citeturn30search15turn13search1  

（解釈）bigportraitは見た目が大きいぶん、作り手が元画像サイズを自由にしがちですが、**tex化後の寸法が制約を満たさないとUIどころかロード全体を壊し得る**領域です。citeturn30search15  

## 失敗・フォールバック挙動と典型ログメッセージ

### UI画像ロード失敗は大きく2系統ある

1) **ファイル（.tex自体）が見つからない**  
- `resolvefilepath`が`assert`で落ち、ロード中断（MODのAssets列挙と実ファイルの不一致で起きやすい）。citeturn32view0  

2) **atlas（.xml）はあるが、欲しいregion名が無い**  
- `WARNING! Could not find region ... from atlas ...` が出る。  
- 続いて `Looking for default texture '' ...` → `Error Looking for default texture ...` といった“代替探し”ログが出ることがある。citeturn31view0turn30search9  

### 代表的ログメッセージ一覧

| 分類 | 典型メッセージ（要旨） | 起きがちな原因 | まずやる対処 |
|---|---|---|---|
| ファイル未発見 | `Could not find an asset matching .../<name>.tex in any of the search paths` | `.tex`未生成／フォルダ違い／Assets列挙だけ残して実体が無い | `.tex/.xml`の生成確認、パス一致、Assets列挙の整合 citeturn32view0turn30search0 |
| atlas内region不在 | `WARNING! Could not find region 'avatar_<name>.tex' from atlas 'images/avatars.xml'` | `.xml`のElement名が違う／別名でtexを生成した／avatar名のprefixや大文字小文字違い | `.xml`のregion名を期待値に合わせる（例：avatar_…）citeturn31view0turn9view0 |
| default探しで失敗 | `Looking for default texture '' ...` → `Error Looking for default texture ...` | regionが無く代替も無い（空） | region名を正す、atlas参照を正すciteturn30search9 |
| UI由来スタック | `scripts/widgets/image.lua ... SetTexture` → `playerbadge.lua` 等 | UI用画像の欠落が致命になる画面（Tab、バッジ等） | まずavatars系を疑う、パス規約に合わせるciteturn31view0 |
| 画像仕様違反 | `...bigportraits/<name>.tex is ... but compressed textures must have power of 2 dimensions` | tex化後の寸法が2の冪でない（またはtex生成要件違反） | 元画像サイズ・tex生成条件を修正、再生成citeturn30search15turn13search1 |

## 「キャラ選択の顔が出ない」原因トップ5と対処

ここでいう「顔が出ない」は、主に **(A)キャラ選択での頭アイコン**／**(B)プロフィール・ロビーの大きい肖像**／**(C)クラフトUI・Tab等のバッジ**のいずれかが“空・黒・汎用”になる症状を想定します。DSTはUI改修で参照先が増減するので、まず「どの画面で出ないか」を確定させるのが最短です。citeturn28view0turn29view0turn31view0  

### 原因

**原因1：`images/avatars`（avatar_ / avatar_ghost_）の命名・配置が規約からズレている**  
- 典型：`avatar_<prefab>.xml/.tex`ではなく別名、または大文字小文字が混在。  
- 症状：Tabメニューやプレイヤーバッジで落ちる／表示されない。  
- 対処：開発者コメントにあるデフォルト期待値どおりに、`images/avatars/avatar_<prefabname>.xml`＋`avatar_<prefabname>.tex`（ghostも同様）へ揃える。ディレクトリを変えるなら`GLOBAL.MOD_AVATAR_LOCATIONS`で明示する。citeturn31view0  

**原因2：クラフトメニューの“キャラ頭アイコン”が`images/crafting_menu_avatars`に無い**  
- 近年の変更で、クラフトメニューのフィルター等は専用パスを参照し、無ければ`images/avatars`へフォールバックする仕様が明記されています。  
- 症状：クラフトメニューのキャラアイコンだけ欠ける（それ以外は正常）。  
- 対処：`/images/crafting_menu_avatars/avatar_<name>.xml`＋`avatar_<name>.tex`を追加する。専用が不要なら`images/avatars`側に最低限avatarを置く。citeturn28view0turn31view0  

**原因3：`.tex/.xml`をAssetsに列挙しているのに、実体が存在しない（未コンパイル／消し忘れ）**  
- 症状：起動時・ロード時に`Could not find an asset matching ... .tex`で落ちる／キャラが選択画面に出ない。  
- 対処：`.png`編集後に必ずコンパイルし、生成された`.tex/.xml`が正しいパスにあるか確認。テンプレ流用で不要になった画像は、Assetsからも削除して“列挙と実体”を一致させる。citeturn9view0turn32view0turn10view0  

**原因4：atlas(.xml)のregion名（Element name）が期待値と違う**  
- 症状：ファイルはあるのに`Could not find region 'X' from atlas 'Y'`が出て、空表示・汎用・黒四角になる。  
- 対処：`images/...xml`の`Element name`が、UIが要求する名前（例：`avatar_<prefab>.tex`、bigportraitなら`<prefab>_none_oval.tex`等）と一致しているか確認し、必要なら修正する（Extended Sample Character運用で実際にこの修正が必須例として提示されている）。citeturn9view0turn30search9turn31view0  

**原因5：tex生成条件違反（サイズ・mipmaps等）でtex自体が不正**  
- 症状：黒四角・微妙なクラッシュ・特定の画面だけ落ちる等、“パスは合ってるのに挙動が変”。  
- 対処：bigportrait等で「2の冪サイズ」要件に引っかかっていないかログ確認。self_inspectが黒四角なら、再コンパイルに加えてTexCreator等で.texを作り直す（実例あり）。citeturn30search15turn12view0turn13search1  

## フォルダ仕様の比較要約

| フォルダ | 主な用途（一次情報ベース） | 命名キー | 代表suffix | 参照側の根拠（Lua/画面） | 失敗時の代表挙動 |
|---|---|---|---|---|---|
| `images/avatars` | Tabのプレイヤーバッジ／オフスクリーン指標等（開発者コメント＋スタック） | prefab名 | `_ghost_`、状態`_1`等 | `playerbadge.lua`、`Image:SetTexture`、`MOD_AVATAR_LOCATIONS` | region不在で警告→場合によりエラー画面citeturn31view0 |
| `images/avatars`（`self_inspect_`） | “Inspect Self icon”が黒四角になる報告（MOD Skins関連） | prefab名 | `self_inspect_`prefix | 実運用報告（ファイル名想定） | 黒四角・tex作り直しで改善例citeturn12view0 |
| `images/crafting_menu_avatars` | クラフトメニューのキャラフィルター用（開発者コメント） | “name” | `avatar_`prefix固定 | バグトラッカーの開発者回答 | 無ければ`images/avatars`へフォールバックciteturn28view0 |
| `images/saveslot_portraits` | サーバー一覧／作成・resume系でMODキャラの保存枠ポートレート（議論＋コード断片） | キャラ名（MODCHARACTERLIST判定） | なし | `serverlistingscreen.lua` / `redux/...`、`MODCHARACTERLIST` | `mod_small`/`unknown`へ退避分岐例citeturn33view0turn29view0 |
| `images/selectscreen_portraits` | レガシー（2025時点では未使用言及あり） | prefab名 | `_silho` | `.tex`欠落で`resolvefilepath`が落ちる例 | Assets列挙と矛盾するとロード失敗citeturn29view0turn32view0 |
| `bigportraits` | DST特有の大きい肖像＋`character_none`系列（DST→DS移植ガイド、テンプレ修正） | prefab名 | `_none`、`_none_oval`（region名） | テンプレ運用でxml修正指示 | サイズ要件違反でエラー、asset未発見でロード失敗citeturn34view0turn9view0turn30search15turn30search0 |

以上を踏まえると、DSTのUIポートレートは「**prefab名を中心に、画面ごとに“期待するprefix/suffixとフォルダが違う”**」のが本質で、さらにアップデートで参照先（例：crafting_menu_avatars）が追加・変更されるため、**“どの画面用のアイコンか”から逆算して置く場所を決める**のが最短ルートです。citeturn28view0turn31view0turn29view0turn9view0