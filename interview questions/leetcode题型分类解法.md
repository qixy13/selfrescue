### leetcode题型分类解法

#### 二叉树相关

##### 114.二叉树展开为链表

```go
//注意到前序遍历访问各节点的顺序是根节点、左子树、右子树。如果一个节点的左子节点为空，则该节点不需要进行展开操作。如果一个节点的左子节点不为空，则该节点的左子树中的最后一个节点被访问之后，该节点的右子节点被访问。该节点的左子树中最后一个被访问的节点是左子树中的最右边的节点，也是该节点的前驱节点。因此，问题转化成寻找当前节点的前驱节点。

//具体做法是，对于当前节点，如果其左子节点不为空，则在其左子树中找到最右边的节点，作为前驱节点，将当前节点的右子节点赋给前驱节点的右子节点，然后将当前节点的左子节点赋给当前节点的右子节点，并将当前节点的左子节点设为空。对当前节点处理结束后，继续处理链表中的下一个节点，直到所有节点都处理结束。


func flatten(root *TreeNode)  {
    curr := root
    for curr != nil {
        if curr.Left != nil {
            next := curr.Left
            predecessor := next
            for predecessor.Right != nil {
                predecessor = predecessor.Right
            }
            predecessor.Right = curr.Right
            curr.Left, curr.Right = nil, next
        }
        curr = curr.Right
    }
}

func flatten(root *TreeNode) {
    if root == nil {
        return
    }
    left, right := root.Left, root.Right
    predecessor := left
    if left != nil {
        for predecessor.Right != nil {
            predecessor = predecessor.Right
        }
        predecessor.Right = right
        root.Left = nil
        root.Right = left
    }
    flatten(root.Right)
}



```

##### 113.路径总和ii

```go
//我们可以采用深度优先搜索的方式，枚举每一条从根节点到叶子节点的路径。当我们遍历到叶子节点，且此时路径和恰为目标和时，我们就找到了一条满足条件的路径。
func pathSum(root *TreeNode, targetSum int) [][]int {
    path := []int{}
    var dfs func(*TreeNode, int)
    ans := [][]int{}
    dfs = func(node *TreeNode, target int) {
        if node == nil {
            return
        }
        target -= node.Val
        path = append(path, node.Val)
        defer func() { path = path[:len(path)-1] }()//回溯操作
        if node.Left == nil && node.Right == nil && target == 0 {
            ans = append(ans, append([]int(nil), path...))
            return
        }
        dfs(node.Left, target)
        dfs(node.Right, target)
    }
    dfs(root, targetSum)
    return ans
}
```

##### 103.锯齿遍历

```go
func zigzaglevelorder(root *TreeNode) (ans [][]int) {
    if root == nil {
        return 
    }

    queue := []*TreeNode{root}
    for level := 0; len(queue) > 0; level++ {
        vals := []int{}
        q := queue
        queue = nil
        for _, node := range q {
            vals = append(vals, node.Val)
            if node.Left != nil {
                queue = append(queue, node.Left)
            }

            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }
        if level % 2 == 1 {//当前是奇数层，需要先加入右子树，再加入左子树
            for i, n := 0, len(vals); i < n/2; i++ {
                vals[i], vals[n-1-i] = vals[n-1-i], vals[i]
            }
        }
        ans = append(ans, vals)
    }

    return
}

```

##### 105.中序+前序构造二叉树

```go
func buildTree(preorder []int, inorder []int) *TreeNode {
    if len(preorder) == 0 {return nil}

    root := &TreeNode{preorder[0], nil, nil}
    i := 0
    for ; i < len(inorder); i++ {
        if inorder[i] == root.Val {
            break
        }
    }

    //得到左子树的长度len(inorder[:i])，对preorder做分割，分割出左右子树

    root.Left = buildTree(preorder[1:len(inorder[:i])+1], inorder[:i])
    root.Right = buildTree(preorder[len(inorder[:i])+1:], inorder[i+1:])
    
    return root
}
```

##### 112.路径总和

```go
var total int
func hasPathSum(root *TreeNode, targetSum int) bool {
    //初始值为targetsum，不断减去各个节点的值
    var flag bool 
    total = targetSum
    if root == nil {
        return flag
    }
    pathsum(root, total, &flag)
    return flag
}
func pathsum(node *TreeNode, total int, flag *bool) {
    total -= node.Val
    if node.Left == nil && node.Right == nil && total == 0 {
        *flag = true
        return
    }
    if node.Left != nil && *flag == false {
        pathsum(node.Left, total, flag)
    }
    if node.Right != nil && *flag == false {
        pathsum(node.Right, total, flag)
    } 
}
```

##### 98.验证二叉搜索树

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func isValidBST(root *TreeNode) bool {
    return helper(root, math.MinInt64, math.MaxInt64)
}

func helper(root *TreeNode, lower, upper int) bool {
    if root == nil {
        return true
    }

    if root.Val <= lower || root.Val >= upper {
        return false
    }

    return helper(root.Left, lower, root.Val) && helper(root.Right, root.Val, upper)
}
```

##### 层序遍历从下到上

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func levelOrderBottom(root *TreeNode) [][]int {
    levelOrder := [][]int{}
    if root == nil {
        return levelOrder
    }
    queue := []*TreeNode{}
    queue = append(queue, root)
    for len(queue) > 0 {
        level := []int{}
        size := len(queue)
        for i := 0; i < size; i++ {
            node := queue[0]
            queue = queue[1:]
            level = append(level, node.Val)
            if node.Left != nil {
                queue = append(queue, node.Left)
            }
            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }
        levelOrder = append(levelOrder, level)
    }
    for i := 0; i < len(levelOrder) / 2; i++ {
        levelOrder[i], levelOrder[len(levelOrder) - 1 - i] = levelOrder[len(levelOrder) - 1 - i], levelOrder[i]
    }
    return levelOrder
}


```

#### 回溯算法相关

##### 17.电话号码组合

```go
var phoneMap map[string]string = map[string]string{
    "2": "abc",
    "3": "def",
    "4": "ghi",
    "5": "jkl",
    "6": "mno",
    "7": "pqrs",
    "8": "tuv",
    "9": "wxyz",
}

var combinations []string

func letterCombinations(digits string) []string {
    if len(digits) == 0 {
        return []string{}
    }
    combinations = []string{}
    backtrack(digits, 0, "")
    return combinations
}

func backtrack(digits string, index int, combination string) {
    if index == len(digits) {
        combinations = append(combinations, combination)
    } else {
        digit := string(digits[index])
        letters := phoneMap[digit]
        lettersCount := len(letters)
        for i := 0; i < lettersCount; i++ {
            backtrack(digits, index + 1, combination + string(letters[i]))
        }
    }
}
```

##### 37.解数独

```go

```

##### 79.单词搜索i

```go
type pair struct{ x, y int }

var directions = []pair{{-1, 0}, {1, 0}, {0, -1}, {0, 1}} // 上下左右

func exist(board [][]byte, word string) bool {
    h, w := len(board), len(board[0])
    vis := make([][]bool, h)
    for i := range vis {
        vis[i] = make([]bool, w)
    }
    var check func(i, j, k int) bool
    check = func(i, j, k int) bool {
        if board[i][j] != word[k] { // 剪枝：当前字符不匹配
            return false
        }
        if k == len(word)-1 { // 单词存在于网格中
            return true
        }
        vis[i][j] = true
        defer func() { vis[i][j] = false }() // 回溯时还原已访问的单元格
        for _, dir := range directions {
            if newI, newJ := i+dir.x, j+dir.y; 0 <= newI && newI < h && 0 <= newJ && newJ < w && !vis[newI][newJ] {
                if check(newI, newJ, k+1) {
                    return true
                }
            }
        }
        return false
    }
    for i, row := range board {
        for j := range row {
            if check(i, j, 0) {
                return true
            }
        }
    }
    return false
}
```

##### 131.分割回文串

```go
func partition(s string) (ans [][]string) {
    n := len(s)
    f := make([][]bool, n)
    for i := range f {
        f[i] = make([]bool, n)
        for j := range f[i] {
            f[i][j] = true
        }
    }

    for i := n-1; i >= 0; i-- {
        for j := i+1; j < n; j++{
            f[i][j] = s[i] == s[j] && f[i+1][j-1]
        } 
    }
    splits := []string{}//不知道长度的时候，切片就赋值成这个样子
    var dfs func(int)
    dfs = func(i int) {
        if i == n {
            ans = append(ans, append([]string(nil), splits...))
            return
        }

        for j := i; j < n; j++ {
            if f[i][j] {
                splits = append(splits, s[i:j+1])
                dfs(j+1)
                splits = splits[:len(splits)-1]
            }
        }
    }

    dfs(0)
    return
}
```

##### 39.组合总和

```go
func combinationSum(candidates []int, target int) [][]int {
    var track []int
    var res [][]int
    backtracking(0,0,target,candidates,track,&res)
    return res
}
func backtracking(startIndex,sum,target int,candidates,track []int,res *[][]int){
    //终止条件
    if sum==target{
        tmp:=make([]int,len(track))
        copy(tmp,track)//拷贝
        *res=append(*res,tmp)//放入结果集
        return
    }
    if sum>target{return}
    //回溯
    for i:=startIndex;i<len(candidates);i++{
        //更新路径集合和sum
        track=append(track,candidates[i])
        sum+=candidates[i]
        //递归
        backtracking(i,sum,target,candidates,track,res)
        //回溯
        track=track[:len(track)-1]
        sum-=candidates[i]
    }

}
```

##### 40.组合总和2

```go
func combinationSum2(candidates []int, target int) (ans [][]int) {
    sort.Ints(candidates)
    var freq [][2]int
    for _, num := range candidates {
        if freq == nil || num != freq[len(freq)-1][0] {
            freq = append(freq, [2]int{num, 1})
        } else {
            freq[len(freq)-1][1]++
        }
    }

    var sequence []int
    var dfs func(pos, rest int)
    dfs = func(pos, rest int) {
        if rest == 0 {
            ans = append(ans, append([]int(nil), sequence...))
            return
        }
        if pos == len(freq) || rest < freq[pos][0] {
            return
        }

        dfs(pos+1, rest)

        most := min(rest/freq[pos][0], freq[pos][1])
        for i := 1; i <= most; i++ {
            sequence = append(sequence, freq[pos][0])
            dfs(pos+1, rest-i*freq[pos][0])
        }
        sequence = sequence[:len(sequence)-most]
    }
    dfs(0, target)
    return
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

##### 216.组合总和3

```go
func combinationSum3(k int, n int) [][]int {
    ans:=[][]int{}
    path:=[]int{}
    var dfs func(t int)
    dfs = func(t int){
        if n==0 && len(path)==k{
            ans = append(ans,append([]int(nil),path...))
            return
        }
        if t==10 || n-k<0{
            return
        }
        //跳过当前数
        dfs(t+1)
        //不跳过
        path = append(path,t)
        n-=t
        dfs(t+1)
        //回溯
        path = path[:len(path)-1]
        n+=t
    }
    dfs(1)
    return ans
}
```

##### 排列组合

###### 31.下一个排列

```go
func nextPermutation(nums []int)  {
    if len(nums) < 1{
        return
    }
    i,j,k := len(nums)-2,len(nums)-1,len(nums)-1
    for i>=0 && nums[i]>=nums[j] {//从后面开始遍历找到第一个前一个数小于后一个数的时候
        i--
        j--
    }
    //此时找到了一个前一个数小于后一个数的情况，跳出循环
    if i>=0 {//确定不是最后一个元素
        //找到第一个比nums[i]大的整数
        for nums[i] >= nums[k] {
            k--
        }
        //找到后交换两个数
        nums[i],nums[k] = nums[k],nums[i]
    }
    //此时[j,len(nums))这段序列的数是降序排列，变为升序
        for i,j:=j,len(nums)-1; i<j; i,j = i+1,j-1{
            nums[i], nums[j] = nums[j], nums[i]
        }
}
```

###### 46.全排列

```go
// 最终结果
var result [][] int

// 回溯核心
// nums: 原始列表
// pathNums: 路径上的数字
// used: 是否访问过
func backtrack(nums, pathNums []int, used[]bool) {
    // 结束条件：走完了，也就是路径上的数字总数等于原始列表总数
    if len(nums) == len(pathNums) {
        tmp := make([]int, len(nums))
        // 切片底层公用数据，所以要copy
        copy(tmp, pathNums)
        // 把本次结果追加到最终结果上
        result = append(result, tmp)
        return
    }

    // 开始遍历原始数组的每个数字
    for i:=0; i<len(nums); i++ {
        // 检查是否访问过
        if !used[i] {
            // 没有访问过就选择它，然后标记成已访问过的
            used[i] = true
            // 做选择：将这个数字加入到路径的尾部，这里用数组模拟链表
            pathNums = append(pathNums, nums[i])
            backtrack(nums,pathNums,used)
            // 撤销刚才的选择，也就是恢复操作
            pathNums = pathNums[:len(pathNums) -1]
            // 标记成未使用
            used[i] = false
        }
    }
}

func permute(nums []int) [][]int {
    var pathNums []int
    var used = make([]bool, len(nums))
    // 清空全局数组（leetcode多次执行全局变量不会消失）
    result = [][]int{}
    backtrack(nums, pathNums, used)
    return result
}
```

###### 47.全排列2

```go
var res [][]int
func permuteUnique(nums []int) [][]int {
    res = [][]int{}
    backTrack(nums,len(nums),[]int{})
    return res
}
func backTrack(nums []int,numsLen int,path []int)  {
    if len(nums)==0{
        p:=make([]int,len(path))
        copy(p,path)
        res = append(res,p)
    }
    used := [21]int{}//跟前一题唯一的区别，同一层不使用重复的数。关于used的思想carl在递增子序列那一题中提到过
    //同层不能再使用，同列可以使用重复的元素
    for i:=0;i<numsLen;i++{
        if used[nums[i]+10]==1{
            continue
        }
        cur:=nums[i]
        path = append(path,cur)
        used[nums[i]+10]=1
        nums = append(nums[:i],nums[i+1:]...)
        backTrack(nums,len(nums),path)
        nums = append(nums[:i],append([]int{cur},nums[i:]...)...)
        path = path[:len(path)-1]

    }

}
```

###### 78.子集

```go
// 单看每个元素，都有两种选择：选入子集，或不选入子集。

// 比如[1,2,3]，先看1，选1或不选1，都会再看2，选2或不选2，以此类推。

// 考察当前枚举的数，基于选它而继续，是一个递归分支；基于不选它而继续，又是一个分支。
func subsets(nums []int) (ans [][]int) {
    set := []int{}
    var dfs func(int)
    dfs = func(cur int) {
        if cur == len(nums) {
            ans = append(ans, append([]int(nil), set...))
            return
        }
        set = append(set, nums[cur])
        dfs(cur + 1)
        set = set[:len(set)-1]
        dfs(cur + 1)
    }
    dfs(0)
    return
}
```

###### 90.子集2

```go
var res[][]int
func subsetsWithDup(nums []int)[][]int {
    res=make([][]int,0)
    sort.Ints(nums)
    dfs([]int{},nums,0)
    return res
}
func dfs(temp, num []int, start int)  {
    tmp:=make([]int,len(temp))
    copy(tmp,temp)

    res=append(res,tmp)
    for i:=start;i<len(num);i++{
        if i>start&&num[i]==num[i-1]{//去重，当前元素如果之前使用过则不能再使用
            continue
        }
        temp=append(temp,num[i])
        dfs(temp,num,i+1)
        temp=temp[:len(temp)-1]
    }
}
```

##### 93.IP地址复原

```go
func restoreIpAddresses(s string) []string {
    var res,path []string
    backTracking(s,path,0,&res)
    return res
}
func backTracking(s string,path []string,startIndex int,res *[]string){
    //终止条件
    if startIndex==len(s)&&len(path)==4{
        tmpIpString:=path[0]+"."+path[1]+"."+path[2]+"."+path[3]
        *res=append(*res,tmpIpString)
    }
    for i:=startIndex;i<len(s);i++{
        //处理
        path:=append(path,s[startIndex:i+1])
        if i-startIndex+1<=3&&len(path)<=4&&isNormalIp(s,startIndex,i){
            //递归
            backTracking(s,path,i+1,res)
        }else {//如果首尾超过了3个，或路径多余4个，或前导为0，或大于255，直接回退
            return
        }
        //回溯
        path=path[:len(path)-1]
    }
}
func isNormalIp(s string,startIndex,end int)bool{
    checkInt,_:=strconv.Atoi(s[startIndex:end+1])
    if end-startIndex+1>1&&s[startIndex]=='0'{//对于前导 0的IP（特别注意s[startIndex]=='0'的判断，不应该写成s[startIndex]==0，因为s截取出来不是数字）
        return false
    }
    if checkInt>255{
        return false
    }
    return true
}
```

#### 动态规划

##### 124.二叉树的最大路径和

```go
func maxPathSum(root *TreeNode) int {
    maxSum := math.MinInt32
    var maxGain func(*TreeNode) int
    maxGain = func(node *TreeNode) int {
        if node == nil {
            return 0
        }

        // 递归计算左右子节点的最大贡献值
        // 只有在最大贡献值大于 0 时，才会选取对应子节点
        leftGain := max(maxGain(node.Left), 0)
        rightGain := max(maxGain(node.Right), 0)

        // 节点的最大路径和取决于该节点的值与该节点的左右子节点的最大贡献值
        priceNewPath := node.Val + leftGain + rightGain

        // 更新答案
        maxSum = max(maxSum, priceNewPath)

        // 返回节点的最大贡献值
        return node.Val + max(leftGain, rightGain)
    }
    maxGain(root)
    return maxSum
}

func max(x, y int) int {
    if x > y {
        return x
    }
    return y
}
```

##### 1143.最长公共子序列

```go
func longestCommonSubsequence(text1, text2 string) int {
    m, n := len(text1), len(text2)
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    for i, c1 := range text1 {
        for j, c2 := range text2 {
            if c1 == c2 {
                dp[i+1][j+1] = dp[i][j] + 1//注意下标计算，不能为负数，所以从i+1开始
            } else {
                dp[i+1][j+1] = max(dp[i][j+1], dp[i+1][j])
            }
        }
    }
    return dp[m][n]
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

#### 状态机

##### 8.atoi

```go
type State int
type CharType int

const (
    STATE_START State = iota
    STATE_SIGN
    STATE_NUMBER
    STATE_END
)

const (
    CHAR_SPACE CharType = iota
    CHAR_NUMBER
    CHAR_SIGN
    CHAR_OTHER
)

func toCharType(ch byte) CharType {
    switch ch {
    case '0', '1', '2', '3', '4', '5', '6', '7', '8', '9':
        return CHAR_NUMBER
    case '+', '-':
        return CHAR_SIGN
    case ' ':
        return CHAR_SPACE
    default:
        return CHAR_OTHER
    }
}

func myAtoi(s string) int {
    transfer := map[State]map[CharType]State{
        STATE_START: map[CharType]State{
            CHAR_SPACE:STATE_START,
            CHAR_SIGN:STATE_SIGN,
            CHAR_NUMBER:STATE_NUMBER,
        },
        STATE_SIGN: map[CharType]State{
            CHAR_NUMBER:STATE_NUMBER,
        },
        STATE_NUMBER:map[CharType]State{
            CHAR_NUMBER:STATE_NUMBER,
        },
    }
    ans := 0
    sign := 1
    state := STATE_START
    for i := 0; i < len(s); i++ {
        typ := toCharType(s[i])
        if _, ok := transfer[state][typ]; !ok {
            return sign * ans
        } else {
            if transfer[state][typ] == STATE_SIGN && s[i] == '-' {
                sign = -1
            }
            if transfer[state][typ] == STATE_NUMBER {
                ans = ans * 10 + int(s[i] - '0')
                if sign == 1 && ans > math.MaxInt32 {
                    return math.MaxInt32
                }
                if sign == -1 && -ans < math.MinInt32 {
                    return math.MinInt32
                }
            }
            state = transfer[state][typ]
        }
    }
    return sign * ans
}
```
