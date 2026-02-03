# DSTキャラ選択画面におけるSetTexture呼び出し箇所  
DST（Don’t Starve Together）のキャラクター選択UIでは、ビッグポートレート用のテクスチャ設定に`SetTexture(atlas, region)`が使われています。フォーラムのエラーログによれば、`scripts/widgets/image.lua`の`SetTexture`呼び出しは最終的に`scripts/screens/lobbyscreen.lua`内の`SetPortraitImage`経由で行われています【83†L135-L143】。具体的には、**DressupPanel**や**AnimSpinner**といったウィジェットがスキン変更イベントを受け取り、`onChanged`→`SetPortraitImage`というチェーンで`SetTexture`が実行されます【83†L135-L143】。つまり、キャラ/スキン選択UIでユーザ操作があると、順に`lobbyscreen.lua:onChanged`→`SetPortraitImage`が呼ばれ、ここで`SetTexture`によって大きいポートレート画像が切り替わる仕組みです【83†L135-L143】。

## atlas/region文字列生成ロジック  
テクスチャ設定時の`atlas`（アトラスファイル）と`region`（画像領域）文字列は、キャラクターのプレハブ名とスキン名に基づき生成されます。たとえばフォーラム例では、モンキーキングの場合に以下のように呼ばれています：  

```lua
image_widget:SetTexture("bigportraits/monkey_king_none.xml", "monkey_king_none_oval.tex")
```  

ここで **atlas** は `"bigportraits/<プレハブ名>_<スキン名>.xml"`、**region** は `"<プレハブ名>_<スキン名>_oval.tex"` の形です【18†L158-L166】。上記ではプレハブ名が `monkey_king`、スキン名（デフォルト）が`none`なので、`bigportraits/monkey_king_none.xml` と `monkey_king_none_oval.tex` となります【18†L158-L166】。同様に、例えばキャラ「jock」の場合もアトラスXMLに `<Element name="jock_none_oval.tex" …>` が定義されています【31†L150-L154】。これらの例から、*Atlasファイルは「bigportraits/」直下にあり、*.xmlのファイル名は`<prefab>_<skin>.xml`、領域名（region）は`.tex`付きで`<prefab>_<skin>_oval.tex`*というパターンだとわかります【18†L158-L166】【31†L150-L154】。

| キャラ名（プレハブ） | atlas文字列例                   | region文字列例               |
|-----------------|----------------------------|------------------------|
| monkey_king（デフォルト） | `bigportraits/monkey_king_none.xml` | `monkey_king_none_oval.tex` 【18†L158-L166】 |
| jock（デフォルト）       | `bigportraits/jock_none.xml`       | `jock_none_oval.tex`       【31†L150-L154】 |
| （一般例）            | `bigportraits/<prefab>_<skin>.xml`   | `<prefab>_<skin>_oval.tex` |

（例: *monkey_king* や *jock* のケースを上記の引用から抽出【18†L158-L166】【31†L150-L154】）

## `prefab_none`・`prefab_none_oval`・`skinid`組み立てロジック  
DSTではキャラクターのスキン管理に新仕様があり、デフォルトスキンではプレハブ名に`_none`を付けます。たとえばチュートリアル例では`CreatePrefabSkin("jangton_none", { … build_name_override = "jangton", … })`とあり、キャラ名`jangton`に`_none`を付けた文字列がプレハブ名になっています【50†L200-L207】。これにより**prefab_none**（プレハブ）の命名規則が「キャラ名＋`_none`」となり、選択画面でもこの名前に対応したテクスチャが参照されます。さらに、ゲーム全体では`GLOBAL.PREFAB_SKINS["キャラ名"] = { "<キャラ名>_none", ... }`という形でスキン一覧が登録され、これによって大ポートレートに使うプレハブ名が指定されます【74†L270-L274】。結果として、デフォルトのatlas/region文字列は `<プレハブ名>_none` と `<プレハブ名>_none_oval` で組み立てられます。スキン切替時は、`skinid`（DSTのスキンID）に応じて`<プレハブ名>_<skinid>`や`<プレハブ名>_<skinid>_oval`といった名前に変化します。つまり、デフォルト状態では  
```lua
prefab_none      = character_name .. "_none"
prefab_none_oval = prefab_none .. "_oval"
```  
となり、スキン指定があれば`skinid`の文字列を当てはめて名前を組み立てるロジックです【50†L200-L207】【74†L270-L274】。

## コード例（擬似コード）  
上記ロジックを擬似的にまとめると、例えば以下のようになります（引用例の `monkey_king` を一般化）：

```lua
-- キャラクター名と現在のスキン名を結合し、atlas/region名を生成
local prefab_none      = character_name .. "_none"            -- 例: "monkey_king_none"
local prefab_none_oval = prefab_none .. "_oval"              -- 例: "monkey_king_none_oval"
-- bigportraitsフォルダ内のxmlパスと、region文字列を指定
local atlas_path = "bigportraits/" .. prefab_none .. ".xml"   -- 例: "bigportraits/monkey_king_none.xml"
local region_name = prefab_none_oval .. ".tex"                -- 例: "monkey_king_none_oval.tex"
image_widget:SetTexture(atlas_path, region_name)
```

上記は*擬似コード*ですが、実際のDSTコードでも同様にプレハブ名＋`_none`（デフォルト）を組み合わせてアトラスファイルと領域名を生成し、`SetTexture(atlas, region)`でテクスチャをセットしています【18†L158-L166】。

## コンポーネント連携とデータフロー  
これらの文字列生成ロジックはキャラ選択UIのフローの中で呼び出されます。前述のエラーログ例から、DressupPanelコンポーネントがアニメーションスピナー（AnimSpinner）を経由して`SetPortraitImage`を呼び出している様子が見て取れます【83†L135-L143】。ユーザーがキャラやスキンを切り替える操作を行うと、まず`animspinner.lua`の`OnChanged`が発火し、続いて`dressuppanel.lua`の`Reset`が実行されます。最終的に`lobbyscreen.lua`内の`SetPortraitImage`で前述したatlas/region文字列が渡され、`image_widget:SetTexture`により画像が更新されます【83†L135-L143】【18†L158-L166】。この一連の流れにより、選択画面でプレイヤーが見ている丸型ポートレート（oval portrait）が適切に差し替えられます。

以上が、DSTのキャラクター選択UIにおいて**bigportraits**用のアトラスとリージョン文字列がどのように構成され、`SetTexture`がどこで呼ばれるかの解析です。スクリプト中ではパターンが決まっており、ここで挙げた規則に沿って正しいファイル名を指定すれば、ビッグポートレートも問題なく表示できるはずです【18†L158-L166】【31†L150-L154】。がんばってチャレンジしてみてください！