# LC

## 46.全排列

### 题目大意

给定一个不含重复数字的数组 `nums` ，返回其 **所有可能的全排列** 。你可以 **按任意顺序** 返回答案。

### 代码

```python
class Solution {
    List<List<Integer>> result = new ArrayList<>();
    LinkedList<Integer> path = new LinkedList<>();
    boolean[] used;
    public List<List<Integer>> permute(int[] nums) {
        used = new boolean[nums.length];
        backtracking(nums);
        return result;
    }

    private void backtracking(int nums[]){
        if(path.size() == nums.length){
            result.add(new ArrayList<>(path));
            return;
        }
        for(int i = 0; i < nums.length; i++){
            if(used[i] == true){
                continue;
            }
            used[i] = true;
            path.add(nums[i]);
            backtracking(nums);
            used[i] = false;
            path.removeLast();
        }
    }
}
```

## 47.全排列2

### 题目大意

给定一个**可包含重复**数字的序列 `nums` ，**按任意顺序** 返回所有不重复的全排列。

### 代码

```python
class Solution {
    List<List<Integer>> result = new ArrayList<>();
    LinkedList<Integer> path = new LinkedList<>();
    boolean[] used;
    public List<List<Integer>> permuteUnique(int[] nums) {
        used = new boolean[nums.length];
        Arrays.sort(nums);
        backtracking(nums);
        return result;
    }
    private void  backtracking(int[] nums){
        if(path.size() == nums.length){
            result.add(new ArrayList<>(path));
            return;
        }
        for(int i = 0; i< nums.length; i++){
            if(i > 0 && nums[i - 1] == nums[i] && used[i - 1] == true){
                continue;
            }
            if(used[i] == false){
                used[i] = true;
                path.add(nums[i]);
                backtracking(nums);
                used[i] = false;
                path.removeLast();
            }
        }
    }
}
```

## 48.旋转图像

给定一个 n × n 的二维矩阵 matrix 表示一个图像。请你将图像顺时针旋转 90 度。

你必须在 原地 旋转图像，这意味着你需要直接修改输入的二维矩阵。

### 代码

```python
class Solution {
    public void rotate(int[][] matrix) {
        int n = matrix.length;
        int[][] matrix_new = new int[n][n];
        for(int i = 0;i < n;i++){
            for(int j = 0;j<n;j++){
                matrix_new[j][n-i-1] = matrix[i][j];
            }
        }
        for(int i = 0;i<n;i++){
            for(int j = 0;j<n;j++){
                matrix[i][j] = matrix_new[i][j];
            }
        }
    }
}
```

## 54. 螺旋矩阵

### 题目大意

给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。

### 代码

模拟从左到右，从上到下

```java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> result = new ArrayList<>();
        int n = matrix.length;
        int m = matrix[0].length;
        int left = 0,right = n -1, top = 0,bottom = m - 1;
        while(left <= right && top <= bottom){
            for(int column = left; column <= right; column++){
                result.add(matrix[top][column]);
            }
            for(int row = top + 1; row <= bottom; row++){
                result.add(matrix[row][right]);
            }
            if(left < right && top < bottom){
                for(int column = right - 1; column > left; column--){
                    result.add(matrix[bottom][column]);
                }
                for(int row = bottom - 1; bottom > top; row--){
                    result.add(matrix[row][left]);
                }
            }
            left++;
            right--;
            top++;
            bottom--;
        }
        return result;
    }
}
```

## 53.最大子序和

### 题目大意

给定一个整数数组 `nums` ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

### 代码

```java
class Solution{
    public int maxSubArray(int[] nums){
        int result = Integer.MIN_VALUE;
        int count = 0;
        for(int i = 0;i<nums.length;i++){
            count = count + nums[i];
            result = Math.max(result,count);
            if(count < 0){
                count = 0;
            }
        }
        return result;
    }
}
```

## 55.跳跃游戏

### 题目大意

给定一个非负整数数组 `nums` ，你最初位于数组的 **第一个下标** 。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标。

### 代码

```java
class Solution {
    public boolean canJump(int[] nums) {
        int cover = 0;
        if(nums.length == 1){
            return true;
        }
        for(int i = 0;i <= cover; i++){
            cover = Math.max(nums[i] + i, cover);
            if(cover >= nums.length - 1){
                return true;
            }
        }
        return false;
    }
}
```

## 81. 搜索旋转排序数组Ⅱ

### 题目大意

已知存在一个按非降序排列的整数数组 nums ，数组中的值不必互不相同。

给你 旋转后 的数组 nums 和一个整数 target ，请你编写一个函数来判断给定的目标值是否存在于数组中。如果 nums 中存在这个目标值 target ，则返回 true ，否则返回 false 。

### 思路

二分查找

### 代码

```java
class Solution {
    public boolean search(int[] nums, int target) {
        int n = nums.length;
        if (n == 0) {
            return false;
        }
        if (n == 1) {
            return nums[0] == target;
        }
        int l = 0, r = n - 1;
        while (l <= r) {
            int mid = (l + r) / 2;
            if (nums[mid] == target) {
                return true;
            }
            if (nums[l] == nums[mid] && nums[mid] == nums[r]) {
                ++l;
                --r;
            } else if (nums[l] <= nums[mid]) {
                if (nums[l] <= target && target < nums[mid]) {
                    r = mid - 1;
                } else {
                    l = mid + 1;
                }
            } else {
                if (nums[mid] < target && target <= nums[n - 1]) {
                    l = mid + 1;
                } else {
                    r = mid - 1;
                }
            }
        }
        return false;
    }
}

```

## 1743.从相邻元素对还原数组(不会)

### 题目大意

存在一个由 `n` 个不同元素组成的整数数组 `nums` ，但你已经记不清具体内容。好在你还记得 `nums` 中的每一对相邻元素。

返回 **原始数组** `nums` 。如果存在多种解答，返回 **其中任意一个** 即可。

### 解题思路

存入哈希表，开头和末尾的元素只有一个与他们相邻。

### 代码

```java
class Solution {
    public int[] restoreArray(int[][] adjacentPairs) {
        Map<Integer, List<Integer>> map = new HashMap<Integer, List<Integer>>();
        for (int[] adjacentPair : adjacentPairs) {
            map.putIfAbsent(adjacentPair[0], new ArrayList<Integer>());
            map.putIfAbsent(adjacentPair[1], new ArrayList<Integer>());
            map.get(adjacentPair[0]).add(adjacentPair[1]);
            map.get(adjacentPair[1]).add(adjacentPair[0]);
        }

        int n = adjacentPairs.length + 1;
        int[] ret = new int[n];
        for (Map.Entry<Integer, List<Integer>> entry : map.entrySet()) {
            int e = entry.getKey();
            List<Integer> adj = entry.getValue();
            if (adj.size() == 1) {
                ret[0] = e;
                break;
            }
        }

        ret[1] = map.get(ret[0]).get(0);
        for (int i = 2; i < n; i++) {
            List<Integer> adj = map.get(ret[i - 1]);
            ret[i] = ret[i - 2] == adj.get(0) ? adj.get(1) : adj.get(0);
        }
        return ret;
    }
}


```

## 88 合并两个有序数组

### 题目大意

两个有序整数数 `nums1` 和 `nums2`，合并`num2`到`num1`中

### 解题思路

双指针，每次将`num1`或者`num2`对头较小的元素取出

### 代码1

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int p1 = 0, p2 = 0;
        int[] sorted = new int[m + n];
        int cur;
        while (p1 < m || p2 < n) {
            if (p1 == m) {
                cur = nums2[p2++];
            } else if (p2 == n) {
                cur = nums1[p1++];
            } else if (nums1[p1] < nums2[p2]) {
                cur = nums1[p1++];
            } else {
                cur = nums2[p2++];
            }
            sorted[p1 + p2 - 1] = cur;
        }
        for (int i = 0; i != m + n; ++i) {
            nums1[i] = sorted[i];
        }
    }
}
```

### 代码2

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int p1 = m - 1, p2 = n - 1;
        int tail = m + n - 1;
        int cur;
        while (p1 >= 0 || p2 >= 0) {
            if (p1 == -1) {
                cur = nums2[p2--];
            } else if (p2 == -1) {
                cur = nums1[p1--];
            } else if (nums1[p1] > nums2[p2]) {
                cur = nums1[p1--];
            } else {
                cur = nums2[p2--];
            }
            nums1[tail--] = cur;
        }
    }
}
```

## 91. 解码方法

### 题目大意

一条包含字母 `A-Z` 的消息通过以下映射进行了 **编码** ：

```
'A' -> 1
'B' -> 2
...
'Z' -> 26
```

**示例 1：**

```
输入：s = "12"
输出：2
解释：它可以解码为 "AB"（1 2）或者 "L"（12）。
```

### 解题思路

动态规划

### 代码

```java
class Solution {
    public int numDecodings(String s) {
        int n = s.length();
        int[] f = new int[n + 1];
        f[0] = 1;
        for (int i = 1; i <= n; ++i) {
            if (s.charAt(i - 1) != '0') {
                f[i] += f[i - 1];
            }
            if (i > 1 && s.charAt(i - 2) != '0' && ((s.charAt(i - 2) - '0') * 10 + (s.charAt(i - 1) - '0') <= 26)) {
                f[i] += f[i - 2];
            }
        }
        return f[n];
    }
}

```

## 92. 反转链表Ⅱ

### 题目大意

给你单链表的头指针 `head` 和两个整数 `left` 和 `right` ，其中 `left <= right` 。请你反转从位置` left `到位置 `right `的链表节点，返回 反转后的链表 。

### 解题思路

在需要反转的区间里，每遍历到一个节点，让这个新节点来到反转部分的起始位置。

- curr: 指向待反转区域的第一个节点`left`;
- next: 永远指向`curr`的下一个节点;
- pre: 永远指向待反转区域的第一个节点`left`的前一个节点，在循环过程中不变。

### 代码

```java
class Solution{
	public ListNode reverseBetween(ListNode head,int left,int right){
		ListNode dummy = new ListNode(0);
		dummy.next = head;
		ListNode pre = dummy;
		for(int i = 0 ;i<left-1;i++){
			pre = pre.next;
		}
		ListNode cur = pre.next;
		ListNode next;
		for(int i = 0;i<right-left;i++){
			next = cur.next;
			cur.next = next.next;
			next.next = pre.next;
			pre.next= next;
		}
		return dummy.next;
	}
}
```

## 93.复原IP地址

### 题目大意

给定一个只包含数字的字符串，用以表示一个 IP 地址，返回所有可能从 s 获得的 **有效 IP 地址** 。你可以按任何顺序返回答案。

**有效 IP 地址** 正好由四个整数（每个整数位于 0 到 255 之间组成，且不能含有前导 0），整数之间用 '.' 分隔。

例如："0.1.2.201" 和 "192.168.1.1" 是 **有效** IP 地址，但是 "0.011.255.245"、"192.168.1.312" 和 "192.168@1.1" 是 **无效** IP 地址。

### 解题思路

动态规划

每次截取一部分，判断是否合法。

判断子串是否合法

- 段位以0为开头的数字不合法
- 段位里有非整数字符不合法
- 段位如果大于255不合法

### 代码

```java
class Solution {
    List<String> result = new ArrayList<>();
    public List<String> restoreIpAddresses(String s) {
    if(s.length() > 12){
        return result;
    }
    backtracking(s,0,0);
    return result;
    }   

    //startIndex:搜索起始位置,pointNum:添加逗号的数量
    private void backtracking(String s, int startIndex, int pointNum){
        if(pointNum == 3){   //逗号数量为3，分隔结束
        // 判断是否合法
        if(isValid(s,startIndex,s.length()-1)){
            result.add(s);
        }
        return;
        }
        for(int i = startIndex;i<s.length();i++){
            if(isValid(s,startIndex,i)){
                s = s.substring(0,i+1)+"."+s.substring(i+1);
                pointNum++;
                backtracking(s,i+2,pointNum);
                pointNum--;
                s = s.substring(0,i+1)+s.substring(i+2);
            }
            
        }
    }

    private Boolean isValid(String s,int start,int end){
        if(start > end){
            return false;
        }
        if(s.charAt(start) == '0' && start != end){
            return false;
        }
        int num = 0;
        for(int i = start;i<=end;i++){
            if(s.charAt(i)>'9' ||s.charAt(i)<'0'){
                return false;
            }
            num = num*10 +(s.charAt(i) - '0');
            if(num >255){
                return false;
            }
        }
        return true;
    }
}
```

## 94.二叉树的中序遍历

### 题目大意

给定一个二叉树的根节点 `root` ，返回它的 **中序** 遍历。

### 代码

#### 1.递归

```java
class Solution {
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();
        inorder(root,res);
        return res;
    }
    public List<Integer> inorder(TreeNode root,List<Integer> res){
        if(root == null){
            return res;
        }
        inorder(root.left,res);
        res.add(root.val);
        inorder(root.right,res);
        return res;
    }
}
```

#### 2.迭代

```java
class Solution{
    public List<Integer> inorderTraversal(TreeNode root){
        List<Integer> res = new ArrayList<Integer>();
        Stack<TreeNode> stack = new Stack<TreeNode>();
        while(root != null || !stack.isEmpty()){
            while(root != null){
                stack.push(root);
                root = root.left;
            }
            root = stack.pop();
            res.add(root.val);
            root = root.right;
        }
        return res;
    }
}
```

## 97. 不同的二叉排序树

### 题目大意

给你一个整数 `n` ，求恰由 `n` 个节点组成且节点值从 `1` 到 `n` 互不相同的 **二叉搜索树** 有多少种？返回满足题意的二叉搜索树的种数。

### 解题思路

**动态规划**

给定一个有序序列 1⋯n，为了构建出一棵二叉搜索树，我们可以遍历每个数字 ii，将该数字作为树根，将 1⋯(i−1) 序列作为左子树，将 (i+1)⋯n 序列作为右子树。接着我们可以按照同样的方式递归构建左子树和右子树。

题目要求是计算不同二叉搜索树的个数。为此，我们可以定义两个函数：

G(n): 长度为 n 的序列能构成的不同二叉搜索树的个数。

F(i,n): 以 i 为根、序列长度为 n 的不同二叉搜索树个数(1≤i≤n)。

***F*(*i*,*n*)=*G*(*i*−1)⋅*G*(*n*−*i*)**

举例而言，创建以 33 为根、长度为 77 的不同二叉搜索树，整个序列是 [1, 2, 3, 4, 5, 6, 7][1,2,3,4,5,6,7]，我们需要从左子序列 [1, 2][1,2] 构建左子树，从右子序列 [4, 5, 6, 7][4,5,6,7] 构建右子树，然后将它们组合（即笛卡尔积）。

我们将 [1,2][1,2] 构建不同左子树的数量表示为 G(2)G(2), 从 [4, 5, 6, 7][4,5,6,7] 构建不同右子树的数量表示为 G(4)G(4)

### 代码

```java
class Solution {
    public int numTrees(int n) {
        int[] G = new int[n + 1];
        G[0] = 1;
        G[1] = 1;

        for (int i = 2; i <= n; ++i) {
            for (int j = 1; j <= i; ++j) {
                G[i] += G[j - 1] * G[i - j];
            }
        }
        return G[n];
    }
}
```

## 100 相同的树

### 题目大意

给你两棵二叉树的根节点 `p` 和 `q` ，编写一个函数来检验这两棵树是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

### 代码

```java
class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
       if(p == null && q == null){
           return true;
       }
       if(p != null && q!= null && p.val == q.val){
           return isSameTree(p.left,q.left)&&isSameTree(p.right,q.right);
       }
       else{
           return false;
       }
    }
}
```

## 101.对称二叉树

### 题目大意

给定一个二叉树，检查它是否是镜像对称的。

例如，二叉树 `[1,2,2,3,4,4,3]` 是对称的。

```
    1
   / \
  2   2
 / \ / \
3  4 4  3
```

### 代码

```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if (root == null)
            return true;
        return check(root.left,root.right);
    }
    public boolean check(TreeNode p,TreeNode q){
        //如果左右子树都为空
        if(p == null && q == null){
            return true;
        }
        //如果只有左右子树只有一个为空
        if(p == null || q == null){
            return false;
        }
        //如果左右子树的对应节点相等，并且判断他们的子节点
        return p.val ==q.val && check(p.left,q.right)&&check(p.right,q.left);
    }
}
```

## 102 二叉树的层序遍历

### 题目大意

给你一个二叉树，请你返回其按 **层序遍历** 得到的节点值。 （即逐层地，从左到右访问所有节点）。

### 代码

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if(root == null){
            return res;
        }
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        queue.offer(root);
        while(!queue.isEmpty()){
            List<Integer> level = new ArrayList<Integer>();
            int size = queue.size();
            for(int i = 0;i<size;i++){
                TreeNode node = queue.poll();
                level.add(node.val);
                if(node.left != null){
                    queue.offer(node.left);
                }
                if(node.right != null){
                    queue.offer(node.right);
                }
            }
            res.add(level);
        }
        return res;
    }
}
```

## 103. 二叉树的锯齿形层序遍历

### 题目大意

给定一个二叉树，返回其节点值的锯齿形层序遍历。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）。

### 代码

```java
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if(root == null){
            return res;
        }
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        queue.offer(root);
        int flag = 0;
        while(!queue.isEmpty()){
            int size = queue.size();
            List<Integer> level = new ArrayList<>();
            for(int i = 0;i<size;i++){
                TreeNode node = queue.poll();
                level.add(node.val);
                if(node.left != null){
                    queue.offer(node.left);
                }
                if(node.right != null){
                    queue.offer(node.right);
                }
            }
            if(flag%2 !=  0){
                Collections.reverse(level);
            }
            flag++;
            res.add(level);
        }
        return res;
    }
}
```

## 104.二叉树的最大深度

### 题目大意

给定一个二叉树，找出其最大深度。

二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

**说明:** 叶子节点是指没有子节点的节点。

### 代码

```java
class Solution{
	public int maxDepth(TreeNode root){
		if(root == null){
			return 0;
		}
		Queue<TreeNode> queue = new LinkedList<TreeNode>();
		queue.offer(root);
		int ans = 0;
		while(!queue.isEmpty()){
			int size = queue.size();
			while(size >0){
				TreeNode node = queue.poll();
				if(node.left != null){
					queue.offer(node.left);
				}
				if(node.right != null){
					queue.offer(node.right);
				}
				size--;
			}
			ans++;
		}
        return ans;
	}
}
```

## 107. 二叉树的层序遍历Ⅱ

### 题目大意

给定一个二叉树，返回其节点值自底向上的层序遍历。 （即按从叶子节点所在层到根节点所在的层，逐层从左向右遍历）

### 代码

```java
class Solution{
	public List<List<Integer>> levelOrderBottom(TreeNode root){
		List<List<Integer>> levelOrder = new LinkedList<List<Integer>>();
		if(root == null){
			return levelOrder;
		}
		Queue<TreeNode> queue = new LinkedList<TreeNode>();
		queue.offer(root);
		while(!queue.isEmpty()){
			List<Integer> level  = new ArrayList<Integer>();
			int size = queue.size();
			for(int i = 0;i<size ;i++){
				TreeNode node = queue.poll();
				level.add(node.val);
				TreeNode left = node.left;
				TreeNode right = node.right;
				if(left != null){
					queue.offer(left);
    
                }
				if(right != null){
					queue.offer(right);
				}
			}
			levelOrder.add(0,level);
		}
		return levelOrder;
	}
}
```

## 108.将有序数组转换为二叉搜索树

### 题目大意

给你一个整数数组 nums ，其中元素已经按 升序 排列，请你将其转换为一棵 高度平衡 二叉搜索树。

高度平衡 二叉树是一棵满足「每个节点的左右两个子树的高度差的绝对值不超过 1 」的二叉树。

### 代码

```java
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        return helper(nums, 0, nums.length - 1);
    }

    public TreeNode helper(int[] nums, int left, int right) {
        if (left > right) {
            return null;
        }

        // 总是选择中间位置左边的数字作为根节点
        int mid = (left + right) / 2;

        TreeNode root = new TreeNode(nums[mid]);
        root.left = helper(nums, left, mid - 1);
        root.right = helper(nums, mid + 1, right);
        return root;
    }
}


```

## 109.有序链表转换为二叉搜索树

### 题目大意

给定一个单链表，其中的元素按升序排序，将其转换为高度平衡的二叉搜索树。

本题中，一个高度平衡二叉树是指一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过 1。

### 代码

```java
```

## 110.平衡二叉树

### 题目大意

给定一个二叉树，判断它是否是高度平衡的二叉树。

本题中，一棵高度平衡二叉树定义为：

> 一个二叉树*每个节点* 的左右两个子树的高度差的绝对值不超过 1 。

### 代码

```java
class Solution {
    public boolean isBalanced(TreeNode root) {
		if(root == null){
			return true;
		}
		else{
			return Math.abs(height(root.left)-height(root.right))<=1 &&isBalanced(root.left)&&isBalanced(root.right);
		}
    }
	public int height(TreeNode root){
		if(root == null){
			return 0;
		}
		else{
			return Math.max(height(root.right),height(root.left))+1;
		}
	}
}
```



## 111. 二叉树的最小深度

### 题目大意

给定一个二叉树，找出其最小深度。

最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

**说明：**叶子节点是指没有子节点的节点。

### 代码

```java
class Solution {
    public int minDepth(TreeNode root) {
            if(root == null){
                return 0;
            }
            else if(root.left == null && root.right == null){
                return 1;
            }
            int min_depth = Integer.MAX_VALUE;
            if(root.left != null){
                min_depth = Math.min(minDepth(root.left),min_depth);
            }
            if(root.right != null){
                min_depth = Math.min(minDepth(root.right),min_depth);
            }
            return min_depth+1;
    }
}
```

## 112. 路径总和

### 题目大意

给你二叉树的根节点 `root` 和一个表示目标和的整数 `targetSum` ，判断该树中是否存在 根节点到叶子节点 的路径，这条路径上所有节点值相加等于目标和 `targetSum` 。

### 解题思路



### 代码

**广度优先**

```java
class Solution{
	public boolean hasPathSum(TreeNode root,int sum){
		if(root == null ){
			return false;
		}
		Queue<TreeNode> queNode = new LinkedList<TreeNode>();
		Queue<Integer> queVal = new LinkedList<Integer>();
		queNode.offer(root);
		queVal.offer(root.val);
		while( !queNode.isEmpty()){
			TreeNode now = queNode.poll();
			int temp = queVal.poll();
			if(now.left == null && now.right == null){
				if(temp == sum){
					return true;
				}
				continue;
			}
			if(now.left != null){
				queNode.offer(now.left);
				queVal.offer(now.left.val+temp);
			}
			if(now.right != null){
				queNode.offer(now.right);
				queVal.offer(now.right.val+temp);
			}
		}
		return false;
	}
}
```

**递归**

```java
class Solution {
    public boolean hasPathSum(TreeNode root, int sum) {
        if (root == null) {
            return false;
        }
        if (root.left == null && root.right == null) {
            return sum == root.val;
        }
        return hasPathSum(root.left, sum - root.val) || hasPathSum(root.right, sum - root.val);
    }
}

```

### 代码

```java
class Solution {

   int target = 0;
   int curSum = 0;
   List<List<Integer>> res = new LinkedList<>();
   LinkedList<Integer> path = new LinkedList<>();

    public List<List<Integer>> pathSum(TreeNode root, int targetSum) {
        this.target = targetSum;
        if (root == null) {
            return res;
        }
        traverse(root);
        return res;
    }

    public void traverse(TreeNode root) {
        if (root == null) {
            return;
        }
        // 前序遍历位置
        path.addLast(root.val);
        curSum += root.val;
        if (root.left == null && root.right == null) {
            if (curSum == target) {
                res.add(new LinkedList<>(path));
            }
        }

        traverse(root.left);
        traverse(root.right);

        // 后序遍历位置
        curSum -= root.val;
        path.removeLast();
    }
}



```



## 114. 二叉树展开为链表

### 题目大意

给你二叉树的根结点 `root `，请你将它展开为一个单链表：

展开后的单链表应该同样使用 `TreeNode` ，其中 `right `子指针指向链表中下一个结点，而左子指针始终为` null` 。
展开后的单链表应该与二叉树 **先序遍历** 顺序相同。

### 代码

```java
class Solution{
	public void flatten(TreeNode root){
		List<TreeNode> list = new ArrayList<TreeNode>();
		preorderTraversal(root,list);
		int size = list.size();
		for(int i = 1;i<size;i++){
			TreeNode prev = list.get(i-1),curr = list.get(i);
			prev.left = null;
			prev.right = curr;
		
		}
	}
	
	public void preorderTraversal(TreeNode root,List<TreeNode> list){
		if(root!= null){
			list.add(root);
			preorderTraversal(root.left,list);
			preorderTraversal(root.right,list);
        }
	}
}
```

## 115 不同的子序列

### 题目大意

给定一个字符串 `s` 和一个字符串 `t` ，计算在 `s` 的子序列中 `t` 出现的个数。

### 解题思路

动态规划

- 确定dp数组以及下标含义

`dp[i][j] :`以`i - 1`为结尾的s子序列中出现以`j - 1`为结尾的t的个数为`dp[i][j]`

- 确定递推公式
  - `s[i - 1]`与`t[i - 1]`相等
  - `s[i - 1]`与`t[i - 1]`不相等



## 116.填充每个节点的下一个右侧节点指针

### 题目大意

### 代码

- 层次遍历

```java
class Solution {
    public Node connect(Node root) {
        if(root == null){
            return root;
        }
        Queue<Node> queue = new LinkedList<>();
        queue.add(root);
        while(!queue.isEmpty()){
            int size = queue.size();
            for(int i = 0; i < size; i++){
                //取出队首元素
                Node node = queue.poll();
                if(i < size - 1){
                    //获取但不移除队首元素
                    node.next = queue.peek();
                }
                if(node.left != null){
                    queue.add(node.left);
                }
                if(node.right != null){
                    queue.add(node.right);
                }
            }
        }
        return root;
    }
}
```

## 123. 二叉树的最大路径和

#### 题目大意

**路径** 被定义为一条从树中任意节点出发，沿父节点-子节点连接，达到任意节点的序列。同一个节点在一条路径序列中 **至多出现一次** 。该路径 **至少包含一个** 节点，且不一定经过根节点。

**路径和** 是路径中各节点值的总和。

#### 解题思路

**递归**

`maxGain(node)`，计算二叉树中的一个节点的最大贡献值。就是在以该节点为根节点的子树中寻找以该节点为起点的一条路径，使得该路径上的节点值之和最大。

具体而言，该函数的计算如下。

- 空节点的最大贡献值等于 00。

- 非空节点的最大贡献值等于节点值与其子节点中的最大贡献值之和（对于叶节点而言，最大贡献值等于节点值）。

#### 代码

```java
class Solution {
    int maxSum = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        maxGain(root);
        return maxSum;
    }

    public int maxGain(TreeNode node) {
        if (node == null) {
            return 0;
        }
        
        // 递归计算左右子节点的最大贡献值
        // 只有在最大贡献值大于 0 时，才会选取对应子节点
        int leftGain = Math.max(maxGain(node.left), 0);
        int rightGain = Math.max(maxGain(node.right), 0);

        // 节点的最大路径和取决于该节点的值与该节点的左右子节点的最大贡献值
        int priceNewpath = node.val + leftGain + rightGain;

        // 更新答案
        maxSum = Math.max(maxSum, priceNewpath);

        // 返回节点的最大贡献值
        return node.val + Math.max(leftGain, rightGain);
    }
}
```

## 127. 最长连续序列

### 题目大意

给定一个未排序的整数数组 nums ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 O(n) 的算法解决此问题。



```java
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> num_set = new HashSet<Integer>();
        for (int num : nums) {
            num_set.add(num);
        }

        int longestStreak = 0;

        for (int num : num_set) {
            if (!num_set.contains(num - 1)) {
                int currentNum = num;
                int currentStreak = 1;

                while (num_set.contains(currentNum + 1)) {
                    currentNum += 1;
                    currentStreak += 1;
                }

                longestStreak = Math.max(longestStreak, currentStreak);
            }
        }

        return longestStreak;
    }
}

```

```java
class Solution {
    public int compareVersion(String version1, String version2) {
        String[] v1 = version1.split("\\.");
        String[] v2 = version2.split("\\.");
        for (int i = 0; i < v1.length || i < v2.length; ++i) {
            int x = 0, y = 0;
            if (i < v1.length) {
                x = Integer.parseInt(v1[i]);
            }
            if (i < v2.length) {
                y = Integer.parseInt(v2[i]);
            }
            if (x > y) {
                return 1;
            }
            if (x < y) {
                return -1;
            }
        }
        return 0;
    }
}

```

## 678 有效的括号字符串

### 题目大意

给定一个只包含三种字符的字符串：`（ `，`）` 和 `*`，写一个函数来检验这个字符串是否为有效字符串。

### 解题思路

动态规划

- dp数组 `dp[i][j]`表示字符串s从下标i到j的字串是否为有效的括号字符串

- 边界条件 1。当字串长度为1时，只有当字符是 * ，才为有效

  2。当字串长度为2时， 有4种情况， ① （） ② （*  ③  *） ④ **

- 动态转移方程

Ⅰ。 如果s[i]和s[j] 分别为左括号和右括号，也可以为 *， 则当dp[i + 1] [j - 1] = true时，`dp[i][j] = true`

Ⅱ。如果存在dp[i] [ k] 和dp[k + 1] [ j] 都为true，则dp[i] [ j]  = true

- 最终答案 **dp[0] [n - 1]**

### 代码

```java
class Solution {
    public boolean checkValidString(String s) {
        int n = s.length();
        boolean[][] dp = new boolean[n][n];
        for (int i = 0; i < n; i++) {
            if (s.charAt(i) == '*') {
                dp[i][i] = true;
            }
        }
        for (int i = 1; i < n; i++) {
            char c1 = s.charAt(i - 1), c2 = s.charAt(i);
            dp[i - 1][i] = (c1 == '(' || c1 == '*') && (c2 == ')' || c2 == '*');
        }
        for (int i = n - 3; i >= 0; i--) {
            char c1 = s.charAt(i);
            for (int j = i + 2; j < n; j++) {
                char c2 = s.charAt(j);
                if ((c1 == '(' || c1 == '*') && (c2 == ')' || c2 == '*')) {
                    dp[i][j] = dp[i + 1][j - 1];
                }
                for (int k = i; k < j && !dp[i][j]; k++) {
                    dp[i][j] = dp[i][k] && dp[k + 1][j];
                }
            }
        }
        return dp[0][n - 1];
    }
}`
```



## 42. 接雨水

### 题目大意

给定 `n` 个非负整数表示每个宽度为 `1` 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

<img src="E:\实习\Java_docs\pics\image-20211008181928571.png" alt="image-20211008181928571" style="zoom:67%;" />

### 代码

```java
class Solution {
    public int trap(int[] height) {
        if(height.length <= 1){
            return 0;
        }

        int max_height = 0;
        int max_height_index = 0;

        for(int i = 0; i < height.length; i++){
            int h = height[i];
            if(h > max_height){
                max_height = h;
                max_height_index = i;
            }
        }

        int area = 0;

        int temp = height[0];
        for(int i = 0; i < max_height_index; i++){
            if(height[i] > temp){
                temp = height[i];
            }
            else{
                area = area + (temp - height[i]);
            }
        }

        temp = height[height.length - 1];
        for(int i = height.length -1; i > max_height_index; i--){
            if(height[i] > temp){
                temp = height[i];
            }
            else{
                area = area + (temp - height[i]);
            }
        }
        return area;
    }
}
```

```java
class MyQueue {
    private Stack<Integer> a;// 输入栈
    private Stack<Integer> b;// 输出栈
    
    public MyQueue() {
        a = new Stack<>();
        b = new Stack<>();
    }
    
    public void push(int x) {
        a.push(x);
    }
    
    public int pop() {
        // 如果b栈为空，则将a栈全部弹出并压入b栈中，然后b.pop()
        if(b.isEmpty()){
            while(!a.isEmpty()){
                b.push(a.pop());
            }
        }
        return b.pop();
    }
    
    public int peek() {
        if(b.isEmpty()){
            while(!a.isEmpty()){
                b.push(a.pop());
            }
        }
        return b.peek();
    }
    
    public boolean empty() {
        return a.isEmpty() && b.isEmpty();
    }
}
```

