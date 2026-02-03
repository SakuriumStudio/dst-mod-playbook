# DST選択画面ポートレートのAtlas/Regionキー

**結論から言うと**、DSTのキャラ選択画面では、通常「bigportraits」フォルダ内のテクスチャが使われます。キャラ名のプレハブ（`prefab`）から最後に使われるキーは、スクリプト中で次のように組み立てられます。通常は*素のプレハブ名*＋`_none`（スキン未設定時）、そして楕円形なら`_none_oval`が付加されます。たとえばキャラ名が `monkey_king` なら、使われるAtlasファイルは`bigportraits/monkey_king_none.xml`、Region（画像）は`monkey_king_none.tex`（枠）および `monkey_king_none_oval.tex`（楕円）となります（下記ユーザースニペットも参照）【43†L154-L162】。スキンが選択されていれば、代わりにプレハブ名の後にスキンID（例: `_goose`）が付き、`bigportraits/<プレハブ>_<skin>.xml`・`<プレハブ>_<skin>.tex/_oval.tex`といった具合に上書きされます。  
```lua
-- 例: デフォルトの楕円ポートレート設定 (キャラが "monkey_king" の場合)
image_widget:SetTexture("bigportraits/monkey_king_none.xml", "monkey_king_none_oval.tex")
```  
この例はフォーラム投稿のコードから抜粋したもので【43†L154-L162】、DST本体でも同様の組み立てが行われています。なお、プレハブ名そのもの（`monkey_king`）は名前表示用（`images/names_<char>.xml`）や内部識別に使われますが、ポートレート用には必ず `*_none` の形式が付加されます。

## 「prefab」「prefab_none」「prefab_none_oval」「skin id」の使い分け

- **プレハブ名 (`prefab`)**  
  キャラの基本名です（例：`wilson`, `wormwood`など）。ポートレート用画像では直接使われず、内部的にAtlasや名前表示用リソース名に含まれることがあります。例えば名前ロゴ画像では `images/names_<キャラ名>.xml` が参照されます（エラー例【71†L128-L130】）。  
- **`prefab_none`**  
  スキンが「none」（デフォルト）の場合に使うキー文字列です。Atlas/PNG名に `_none` を付けることでデフォルトのキャラ画像を指定します。選択中のスキンが未指定のときは、キャラ名の後ろに `_none` を自動付加して画像を探します。  
- **`prefab_none_oval`**  
  楕円形（選択画面左上の小さい顔枠）用のRegion名です。上記のAtlasから楕円用の領域（`.tex`）を指定する際に `_none_oval` が付与されます。通常、Atlasは同一ファイルを使い、Region名だけ異なる例が多いです【43†L154-L162】。  
- **スキンID (`skin id`)**  
  キャラに装備・髪型・コスチュームなどのカスタムスキンが選択されている場合、そのIDがプレハブ名の後ろに付きます（例：`wormwood_owl`）。DSTのスクリプトは「`IsPrefabSkinned` が真」ならAtlas/Region名にこのIDを組み込みます。たとえばスキンID `owl` があると、`bigportraits/wormwood_owl.xml` や `wormwood_owl.tex`/`_oval.tex` が参照されます。  

これらのキーが使われる流れは、CharacterUtilやSkinnerの関数内で条件分岐されています。スキン未設定→`*_none`、設定済→`*_skinid`、楕円は常に `_oval` が付く、という具合です（参考コード例【43†L154-L162】）。なお、DST公式フォーラムでも「最新アップデートで楕円ポートレートが機能しなくなった」という投稿があり、ユーザーが上記のように明示指定することで修正していました【43†L154-L162】【74†L131-L139】。

## ポートレート画像フォルダの優先検索順

DSTはキャラクター関連の画像を複数フォルダに分けて持っており、用途に応じて検索順があります。主なフォルダと優先順位は以下のようになっています（ゲーム内ソースや公式情報を参考に整理）：

| 優先順位 | フォルダ名                   | 用途                           |
|:-------:|:---------------------------|:-----------------------------|
| 1       | `bigportraits`            | 選択画面およびゲーム内で使う大きなキャラ顔画像（楕円枠も同Atlas）。最優先で検索。 |
| 2       | `selectscreen_portraits`  | （DSTでは事実上未使用だが過去用に残存）選択画面用の代替画像。現在は無視してよい。 |
| 3       | `saveslot_portraits`      | セーブスロット画面（ワールド選択）のキャラ像。modで対応させる場合に利用。 |
| 4       | `avatars`                 | マップアイコンや一部UI用の小さなキャラ顔画像（“avatar_<name>.tex”）。CraftingMenu未設定時のフォールバックにも使用【68†L138-L144】。 |
| 5       | `crafting_menu_avatars`   | クラフト画面専用アイコン（“avatar_<name>.tex”）。優先順位はavatarsより上で、存在すればこちらを使う【68†L138-L144】。 |

補足すると、キャラ選択画面の大きいポートレートは**必ず**`bigportraits`を参照します（Atlas名にもbigportraitsパスが明示的）。そのため通常modでも`bigportraits/<プレハブ>_none.tex/.xml`を提供します。`selectscreen_portraits`は古いDST/単体版向けで、DSTでは無視するか代替手段とされています【65†L2490-L2493】【65†L2542-L2544】。逆に「avator」や「crafting_menu_avatars」は選択画面ではなく、マップやクラフトフィルター画面のUI用です。アップデート公式ノートにて、クラフト画面アイコンは `crafting_menu_avatars/avatar_<name>` → なければ `avatars/avatar_<name>` と順に参照する旨が明記されています【68†L138-L144】。

## エラー例：「Atlas/Regionが見つからない」ログ

画像が見つからない場合、ログに**WARNING**で類似のメッセージが出力されます。たとえば公式フォーラムの例では、無効なキーを参照したときに次のような警告が出ています：  

```
WARNING! Could not find region 'FROMNUM' from atlas 'FROMNUM'. Is the region specified in the atlas?
Looking for default texture '' from atlas 'FROMNUM'.【71†L128-L130】
```  

ここでは `atlas='FROMNUM'`、`region='FROMNUM'`（見つからない）が同時に警告され、デフォルトテクスチャが空文字列で検索されている様子が分かります【71†L128-L130】。また別の事例では、`avatars.xml`の中に`avatar_cave_entrance.tex`が見つからないと警告され、代わりに`avatar_unknown.tex`を探すログも確認できます（例:Steamフォーラムのログ）【85†L1-L4】。こうしたメッセージが出た場合は、AtlasやPNGファイルのパス／名前に誤りがある可能性が高いので要チェックです。

## まとめ

上記の内容をまとめると、キャラ選択ポートレートでは以下のようなキーが使われます。  

- **`<prefab>_none`**：スキン未選択時のデフォルトAtlas名（例: `bigportraits/wilson_none.xml`）。  
- **`<prefab>_none_oval`**：楕円枠用Region名（例: `wilson_none_oval.tex`）。  
- **`<prefab>_<skinID>`**/**`_<skinID>_oval`**：スキン選択時のAtlas/Region（例: `wormwood_owl.tex` / `wormwood_owl_oval.tex`）。  

これらは内部で`SetTexture`呼び出しに組み込まれ、優先度順にフォルダから画像を探します。万一見つからないと「Could not find region/atlas…」の警告ログが出力されます【71†L128-L130】【85†L1-L4】ので、キャラmod開発時は画像リソースの追加・名前付けに注意しましょう。  

**参考資料:** DST公式フォーラムやスクリプト例からの抜粋【43†L154-L162】【68†L138-L144】【65†L2542-L2544】【71†L128-L130】および開発者コメントなど。表やコード例で理解を補強してください。各フォルダ・キーの優先度表は上記の通りです。  

