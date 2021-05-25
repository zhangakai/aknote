#树相关题目

---------------------------------------------

寻找重复子树:

    https://leetcode-cn.com/problems/find-duplicate-subtrees/
    思路:将树序列化 不同的树序列化一定不一样 用Map存序列化数据和对应出现的次数

    Map<String,Integer> map;
    List<TreeNode> res;
    public List<TreeNode> findDuplicateSubtrees(TreeNode root) {
        map = new HashMap<>();
        res = new ArrayList<>();
        serial(root);
        return res;
    }
    深度优先遍历
    public String  serial(TreeNode node){
        if(node==null){
            return "#";
        }
        String s = node.val+","+serial(node.left)+","+serial(node.right);
        map.merge(s,1,Integer::sum);
        if(map.get(s)==2){
            res.add(node);
        }
        return s;
    }