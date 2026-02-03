# 03 prefab → skin id 切替条件と対策

スキンシステムが有効になると、UIが探すキーが  
**prefab名 → skin id（skin key）** に切り替わることがある。

## 対策の最短2択
A) **prefab版とskin id版を両方用意**（最短で勝つ）  
B) **参照キーをprefab側に固定**（上級：実装寄り）
