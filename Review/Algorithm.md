#算法+智力题系列

------------------------------------
1.随机数转化
    
    rand(x) -> rand(y) leetcode原题
    https://leetcode-cn.com/problems/implement-rand10-using-rand7/solution/cong-zui-ji-chu-de-jiang-qi-ru-he-zuo-dao-jun-yun-/
    已知 rand_N() 可以等概率的生成[1, N]范围的随机数
    那么
    (rand_X() - 1) × Y + rand_Y() ==> 可以等概率的生成[1, X * Y]范围的随机数
    即实现了 rand_XY()

    