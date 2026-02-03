# 05 トラブルシューティング（ログ辞典）

## ログは仕様書
client_log.txt に出た **探してる atlas/region 文字列** が最重要。  
そこに合わせて作れば勝てる。

## 代表原因TOP
1) .tex/.xml が無い（PNGだけ）  
2) atlasパスが違う（置き場所違い）  
3) region（Element name）が違う（命名ズレ）  
4) *_none/_none_oval の作り忘れ  
5) prefab→skin id切替後にprefabで作ってる
