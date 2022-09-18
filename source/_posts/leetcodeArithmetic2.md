---
title: 力扣刷题总结（mid）
featured_image: ./images/antMan.jpeg
---

#### 两数相加 -- 链表

给你两个 非空 的链表，表示两个非负的整数。它们每位数字都是按照 逆序 的方式存储的，并且每个节点只能存储 一位 数字。

请你将两个数相加，并以相同形式返回一个表示和的链表。

你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var addTwoNumbers = function(l1, l2) {
    let head = null, tail = null, carry = 0;
    while(l1 || l2) {
        const n1 = l1 ? l1.val : 0;
        const n2 = l2 ? l2.val : 0;
        const sum = n1 + n2 + carry;
        if (!head) {
            head = tail = new ListNode(sum % 10);
        } else {
            tail.next = new ListNode(sum % 10);
            tail = tail.next;
        }
        carry = Math.floor(sum / 10);
        if (l1) l1 = l1.next;
        if (l2) l2 = l2.next;
    }
    if (carry > 0) tail.next = new ListNode(carry);
    return head;
};
```

---

#### 无重复字符的最长子串 -- 双指针

无重复字符的最长子串。

```js
/**
 * @param {string} s
 * @return {number}
 */
var lengthOfLongestSubstring = function(s) {
    let start = 0, end = 0;
    let map = new Map;
    let n = s.length; res = 0;
    while(end < n) {
        if (map.has(s[end])) {
            start = Math.max(map.get(s[end]), start);
        }
        res = Math.max(res, end - start + 1)
        map.set(s[end], end + 1)
        end++;
    }
    return res;
};
```

---

#### 最长回文子串 -- 双指针

给你一个字符串 s，找到 s 中最长的回文子串。

```js
/**
 * @param {string} s
 * @return {string}
 */
var longestPalindrome = function(s) {
    if (s.length < 2) return s;
    let max = ''
    const helper = (l, r) => {
        while(l >= 0 && r < s.length && s[l] === s[r]) {
            l--;
            r++;
        }
        let maxstr = s.slice(l + 1, r)
        max = maxstr.length > max.length ? maxstr : max;
    }
    for (let i = 0; i < s.length; i++) {
        helper(i, i);
        helper(i, i + 1);
    }
    return max;
};
```

---

#### 盛最多水的容器 -- 双指针

给定一个长度为 n 的整数数组 height 。有 n 条垂线，第 i 条线的两个端点是 (i, 0) 和 (i, height[i]) 。

找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

说明：你不能倾斜容器。

```js
/**
 * @param {number[]} height
 * @return {number}
 */
var maxArea = function(height) {
    let l = 0, r = height.length - 1, ans = 0;
    while (l < r) {
        let area = Math.min(height[l], height[r]) * (r - l);
        ans = Math.max(ans, area);
        if (height[l] < height[r]) {
            l++;
        } else {
            r--;
        }
    }
    return ans;
};
```

---

#### 三数之和 -- 双指针

给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有和为 0 且不重复的三元组。

注意：答案中不可以包含重复的三元组。

```js
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var threeSum = function(nums) {
    let ans = [];
    if (nums?.length < 3) return ans;
    nums.sort((a, b) => a - b);
    for (let i = 0; i < nums.length; i++) {
        if (nums[i] > 0) break;
        if (i > 0 && nums[i] === nums[i - 1]) continue;
        let l = i + 1, r = nums.length - 1;
        while(l < r) {
            const sum = nums[l] + nums[r] + nums[i];
            if (sum === 0) {
                ans.push([nums[i], nums[l], nums[r]]);
                while (l < r && nums[l] === nums[l + 1]) l++;
                while(l < r && nums[r] === nums[r - 1]) r--;
                l++;
                r--;
            }
            else if (sum > 0) r--;
            else if (sum < 0) l++;
        }
    }
    return ans;
};
```

---

#### 电话号码的字母组合 -- 回溯

给定一个仅包含数字 2-9 的字符串，返回所有它能表示的字母组合。答案可以按 任意顺序 返回。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

```js
/**
 * @param {string} digits
 * @return {string[]}
 */
var letterCombinations = function(digits) {
    if (!digits) return [];
    let res = [];
    let map = new Map([
        ['2','abc'],
        ['3','def'],
        ['4', 'ghi'],
        ['5', 'jkl'],
        ['6', 'mno'],
        ['7', 'pqrs'],
        ['8', 'tuv'],
        ['9', 'wxyz']
    ]);
    const backTracking = (cur, digits) => {
        if (!digits) res.push(cur);
        else {
            let str = map.get(digits[0]);
            for (let i = 0; i < str.length; i++) {
                backTracking(cur + str[i], digits.slice(1))
            }
        }
    };
    backTracking('', digits);
    return res;
    
};

```

---

#### 删除链表的倒数第 N 个结点 -- 快慢指针

给你一个链表，删除链表的倒数第 n 个结点，并且返回链表的头结点。

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val, next) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.next = (next===undefined ? null : next)
 * }
 */
/**
 * @param {ListNode} head
 * @param {number} n
 * @return {ListNode}
 */
var removeNthFromEnd = function(head, n) {
    let dummy = new ListNode(0, head);
    let slow = dummy;
    let fast = head;
    while(n > 0) {
        fast = fast.next;
        n--;
    }
    while(fast) {
        fast = fast.next;
        slow = slow.next;
    }
    slow.next = slow.next.next;
    return dummy.next;
};
```

---

#### 括号生成 -- 回溯

数字 n 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且 有效的 括号组合。

```js
/**
 * @param {number} n
 * @return {string[]}
 */
var generateParenthesis = function(n) {
    let res = [];
    if (n <= 0) return res;

    const findParenthesis = (str, l, r) => {
        if (l === 0 && r === 0) {
            res.push(str);
            return;
        }
        if (l === r) {
            findParenthesis(str + '(', l - 1, r);
        } else {
            if (l > 0) findParenthesis(str + '(', l - 1, r);
            findParenthesis(str + ')', l, r - 1);
        }
    }
    findParenthesis('', n, n);
    return res;
};
```

---

#### 下一个排列 -- 指针

整数数组的一个 排列  就是将其所有成员以序列或线性顺序排列。

例如，arr = [1,2,3] ，以下这些都可以视作 arr 的排列：[1,2,3]、[1,3,2]、[3,1,2]、[2,3,1] 。
整数数组的 下一个排列 是指其整数的下一个字典序更大的排列。更正式地，如果数组的所有排列根据其字典顺序从小到大排列在一个容器中，那么数组的 下一个排列 就是在这个有序容器中排在它后面的那个排列。如果不存在下一个更大的排列，那么这个数组必须重排为字典序最小的排列（即，其元素按升序排列）。

例如，arr = [1,2,3] 的下一个排列是 [1,3,2] 。
类似地，arr = [2,3,1] 的下一个排列是 [3,1,2] 。
而 arr = [3,2,1] 的下一个排列是 [1,2,3] ，因为 [3,2,1] 不存在一个字典序更大的排列。
给你一个整数数组 nums ，找出 nums 的下一个排列。

必须 原地 修改，只允许使用额外常数空间。

思路
如何变大：从低位挑一个大一点的数，交换前面一个小一点的数。
变大的幅度要尽量小。
像 [3,2,1] 这样递减的，没有下一个排列，已经稳定了，没法变大。
像 [1,5,2,4,3,2] 这种，怎么稍微变大？

从右往左，寻找第一个比右邻居小的数，把它换到后面去

“第一个”意味着尽量是低位，“比右邻居小”意味着它是从右往左的第一个波谷
比如，1 5 (2) 4 3 2，中间这个 2。

接着还是从右往左，寻找第一个比这个 2 大的数。15 (2) 4 (3) 2，交换后：15 (3) 4 (2) 2。

还没结束！变大的幅度可以再小一点，仟位微变大了，后三位可以再小一点。

后三位肯定是递减的，翻转，变成[1,5,3,2,2,4]，即为所求。

```js
/**
 * @param {number[]} nums
 * @return {void} Do not return anything, modify nums in-place instead.
 */
var nextPermutation = function(nums) {
    if (nums.length <= 1) return nums;
    let i = nums.length - 2;
    while (i >= 0 && nums[i] >= nums[i + 1]) {
        i--;
    }
    if (i >= 0) {
        let j = nums.length - 1;
        while(j >= 0 && nums[j] <= nums[i]) {
            j--;
        }
        [nums[i], nums[j]] = [nums[j], nums[i]];
    }
    let l = i + 1, r = nums.length - 1;
    while (l < r) {
        [nums[l], nums[r]] = [nums[r], nums[l]];
        l++;
        r--;
    }
};
```

---

#### 搜索旋转排序数组 -- 二分搜索

整数数组 nums 按升序排列，数组中的值 互不相同 。

在传递给函数之前，nums 在预先未知的某个下标 k（0 <= k < nums.length）上进行了 旋转，使数组变为 [nums[k], nums[k+1], ..., nums[n-1], nums[0], nums[1], ..., nums[k-1]]（下标 从 0 开始 计数）。例如， [0,1,2,4,5,6,7] 在下标 3 处经旋转后可能变为 [4,5,6,7,0,1,2] 。

给你 旋转后 的数组 nums 和一个整数 target ，如果 nums 中存在这个目标值 target ，则返回它的下标，否则返回 -1 。

你必须设计一个时间复杂度为 O(log n) 的算法解决此问题。

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number}
 */
var search = function(nums, target) {
    if (nums.length === 0) return -1;
    if (nums.length === 1) {
        return nums[0] === target ? 0 : -1;
    }
    let l = 0; r = nums.length - 1;
    while(l <= r) {
        let mid = Math.floor((r + l) / 2);
        if (nums[mid] === target) return mid;
        if (nums[l] <= nums[mid]) {
            if (nums[l] <= target && target < nums[mid]) {
                r = mid - 1;
            } else {
                l = mid + 1;
            }
        } else {
            if (nums[mid] < target && target <= nums[r] ) {
                l = mid + 1;
            } else {
                r = mid - 1;
            }
        }
    }
    return -1;
};
```

---

#### 在排序数组中查找元素的第一个和最后一个位置 -- 二分查找

给你一个按照非递减顺序排列的整数数组 nums，和一个目标值 target。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 target，返回 [-1, -1]。

你必须设计并实现时间复杂度为 O(log n) 的算法解决此问题。

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[]}
 */
var searchRange = function(nums, target) {
    const n = nums.length - 1;
    if (nums.length === 0) return [-1, -1];
    if (nums[0] === target && nums[n] === target) return [0, n];

    const findIndex = (nums, target, first) => {
        let l = 0, r = n, ans = -1;
        while(l <= r) {
            let mid = Math.floor((l + r) / 2);
            if (nums[mid] === target) {
                ans = mid;
                if (first) {
                    r = mid - 1;
                } else {
                    l = mid + 1;
                }
            } else if (nums[mid] > target) {
                r = mid - 1;
            } else {
                l = mid + 1;
            }
        }
        return ans;
    }
    return [findIndex(nums, target, true), findIndex(nums, target, false)];
};
```

---

#### 组合总和 -- 回溯(未剪枝)

给你一个 无重复元素 的整数数组 candidates 和一个目标整数 target ，找出 candidates 中可以使数字和为目标数 target 的 所有 不同组合 ，并以列表形式返回。你可以按 任意顺序 返回这些组合。

candidates 中的 同一个 数字可以 无限制重复被选取 。如果至少一个数字的被选数量不同，则两种组合是不同的。 

对于给定的输入，保证和为 target 的不同组合数少于 150 个。

```js
/**
 * @param {number[]} candidates
 * @param {number} target
 * @return {number[][]}
 */
var combinationSum = function(candidates, target) {
    if (candidates.length === 0) return [];
    let ans = [];
    const dfs = (combine, target, idx) => {
        if (idx === candidates.length) return;
        if (target === 0) {
            ans.push(combine);
            return;
        }
        dfs(combine, target, idx + 1);
        if (target - candidates[idx] >= 0) {
            dfs([...combine, candidates[idx]], target - candidates[idx], idx);
        }
    };
    dfs([], target, 0);
    return ans;
};
```

---

#### 全排列 -- dfs + 回溯

给定一个不含重复数字的数组 nums ，返回其 所有可能的全排列 。你可以 按任意顺序 返回答案。

```js
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var permute = function(nums) {
    if (nums.length === 0) return [];
    let ans = [];
    const arrange = (path, used) => {
        if (path.length === nums.length) {
            ans.push(path.slice());
            return;
        };
        for (let i = 0; i < nums.length; i++) {
            if (used[i]) {
                continue;
            };
            path.push(nums[i]);
            used[i] = true;
            arrange(path, used);
            path.pop();
            used[i] = false;
        }
    }
    arrange([], []);
    return ans;
};
```

---

#### 字母异位词分组 -- 哈希

```js
/**
 * @param {string[]} strs
 * @return {string[][]}
 */
var groupAnagrams = function(strs) {
    let map = new Map();
    if (strs.length === 0) return [];
    for (let str of strs) {
        let arr = str.split("");
        let key = arr.sort().toString();
        let list = map.has(key) ? map.get(key) : new Array;
        list.push(str);
        map.set(key, list);
    }
    return Array.from(map.values());
};
```

---

#### 最大子序和 -- 动态规划

给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

```js
  /**
  * @param {number[]} nums
  * @return {number}
  */
  var maxSubArray = function(nums) {
    let pre = 0, maxAns = nums[0];
    nums.forEach((x) => {
      pre = Math.max(pre + x, x);
      maxAns = Math.max(maxAns, pre);
    });
    return maxAns;
  };
```

---

#### 跳跃游戏 -- 贪心

给定一个非负整数数组 nums ，你最初位于数组的 第一个下标 。

数组中的每个元素代表你在该位置可以跳跃的最大长度。

判断你是否能够到达最后一个下标。

```js
/**
 * @param {number[]} nums
 * @return {boolean}
 */
var canJump = function(nums) {
    let n = nums.length;
    let reach = 0;
    for (let i = 0; i <= reach && reach < n - 1; i++) {
        reach = Math.max(nums[i] + i, reach);
    }
    return reach >= n - 1;
};
```

```js
/**
 * @param {number[]} nums
 * @return {boolean}
 */
var canJump = function(nums) {
    let n = nums.length, last = n - 1;
    for(let i = n - 2; i >= 0; i--) {
        if (i + nums[i] >= last) last = i;
    }
    return last === 0;
};
```

---

#### 合并区间 -- 排序

以数组 intervals 表示若干个区间的集合，其中单个区间为 intervals[i] = [starti, endi] 。请你合并所有重叠的区间，并返回 一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间 。

```js
/**
 * @param {number[][]} intervals
 * @return {number[][]}
 */
var merge = function(intervals) {
    let n = intervals.length;
    if (n === 0) return [];
    intervals.sort((a, b) => a[0] - b[0]);
    let ans = []
    for (let i = 0; i < n; i++) {
        let m = ans.length;
        if (m === 0 || ans[m - 1][1] < intervals[i][0]) {
            ans.push(intervals[i]);
        } else {
            ans[m - 1][1] = Math.max(ans[m - 1][1], intervals[i][1])
        }
    }
    return ans;
};
```

---

#### 不同路径 -- 动态规划

一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

```js
/**
 * @param {number} m
 * @param {number} n
 * @return {number}
 */
var uniquePaths = function(m, n) {
    let dp = new Array(m).fill(0).map(() => new Array(n).fill(0));
    for (let i = 0; i < m; i++) {
        dp[i][0] = 1;
    }
    for (let i = 0; i < n; i++) {
        dp[0][i] = 1;
    }
    for (let i = 1; i < m; i++) {
        for (let j = 1; j < n; j++) {
            dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
        }
    }
    return dp[m - 1][n - 1];
};
```

---

#### 最小路径和 -- 动态规划

给定一个包含非负整数的 m x n 网格 grid ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小。

说明：每次只能向下或者向右移动一步。

```js
/**
 * @param {number[][]} grid
 * @return {number}
 */
var minPathSum = function(grid) {
    if (grid.length === 0) return 0;
    let m = grid.length, n = grid[0].length;
    let dp = new Array(m).fill(0).map(() => new Array(n).fill(0));
    dp[0][0] = grid[0][0];
    for (let i = 1; i < m; i++) {
        dp[i][0] = dp[i - 1][0] + grid[i][0];
    }
    for (let i = 1; i < n; i++) {
        dp[0][i] = dp[0][i - 1] + grid[0][i];
    }
    for (let i = 1; i < m; i++) {
        for (let j = 1; j < n; j++) {
            dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + grid[i][j];
        }
    }
    return dp[m - 1][n - 1];
};
```

---

#### 颜色分类 -- 双指针

给定一个包含红色、白色和蓝色、共 n 个元素的数组 nums ，原地对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。

我们使用整数 0、 1 和 2 分别表示红色、白色和蓝色。

必须在不使用库的sort函数的情况下解决这个问题。

```js
/**
 * @param {number[]} nums
 * @return {void} Do not return anything, modify nums in-place instead.
 */
var sortColors = function(nums) {
    let n = nums.length;
    if (n < 2) return nums;
    let p = 0, i = 0; q = n - 1;
    while (i <= q) {
        if (nums[i] === 0) {
            nums[i] = nums[p];
            nums[p] = 0;
            p++;
        } else if (nums[i] === 2) {
            nums[i] = nums[q];
            nums[q] = 2;
            q--;
            if (nums[i] !== 1) i--;
        };
        i++;
    }
    return nums;
};
```

---

#### 子集 -- 回溯

给你一个整数数组 nums ，数组中的元素 互不相同 。返回该数组所有可能的子集（幂集）。

解集 不能 包含重复的子集。你可以按 任意顺序 返回解集。

```js
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
var subsets = function(nums) {
    if (nums.length === 0) return [];
    let ans = [];
    const backtrack = (path, m) => {
        ans.push(path.slice());
        for (let i = m; i < nums.length; i++) {
            path.push(nums[i]);
            backtrack(path, i + 1);
            path.pop();
        }
    }
    backtrack([], 0);
    return ans;
};
```

---

#### 不同的二叉搜索树 -- 动态规划

给你一个整数 n ，求恰由 n 个节点组成且节点值从 1 到 n 互不相同的 二叉搜索树 有多少种？返回满足题意的二叉搜索树的种数。

```js
/**
 * @param {number} n
 * @return {number}
 */
var numTrees = function(n) {
    if (n <= 1) return 1;
    let dp = new Array(n + 1).fill(0);
    dp[0] = 1;
    dp[1] = 1;
    for (let i = 2; i <= n; i++) {
        for (let j = 1; j <= i; j++) {
            dp[i] += dp[j - 1] * dp[i - j]
        }
    }
    return dp[n];
};
```

---

#### 验证二叉搜索树 -- 递归

给你一个二叉树的根节点 root ，判断其是否是一个有效的二叉搜索树。

有效 二叉搜索树定义如下：

节点的左子树只包含 小于 当前节点的数。
节点的右子树只包含 大于 当前节点的数。
所有左子树和右子树自身必须也是二叉搜索树。

```rust
// Definition for a binary tree node.
// #[derive(Debug, PartialEq, Eq)]
// pub struct TreeNode {
//   pub val: i32,
//   pub left: Option<Rc<RefCell<TreeNode>>>,
//   pub right: Option<Rc<RefCell<TreeNode>>>,
// }
//
// impl TreeNode {
//   #[inline]
//   pub fn new(val: i32) -> Self {
//     TreeNode {
//       val,
//       left: None,
//       right: None
//     }
//   }
// }
use std::rc::Rc;
use std::cell::RefCell;
impl Solution {
    pub fn helper(root: &Option<Rc<RefCell<TreeNode>>>, lower: i64, upper: i64) -> bool {
        match root {
            None => true,
            Some(node) => {
                if node.borrow().val as i64 <= lower || node.borrow().val as i64 >= upper {
                    false
                } else {
                    Solution::helper(&node.borrow().left, lower, node.borrow().val as i64) &&
                    Solution::helper(&node.borrow().right, node.borrow().val as i64, upper)
                }
            }
        }
    }
    pub fn is_valid_bst(root: Option<Rc<RefCell<TreeNode>>>) -> bool {
        Solution::helper(&root, i64::min_value(), i64::max_value())
    }
}
```

---

#### 二叉树的层序遍历 -- bfs

给你二叉树的根节点 root ，返回其节点值的 层序遍历 。 （即逐层地，从左到右访问所有节点）。

```rust
// Definition for a binary tree node.
// #[derive(Debug, PartialEq, Eq)]
// pub struct TreeNode {
//   pub val: i32,
//   pub left: Option<Rc<RefCell<TreeNode>>>,
//   pub right: Option<Rc<RefCell<TreeNode>>>,
// }
//
// impl TreeNode {
//   #[inline]
//   pub fn new(val: i32) -> Self {
//     TreeNode {
//       val,
//       left: None,
//       right: None
//     }
//   }
// }
use std::rc::Rc;
use std::cell::RefCell;
impl Solution {
    pub fn level_order(root: Option<Rc<RefCell<TreeNode>>>) -> Vec<Vec<i32>> {
        let mut ans = Vec::new();
        let mut levels = Vec::new();
        if root.is_none() {
            return ans;
        }
        levels.push(root.unwrap());
        while levels.is_empty()!= true {
            let mut cur = Vec::new();
            for i in 0..levels.len() {
                let mut node = levels.remove(0);
                cur.push(node.borrow_mut().val);
                if node.borrow_mut().left.is_some() {
                    levels.push(node.borrow_mut().left.take().unwrap());
                };
                if node.borrow_mut().right.is_some() {
                    levels.push(node.borrow_mut().right.take().unwrap());
                };
            }
            ans.push(cur);
        };
        ans
    }
}
```

---

#### 从前序与中序遍历序列构造二叉树 -- 递归

给定两个整数数组 preorder 和 inorder ，其中 preorder 是二叉树的先序遍历， inorder 是同一棵树的中序遍历，请构造二叉树并返回其根节点。

```rust
// Definition for a binary tree node.
// #[derive(Debug, PartialEq, Eq)]
// pub struct TreeNode {
//   pub val: i32,
//   pub left: Option<Rc<RefCell<TreeNode>>>,
//   pub right: Option<Rc<RefCell<TreeNode>>>,
// }
//
// impl TreeNode {
//   #[inline]
//   pub fn new(val: i32) -> Self {
//     TreeNode {
//       val,
//       left: None,
//       right: None
//     }
//   }
// }
use std::rc::Rc;
use std::cell::RefCell;
use std::collections::HashMap;
impl Solution {
    pub fn build_tree(preorder: Vec<i32>, inorder: Vec<i32>) -> Option<Rc<RefCell<TreeNode>>> {
        let preLen = preorder.len();
        let inLen = inorder.len();
        if preLen != inLen {
            panic!("Incorrect");
        };
        let mut map = HashMap::new();
        for i in 0..inLen {
            map.insert(inorder[i], i);
        };
        Solution::helper(&preorder, 0, preLen - 1, &map, 0, inLen - 1)
    }
    pub fn helper(preorder: &Vec<i32>, preleft: usize, preRight: usize, map: &HashMap<i32, usize>,
                  inLeft: usize, inRight: usize) -> Option<Rc<RefCell<tree_node::TreeNode>>> {
        if preleft > preRight || inLeft > inRight {
            return None
        }
        let mut rootVal = &preorder[preleft];
        let mut root = TreeNode::new(*rootVal);
        let pIndex = map.get(rootVal).unwrap();
        root.left = Solution::helper(&preorder, preleft + 1, pIndex - inLeft + preleft, map, inLeft, pIndex - 1);
        root.right = Solution::helper(&preorder, pIndex - inLeft + preleft + 1, preRight, map, pIndex + 1, inRight);
        Some(Rc::new(RefCell::new(root)))
    }
}
```

---

#### 最长连续序列 -- 哈希

给定一个未排序的整数数组 nums ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

请你设计并实现时间复杂度为 O(n) 的算法解决此问题。

```rust
use std::collections::HashSet;
impl Solution {
    pub fn longest_consecutive(nums: Vec<i32>) -> i32 {
        if nums.len() == 0 {
            return 0;
        };
        let mut num_set = HashSet::new();
        for num in nums {
            num_set.insert(num);
        }
        let mut longest_len = 0;
        for &num in num_set.iter() {
            if !num_set.contains(&(num - 1)) {
                let mut current_num = num;
                let mut current_len = 1;
                while (num_set.contains(&(current_num + 1))) {
                    current_num += 1;
                    current_len += 1;
                }
                longest_len = std::cmp::max(longest_len, current_len);
            }
        }
        longest_len
    }
}
```