# DSTのスキンシステムと参照キーの動作

DSTのキャラクターやアイテムにスキンを適用すると、画像参照のキー（ファイル名）が「Prefab名」からスキンIDに切り替わります。例えば、標準キャラのウィローなら本来はビルド名 “wilson” ですが、スキンシステムが有効だと常に“wilson_none”（デフォルトスキン）や “wilson_<skinID>” が使われます【109†L1149-L1151】【112†L159-L162】。つまりスキンなし状態でもゲームは内部的に `<キャラ>_none` を参照し、スキン適用中は `<キャラ>_<skinID>` を参照する仕様です。この挙動は DST公式スクリプト（`skinner`コンポーネント）によって制御されており、MakeModCharacterSkinnable → AddModCharacterSkin でスキン対応したキャラは「IsPrefabSkinned(character)==true」と判定され、選択画面では自動的に `character_none` が参照されるようになります【109†L1149-L1151】【112†L159-L162】。  

また、最新のスキンAPI（[API] Modded Skins）では、キャラを初期生成した直後はスキナーの `skin_name` が `nil` になってしまうことがあります。開発者Hornet氏によれば、これを防ぐにはスポーン時に必ず `skinner:SetSkinName("<キャラ>_none")` を呼び、デフォルトスキン名を明示的に設定する必要があります【103†L332-L336】。これを忘れると、デフォルト（None）状態でアニメやポートレートが正しく読み込まれずバグになります。  

## 参照される画像・アトラスとファイル命名

DSTではキャラ選択画面やHUDで使用する各種画像（大判ポートレート、アバターアイコン、キャラ名表示など）を`.xml`/.texファイルで定義します。スキンシステムが有効な場合、これらのファイル名にも「_<skinID>」や「_none」が付きます。以下に主なファイル名パターンと用途をまとめます（`<char>` はキャラクター識別子、`<skinID>` はスキンID、`<avatarset>` はAPIで指定したアバターセット名、`<uiname>` はカスタム名前画像名）。

| 種別                   | ファイル例（.tex/.xml）                          | 説明・備考 |
|:---------------------|:---------------------------------------------|:---------|
| 選択画面用大判ポートレート（通常）| `bigportraits/<char>_none.tex`<br>`bigportraits/<char>_none.xml`<br>（同フォルダ内で `<char>_none_oval.tex` を参照）【109†L1149-L1151】【112†L159-L162】 | デフォルト（スキンなし）時のキャラ画像。DSTは`<char>_none`キーで探し、`.xml`の中で同名の`_oval.tex`（楕円マスクされたテクスチャ）を使います【112†L159-L162】。例：ウィローなら `wilson_none.tex`/`.xml` （および `wilson_none_oval.tex`）。 |
| 選択画面用大判ポートレート（スキン別）| `bigportraits/<char>_<skinID>.tex`<br>`bigportraits/<char>_<skinID>.xml`<br>（`<skinID>_oval.tex` を参照） | 指定スキンが選択された際のキャラ画像。対応する`.xml`は必ず`<char>_<skinID>.xml`とし、中で`<char>_<skinID>_oval.tex`を使います。スキンIDは `AddModCharacterSkin` で定義した文字列をそのまま使います。 |
| HUDアバター（小アイコン）  | `images/avatar_<char>.tex`/`.xml`<br>`images/avatar_<avatarset>.tex`/`.xml`【106†L1-L4】 | 通常HUD右上のキャラアイコン（丸型）。スキンAPIの `options.avatarset` 指定があれば、`avatar_<avatarset>` ファイルに差し替え【106†L1-L4】。例：`avatar_wilson` または `avatar_roboticcharacter`。 |
| キャラ名表示（選択画面下部）| `images/names_<char>.tex`/`.xml`<br>`images/names_<uiname>.tex`/`.xml`【106†L3-L4】 | キャラ名やスキン名などを表示する背景画像。デフォルトは `names_<char>` ですが、`options.uiname`指定で `names_<uiname>` を使います【106†L3-L4】。 |
| インスペクト画面画像    | `images/self_inspect_<char>.tex`/`.xml`<br>`images/self_inspect_<avatarset>.tex`/`.xml`【106†L1-L4】 | キャラ詳細画面（Fキー等）のバストアップ。通常は `self_inspect_<char>`、アバターセット指定時は `self_inspect_<avatarset>`。 |
| その他HUD要素         | `images/portrait_<char>.tex`/`.xml`など | （通常は内部処理に伴う画像。必要に応じて同様の命名規則で用意する。） |

例えば、NamelessName氏のコードでは「`bigportraits/monkey_king_none.xml` が内部的に `monkey_king_none_oval.tex` を参照している」と報告されています【112†L159-L162】。DSTは**必ず**`<char>_none`というファイルを探すため、スキンを追加したキャラには必須です【109†L1149-L1151】。

## 既存MODの実装例と対策

多くの既存MODでは、上記の命名規則に従って画像を用意し、スキン変更時にアニメーションや画像を切り替えています。代表的な手法は以下の通りです：

- **OnSkinsChanged（SetSkinName）フック**：スキン変更イベントにフックして、`inst.AnimState` や `inventoryitem` の画像名を更新します。公式フォーラムのサンプルでは `skinner:SetSkinName` 内で分岐処理を行い、スキン名に応じてアニメーションビルドを切り替えるよう案内されています【110†L1278-L1282】。たとえば、`basic_init_fn(inst, build_name)` の後に `inst.AnimState:SetSkin(build_name, "char_build")` とし、最後に `ChangeImageName(inst:GetSkinName())` を呼ぶことで、アイテムのインベントリアイコンも更新できます【117†L689-L697】【110†L1278-L1282】。
- **デフォルトスキン名の設定**：上記の通り、`SetSkinName("<char>_none")` をスポーン時に呼ぶMODもあります【103†L332-L336】。Hornet氏は「デフォルトスキン名がnilにならないよう、キャラ生成後に必ず`<char>_none`を設定する」ことで動作安定化を勧めています【103†L332-L336】。
- **画像ファイルの配置**：Fidooop氏らのチュートリアルやユーザー報告では、「必ず `bigportraits/<char>_none` を置く」「スキンIDで画像ファイル名を付ける」ことが強調されています【109†L1149-L1151】【112†L159-L162】。実例として「キャラ識別子が writer なら、デフォルトは `writer_none` という大判ポートレートを用意せよ」というアドバイスがなされています【109†L1149-L1151】。
- **Avatar/SelfInspectの切り替え**：スキンAPIの `avatarset` オプションを利用するMODでは、アバター画像やインスペクト画像にカスタムファイル名を割り当てています【106†L1-L4】。これにより、スキン時のみ別のアバター画像を表示できます。

## 実装のベストプラクティス

以上の知見を踏まえ、MOD開発者向けの実装ガイドラインを整理します：

1. **モッドキャラをスキン対応にする**：`modmain.lua` で `MakeModCharacterSkinnable("<キャラPrefab>")` を呼び、`AddModCharacterSkin` で各スキンを登録します【105†L273-L281】。これによりDST側で該当キャラのスキン判定が有効になります。
2. **デフォルトスキン名を設定**：キャラスポーン（`AddPlayerPostInit` など）時に必ず `inst.components.skinner:SetSkinName("<キャラ>_none", true)` を呼び出しましょう【103†L332-L336】。これでデフォルト（無し）状態の `skin_name` が `nil` ではなく `"キャラ_none"` となり、参照整合性が保たれます。
3. **画像ファイルを用意**：先述の表に従い、必須画像をすべて用意します。特に**選択画面ポートレートは `<char>_none`（と楕円版`_none_oval`）**が必須です【109†L1149-L1151】【112†L159-L162】。スキンごとには `<char>_<skinID>` 画像も同様に配置します。必要に応じて `images/avatar_*.xml` や `images/names_*.xml` も用意してください【106†L1-L4】。
4. **OnSkinsChangedで切り替え処理**：`components.skinner:OnSkinsChanged()` あるいは `SetSkinName` 呼び出し後の処理で、`inst.AnimState:SetBuild`/`SetSkin` や `ChangeImageName` を行います。フォーラム例では `skinner:SetSkinName` にフックし、スキン名に合わせてビルド名をセットするコードが紹介されています【110†L1278-L1282】。同時に `inst.components.inventoryitem:ChangeImageName(inst:GetSkinName())` を使えば、インベントリアイコンも切り替え可能です【117†L689-L697】。
5. **アトラス登録**：アイテムなどで `ChangeImageName` を使う場合、エンジンは画像名のハッシュ値でアトラスを検索します【62†L114-L121】。そのため、Atlas登録時はハッシュ（`texd_hash`）をキーにするか、`RegisterInventoryItemAtlas(inst, "images/inventoryimages/<atlas>.xml")` のように適切に登録しておきます。
6. **検証とデバッグ**：Modのロードログやゲーム内で `print(inst:GetSkinName())` を挟んで、予期したキーでスキンが設定されているか確認します。参照エラーが出るとファイル名のミスの可能性が高いので、必ず `.xml` 中の `<atlases>` や `<atlas>` 名、`.tex` 名が一致しているかチェックしましょう。

以上の方法を採れば、DSTでスキン付きキャラのポートレートが確実に表示されるはずです。特に **「キャラ_none」 ファイルの用意** と **`SetSkinName("<char>_none")` の設定** はよく見落とされがちな重要ポイントなので注意してください【109†L1149-L1151】【103†L332-L336】。

**参考資料:** Klei公式フォーラムのモッドスキンチュートリアルや開発者投稿【109†L1149-L1151】【110†L1278-L1282】【103†L332-L336】【106†L1-L4】、および実際のモッド実装例。これらから得られた知見を基に上記説明をまとめました。

