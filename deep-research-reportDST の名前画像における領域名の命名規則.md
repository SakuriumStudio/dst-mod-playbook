# DST の名前画像における領域名の命名規則

**期待される命名規則:** DST（Don’t Starve Together）のキャラクター名などを表示する画像（names_*.tex）のAtlas XMLでは、`<Element name=...>` に**プレハブ名+`.tex`**を指定するのが正しいパターンです。たとえばキャラ名が「esctemplate」であれば、XML 中の要素名は `esctemplate.tex` とします【6†L468-L472】【74†L144-L152】。Texture ファイル名は `names_esctemplate.tex`（`names_`プレフィックス付き）ですが、Element 名にはこの `names_` を含めません。実際、Klei フォーラムやガイドでは次のように説明されています：  
> `<Atlas><Texture filename="names_charactername.tex"/><Elements><Element name="charactername.tex" …/></Elements></Atlas>`【76†L198-L203】【15†L467-L474】.  

**autocompiler の挙動:** DSTの自動コンパイラ（autocompiler）は、デフォルトで画像ファイル名（`names_<prefix>.png`）に基づいてXMLを生成します。つまり、`names_<prefab>.png` をコンパイルすると、Texture ファイルは `names_<prefab>.tex`、Element 名もそのまま `names_<prefab>.tex` になります。しかしゲームは **Element 名に `names_` を含まない** ことを期待するため、このままだと名前が表示されません【46†L185-L193】【76†L198-L203】。たとえば「Jonathan」キャラの例では、autocompiler生成時に  
```xml
<Atlas><Texture filename="names_jonathan.tex"/><Elements><Element name="names_jonathan.tex" …/></Elements></Atlas>
```  
となってしまい、正しくは `<Element name="jonathan.tex" …/>` であるべきだと報告されています【46†L185-L193】。また、拡張サンプルキャラ（esctemplate）を流用した場合、autocompilerが要素名をテンプレート名「esctemplate」にしがちで、手動で修正が必要になります【76†L198-L203】。  

**命名ズレの原因と例:** autocompiler生成物と期待値が食い違う主な例をまとめると下表の通りです。

| 条件・状況                     | 自動生成されるElement名               | 期待されるElement名       | 備考                                                   |
|:---------------------------|:---------------------------|:----------------------|:-----------------------------------------------------|
| 通常の名前画像 (`names_*.png`) | `names_<キャラ名>.tex`        | `<キャラ名>.tex`         | ファイル名に `names_` が付くため、自動生成時にPrefix付き。        |
| 拡張サンプル (`esctemplate`)  | `esctemplate.tex`          | `<本来のキャラ名>.tex`    | テンプレート名そのまま。PNG名を変えてもElement名が古いまま残る。   |
| ゴールド名前画像 (`names_gold_*.png`) | `names_gold_<キャラ名>.tex` | `<キャラ名>.tex`         | 上記同様、`names_gold_` プレフィックス付きで生成される。          |

これらのケースでは、**自動生成後にXML内のElement名を手動で修正**し、`names_` や `names_gold_` を取り除く必要があります【46†L185-L193】【76†L198-L203】。拡張サンプル利用時は特に要注意で、V1.2.4のテンプレートに含まれる注意書きにも「`<Element name="charactername.tex">…` とすべし」と明記されています【76†L198-L203】。

**既存MODでの成功例:** 多くの完成度の高いキャラMODでは、上記の通りElement名をプレハブ名.texに合わせています。例えばチュートリアルガイドに掲載されているワトソンの場合、XMLは  
```xml
<Atlas><Texture filename="names_gold_watson.tex"/><Elements><Element name="watson.tex" …/></Elements></Atlas>
```  
となっており、`names_gold_` がつかない「watson.tex」が使用されています【54†L1-L4】。同様に、フォーラムで紹介されたカスタムキャラ Illyria の名前画像では、`names_illyria.tex` に対して `<Element name="illyria.tex">` を指定して成功しています【74†L144-L152】。これらはすべてプレハブ名で領域名を指定した例です。実際、多くのMOD開発者や公式ガイドでも**「Element名はキャラ名.texにせよ」**と明言されており、自動生成時の修正が暗黙の前提となっています【15†L467-L474】【54†L1-L4】。

```diff
+--------------------------------------------------------------------------+
| 成功例（MOD）        | Textureファイル名         | Element名（設定例）      | 備考                           |
|-------------------|-----------------------|----------------------|------------------------------|
| ガイド: Watson   | names_gold_watson.tex | watson.tex           | 公式ガイド掲載の例【54†L1-L4】   |
| フォーラム: Illyria | names_illyria.tex      | illyria.tex          | 実際のMODコード例【74†L144-L152】 |
+--------------------------------------------------------------------------+
```

**まとめ:** DST のキャラ選択画面で名前を表示するには、XML の `<Element name>` に**プレハブ名＋`.tex`**を指定する必要があります【76†L198-L203】【46†L185-L193】。自動コンパイラはファイル名（`names_…`）由来の領域名を出力するので、このままではゲーム側の期待とズレるため、手動でprefixを除去する対応が必須です。自動生成のクセ（特に拡張テンプレートのデフォルト名）さえ押さえておけば、MOD作成時の命名ミスは防げます【46†L185-L193】【76†L198-L203】。以上の知見を参考にすれば、DST MOD 開発の命名問題は安心して解決できるでしょう！ 

**参考資料:** Klei公式フォーラムやガイドでの説明【6†L468-L472】【15†L467-L474】【46†L185-L193】【54†L1-L4】【76†L198-L203】など。