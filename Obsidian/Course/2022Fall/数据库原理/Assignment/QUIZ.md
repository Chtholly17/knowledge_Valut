## Q1
可包含4个指针，说明阶数为3，最多包含3个元素，最少包含一个元素
-insert 10
![[Pasted image 20221109104127.png]]
-insert 7
![[Pasted image 20221109104158.png]]
-insert 12
![[Pasted image 20221109104219.png]]
-insert 5
![[Pasted image 20221109105453.png]]
-insert 9
![[Pasted image 20221109105534.png]]
-insert 15
![[Pasted image 20221109105557.png]]
-insert 30
![[Pasted image 20221109105823.png]]
-insert 23
![[Pasted image 20221109105858.png]]
-insert 17
![[Pasted image 20221109110032.png]]
-insert 26
![[Pasted image 20221109110055.png]]
-delete 9
![[Pasted image 20221109110124.png]]
-delete 10
![[Pasted image 20221109110234.png]]
-delete 15
![[Pasted image 20221109111121.png]]
## Q2
- B-树，每个节点都具有一定的范围区间，可以存储多个值（指向磁盘中数据的指针）
- B+树，非叶子节点中不存储指向数据的指针，所有的指针均存放在叶子节点中，同时添加了叶子节点之间的指针
- 区别：
	- B+树内节点不存储数据，结构更加扁平，所有 data 存储在叶节点导致查询时间复杂度固定为 log n。而B-树查询时间复杂度不固定，与 key 在树中的位置有关，最好为O(1)
	- 由于B+树数据只存放在叶子节点中，且叶子节点两两相连，因此在区间查找上更具有优势。
