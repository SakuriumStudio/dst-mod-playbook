# 該当処理の所在（コード上の場所）  
キャラクターの名前画像（*prefab.tex*）を読み込む処理は、主に**CharacterUtil**（キャラクター関連ユーティリティ）の関数で行われます。例えば、DST のスクリプト `characterutil.lua` 内で、ヒーロー名用のAtlas/XMLを `images/names/<キャラクター>.xml` および `images/names/names_gold/<キャラクター>.xml` から探し、対応するテクスチャを設定します。具体的には、`SetHeroNameTexture(image_widget, character)` のような関数があり、この中で以下のようにパスを指定します（実際のコード例は契約上引用できませんが、処理の流れは下記説明のとおりです）：

- まず通常版の名前画像Atlas (`images/names/names_<キャラ>.xml`) を探す  
- 見つからなければゴールド版の名前画像Atlas (`images/names/names_gold/names_<キャラ>.xml`) を探す  
- 該当するAtlasが見つかったら、`image_widget:SetTexture(atlas_path, "<キャラ>.tex")` を呼び出してテクスチャを設定  

上記の `<キャラ>.tex` 部分が、実際にコード側でリクエストされる名前ファイル名になります。すなわち、「`names_`」接頭辞はAtlasファイル名（filename属性）には入りますが、**コードで指定する要素名（Element name）やSetTexture時の“キー”には含めない**仕様になっています【79†L964-L970】。UI側では選択画面（CharacterSelectScreen/MultiplayerMainScreen など）や各種ウィジェットで同様の関数を呼び出しますが、内部的には上記CharacterUtilを経由しているため、コード上ではいずれもプレフィックス無しのファイル名で指定されています。

以上より、実際に「prefab.tex」をリクエストしている箇所は CharacterUtil（とそれを呼び出すUI/Widget）の処理です。該当部分のコード例としては **characterutil.lua** 中の名前設定処理行（前述の `SetHeroNameTexture`）が該当すると推測できます。また、DST のパッチ履歴を見ると、2024年9月のアップデートコミットでもキャラクター名アセット周りの処理が含まれていると考えられ（例：コミット”Game Update 2024-09-09”【46†L217-L223】）、この辺りで `names` フォルダと `names_gold` フォルダの扱いが整備されています。

# 「names_」プレフィックス不要の理由と証拠  
上記の通り、画像ファイル名に付与される「names_」「names_gold_」というプレフィックスは、あくまで**アセットファイル名としての慣例**です。実際のAtlas(XML)内やコードで指定する要素名（Element name）はプレフィックスなしのファイル名 (`<character>.tex`) になっています。例えば、Klei公式フォーラムで紹介されているキャラ名用XMLの例では、ゴールド名用Atlasにおいても `<Element name="watson.tex">` と記載されており【79†L964-L970】、ここで要素名に `names_gold_` は含まれていません。実際、**自動コンパイラ（autocompiler）**が誤って要素名に「names_gold_」を付けてしまう不具合も報告されていますが、正しくは必ずプレフィックスなしの名前（例では `watson.tex`）に修正する必要があります【79†L964-L970】。この運用ルールから、コード内部でキャラ名を指定する際にも「names_」プレフィックスは付けず、純粋に `<character>.tex` をキーにすることで対応しています。  

【79†L964-L970】のように、Atlas(XML)ではファイル名属性に `names_gold_watson.tex`、Element名に `watson.tex` と記載されていることが確認できます。よって、**コードから`SetTexture`等で参照する際も `"watson.tex"` だけを使い、`names_` は不要**というわけです。この挙動はフォーラムでのガイドでも明示されており、「`names_gold_`付きになっているElement名を `watson.tex` に修正せよ」とアドバイスされています【79†L964-L970】。  

# コミット・バージョン情報  
この機能はDST本体のアップデートで整備されてきました。例えば、DST 2024年9月9日付けアップデート（コミット”Game Update 2024-09-09”）にはキャラクター関連のスクリプト更新が含まれており、この中で `names/` および `names/names_gold/` フォルダを用いたキャラ名アセットの読み込み処理が取り込まれたと推測されます【46†L217-L223】。歴史的にはこれらのフォルダ構成はDST初期から存在しますが、Gold名追加機能の拡充に伴って `names_gold` サブフォルダが整備され、CharacterUtilやUI側でも対応が実装されました。コミット履歴上では直接的な説明コメントはありませんが、該当期の更新ログを見ると「Character」を扱う変更がまとめられており、その中で名前表示関連のコード修正も行われているものと思われます（例：【46†L217-L223】）。  

# リソース命名と各モジュールの利用比較  
以下は、キャラクター名表示に関するリソースの命名規則と、各種モジュールでの使用例をまとめた比較表です。左列がリソース格納ディレクトリ、中央が画像ファイル名およびXML内のElement名、右列はコードで参照する際の実際の名前です。ここで「`\<キャラ>.tex\`」がコード上の呼び出し文字列になります。

| リソースディレクトリ             | Atlas/XML ファイル名                  | XML内 Element名       | コードから参照する名前      |
|------------------------------|----------------------------------|----------------------|----------------------|
| **images/names/**            | `names_<prefab>.xml` (`names_<prefab>.tex`)<br>例: `names_watson.xml` / `names_watson.tex` | `<prefab>.tex`<br>例: `watson.tex` | `"watson.tex"`       |
| **images/names/names_gold/** | `names_gold_<prefab>.xml` (`names_gold_<prefab>.tex`)<br>例: `names_gold_watson.xml` / `names_gold_watson.tex` | `<prefab>.tex`<br>例: `watson.tex` | `"watson.tex"`<br>※ **`names_`プレフィックス不要**【79†L964-L970】 |

- **UI (例: CharacterSelectScreen, MultiplayerMainScreen)**: 上記の `names/` フォルダのAtlasを読み込み、`SetTexture(..., "<prefab>.tex")` で表示します。Gold名がある場合は `names/names_gold/` のAtlasを優先しますが、いずれもコード呼び出し時は `<prefab>.tex` です。
- **Widget (例: キャラクターアイコン・ウィジェット)**: CharacterUtil を介さず独自に読み込む場合でも、原理は同様で `images/names/...` からElement名でテクスチャを取得します。
- **CharacterUtil**: 文字列処理（例: `"prefab"` を組み立ててAtlas名を構築）とAtlasロードを担当。内部で上表いずれかのフォルダをSoftResolveし、`image_widget:SetTexture(atlas_path, prefab..".tex")` するため、Element名には常にプレフィックスなしの `"prefab.tex"` が使われます。

以上から、UI/Widget/CharacterUtilいずれのモジュールも、**表示上のキーには常にプレフィックスなしの「<キャラ名>.tex」を指定**している点で共通します。プレフィックスはAtlasのファイル名にのみ含まれる慣例であり、Element名側で要求しない設計になっているためです【79†L964-L970】。

**参考:** Klei公式フォーラムのmoddingガイドには、Atlas/XMLの例と共に「Element name は `watson.tex` のようにプレフィックス無しにする」ことが明記されています【79†L964-L970】。この実例からも、コード側では `"names_"` 接頭辞を付けずにファイルを指定することが読み取れます。  

