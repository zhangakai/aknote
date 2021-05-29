#算法题

拼多多


    多多君最近在研究某种数字组合：
    定义为：每个数字的十进制表示中(0~9)，每个数位各不相同且各个数位之和等于N。
    满足条件的数字可能很多，找到其中的最小值即可。
     public static int getMin(int num){
        if(num>45){
            return -1;
        }
        int[] res = new int[10];
        int index = 9;
        while(num>9){
            res[index]=index;
            num-=index;
            index--;
        }
        res[index]=num;
        int t = 0;
        for (int i = index; i < res.length; i++) {
            t=t*10+res[i];
        }
        return t;
    }

 

    多多路上从左到右有N棵树（编号1～N），其中第i个颗树有和谐值Ai。
    多多鸡认为，如果一段连续的树，它们的和谐值之和可以被M整除，那么这个区间整体看起来就是和谐的。
    现在多多鸡想请你帮忙计算一下，满足和谐条件的区间的数量。
     public static int sumOfPart(int[] nums,int k) {
        int[] rems = new int[k];
        rems[0]=1;
        int res=0;
        long sum=0;
        for (int num : nums) {
            sum+=num;
            int index = Math.floorMod(sum,k);
            res+=rems[index];
            rems[index]++;
        }
        return res;
    }