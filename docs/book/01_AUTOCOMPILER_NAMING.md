# 01 autocompiler：命名思想と例外

## 核心
最終的にゲームが探すのは **atlas（xml）** と **region（Element name / tex名）**。  
“ファイル名”より、**xml内の <Element name="...">** が強い。

## 基本ルール
多くのケースで、autocompilerは **PNG名 → 
ame.tex** として <Element name> を作る。  
だから勝ち筋はいつもこれ：
- ゲームが探す region 名に **PNG名（または Element name）を合わせる**

## 例外が起きる理由
- 生成物が古いまま（再生成されてない）
- PNG改名だけして xml/tex が更新されてない
- 参照キーが prefab ではなく skin id 側に切り替わってる

## 運用の2択
A) **PNG名で合わせる（推奨）**  
B) **生成後にxmlの Element name を直す**（再生成で戻るリスクあり）
