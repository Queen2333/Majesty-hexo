---
title: 力扣刷题总结
featured_image: ./images/spiderman.jpg
---

#### 两数之和 -- 哈希映射

给定一个整数数组 nums  和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那   两个   整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

```
  /**
  * @param {number[]} nums
  * @param {number} target
  * @return {number[]}
  */
  var twoSum = function(nums, target) {
    if (!nums) return

    let obj = new Map()
    for (let i = 0; i < nums.length; i++) {
      let part = target - nums[i]
      if (obj.get(part) !== undefined) {
          return [obj.get(part), i]
      }
      obj.set(nums[i], i)
    }
  };
```

---

#### 最长公共前缀 -- 链表

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 ""。

```
  /**
  * @param {string[]} strs
  * @return {string}
  */
  var longestCommonPrefix = function(strs) {
    if (strs.length === 0) return ''
    let ans = strs[0];
    for(let i =1;i<strs.length;i++) {
      let j
      for(j = 0; j<ans.length && j < strs[i].length; j++) {
        if(ans[j] != strs[i][j]) break;
      }
      ans = ans.substr(0, j);
      if(ans === '') return ans;
    }
    return ans
  };
```

---

#### 有效的括号 -- 栈

给定一个只包括 '('，')'，'{'，'}'，'['，']'  的字符串 s ，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。

```
  /**
 * @param {string} s
 * @return {boolean}
 */
  var isValid = function(s) {
    const n = s.length
    if (n % 2 !== 0) return false
    const map = new Map([
      [')', '('],
      [']', '['],
      ['}', '{']
    ])
    const stk = []
    s.split('').forEach(item => {
      if (map.has(item)) {
        if (map.get(item) !== stk[stk.length - 1] || !stk.length) {
          return false
        }
        stk.pop()
      } else {
        stk.push(item)
      }
    })
    return !stk.length
  };
```

---

#### 合并两个有序链表 -- 链表 递归

将两个升序链表合并为一个新的 升序 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。

```
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
  var mergeTwoLists = function(l1, l2) {
    if (l1 === null) return l2
    if (l2 === null) return l1
    if (l1.val < l2.val) {
      l1.next = mergeTwoLists(l1.next, l2)
      return l1
    } else {
      l2.next = mergeTwoLists(l1, l2.next)
      return l2
    }
  };
```

---

#### 删除有序数组中的重复项 -- 双指针(快慢指针)

给你一个有序数组 nums ，请你 原地 删除重复出现的元素，使每个元素 只出现一次 ，返回删除后数组的新长度。

不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。

说明:

为什么返回数值是整数，但输出的答案是数组呢?

请注意，输入数组是以「引用」方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下:

```
  // nums 是以“引用”方式传递的。也就是说，不对实参做任何拷贝
  int len = removeDuplicates(nums);

  // 在函数里修改输入数组对于调用者是可见的。
  // 根据你的函数返回的长度, 它会打印出数组中 该长度范围内 的所有元素。
  for (int i = 0; i < len; i++) {
      print(nums[i]);
  }
```

```
  var removeDuplicates = function(nums) {
    const n = nums.length;
    if (n === 0) {
      return 0;
    }
    let fast = 1, slow = 1;
    while (fast < n) {
      if (nums[fast] !== nums[fast - 1]) {
        nums[slow] = nums[fast];
        ++slow;
      }
      ++fast;
    }
    return slow;
  };

```

---

#### 移除元素 -- 双指针

给你一个数组 nums  和一个值 val，你需要 原地 移除所有数值等于  val  的元素，并返回移除后数组的新长度。

不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

```
  /**
  * @param {number[]} nums
  * @param {number} val
  * @return {number}
  */
  var removeElement = function(nums, val) {
    let n = nums.length
    let i = 0
    while (i < n) {
      if (val === nums[i]) {
        nums[i] = nums[n - 1]
        n--
      } else {
        i++
      }
    }
    return n
  };
```

---

#### 实现 strStr() -- 双指针

实现  strStr()  函数。

给你两个字符串  haystack 和 needle ，请你在 haystack 字符串中找出 needle 字符串出现的第一个位置（下标从 0 开始）。如果不存在，则返回   -1 。

```
  /**
  * @param {string} haystack
  * @param {string} needle
  * @return {number}
  */
  var strStr = function(haystack, needle) {
    let l = 0, n = 0
    while (l < haystack.length && n < needle.length) {
      if (haystack[l + n] === needle[n]) {
        n++
      } else {
        n = 0
        l++
      }
    }
    return n === needle.length ? l : -1
  };
```

---

#### 搜索插入位置 -- 二分法

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 O(log n) 的算法。

```
  /**
  * @param {number[]} nums
  * @param {number} target
  * @return {number}
  */
  var searchInsert = function(nums, target) {
    const n = nums.length;
    let left = 0, right = n - 1, ans = n;
    while (left <= right) {
      let mid = ((right - left) / 2) + left;
      if (target <= nums[mid]) {
        ans = mid;
        right = mid - 1;
      } else {
        left = mid + 1;
      }
    }
    return ans;
  };
```

---

#### 最大子序和 -- 动态规划

给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

```
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

#### 最后一个单词的长度 -- 遍历字符串

给你一个字符串 s，由若干单词组成，单词前后用一些空格字符隔开。返回字符串中最后一个单词的长度。

单词 是指仅由字母组成、不包含任何空格字符的最大子字符串。

```
  /**
  * @param {string} s
  * @return {number}
  */
  var lengthOfLastWord = function(s) {
    let end = s.length - 1
    while(end >= 0 && s[end] === ' ') end --
    if (end < 0) return 0
    let start = end
    while(start >= 0 && s[start] !== ' ') start --
    return end - start
  };
```

---

#### 加一 -- 数组遍历

给定一个由 整数 组成的 非空 数组所表示的非负整数，在该数的基础上加一。

最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。

你可以假设除了整数 0 之外，这个整数不会以零开头。

```
  /**
  * @param {number[]} digits
  * @return {number[]}
  */
  var plusOne = function(digits) {
    const len = digits.length;
    for(let i = len - 1; i >= 0; i--) {
      digits[i]++;
      digits[i] %= 10;
      if(digits[i]!=0)
        return digits;
    }
    digits = [...Array(len + 1)].map(_=>0);;
    digits[0] = 1;
    return digits;
  };
```

---

#### x 的平方根 -- 二分法

实现  int sqrt(int x)  函数。

计算并返回  x  的平方根，其中  x 是非负整数。

由于返回类型是整数，结果只保留整数的部分，小数部分将被舍去。

```
  /**
  * @param {number} x
  * @return {number}
  */
  var mySqrt = function(x) {
    let left = 0, right = x
    while(left <= right) {
      let mid = (left + right) >> 1
      if (mid * mid < x) {
        left = mid + 1
      } else if (mid * mid > x) {
        right = mid - 1
      } else {
        return mid
      }
    }
    return right
  };
```

---

#### 爬楼梯

假设你正在爬楼梯。需要 n 阶你才能到达楼顶。

每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

注意：给定 n 是一个正整数。

```
  /**
  * @param {number} n
  * @return {number}
  */
  var climbStairs = function(n) {
    let p = 0, q = 0, r = 1
    for(let i = 0; i < n; i++) {
      p = q
      q = r
      r = p + q
    }
    return r
  };
```

---

#### 删除排序链表中的重复元素 -- 链表

存在一个按升序排列的链表，给你这个链表的头节点 head ，请你删除所有重复的元素，使每个元素 只出现一次 。

返回同样按升序排列的结果链表。

```
  /**
  * Definition for singly-linked list.
  * function ListNode(val, next) {
  *     this.val = (val===undefined ? 0 : val)
  *     this.next = (next===undefined ? null : next)
  * }
  */
  /**
  * @param {ListNode} head
  * @return {ListNode}
  */
  var deleteDuplicates = function(head) {
    if (!head) return head
    let cur = head
    while (cur.next) {
      if (cur.val === cur.next.val) {
        cur.next = cur.next.next
      } else {
        cur = cur.next
      }
    }
    return head
  };
```

---

#### 合并两个有序数组 -- 双指针

给你两个按 非递减顺序 排列的整数数组  nums1 和 nums2，另有两个整数 m 和 n ，分别表示 nums1 和 nums2 中的元素数目。

请你 合并 nums2 到 nums1 中，使合并后的数组同样按 非递减顺序 排列。

注意：最终，合并后数组不应由函数返回，而是存储在数组 nums1 中。为了应对这种情况，nums1 的初始长度为 m + n，其中前 m 个元素表示应合并的元素，后 n 个元素为 0 ，应忽略。nums2 的长度为 n 。

```
  /**
  * @param {number[]} nums1
  * @param {number} m
  * @param {number[]} nums2
  * @param {number} n
  * @return {void} Do not return anything, modify nums1 in-place instead.
  */
  var merge = function(nums1, m, nums2, n) {
    const sorted = new Array(m + n).fill(0)
    let p1 = 0, p2 = 0
    var cur
    while(p1 < m || p2 < n) {
      if (p1 === m) {
        cur = nums2[p2++]
      } else if (p2 === n) {
        cur = nums1[p1++]
      } else if (nums1[p1] < nums2[p2]) {
        cur = nums1[p1++]
      } else {
        cur = nums2[p2++]
      }
      sorted[p1 + p2 - 1] = cur
    }
    for (let i = 0; i < m + n; i++) {
      nums1[i] = sorted[i]
    }
  };
```

---

#### 二叉树的中序遍历 -- 递归

给定一个二叉树的根节点 root ，返回它的 中序 遍历。

二叉树的中序遍历：按照访问左子树——根节点——右子树的方式遍历这棵树，而在访问左子树或者右子树的时候我们按照同样的方式遍历，直到遍历完整棵树。

```
  /**
  * Definition for a binary tree node.
  * function TreeNode(val, left, right) {
  *     this.val = (val===undefined ? 0 : val)
  *     this.left = (left===undefined ? null : left)
  *     this.right = (right===undefined ? null : right)
  * }
  */
  /**
  * @param {TreeNode} root
  * @return {number[]}
  */
  var inorderTraversal = function(root) {
    let result = []
    this.middleOrder = (root) => {
      if (!root) return
      middleOrder(root.left)
      result.push(root.val)
      middleOrder(root.right)
    }
    middleOrder(root)
    return result
  };
```

---

#### 二叉树的前序遍历 -- 递归

给你二叉树的根节点 root ，返回它节点值的 前序 遍历。

二叉树的前序遍历：按照访问根节点——左子树——右子树的方式遍历这棵树，而在访问左子树或者右子树的时候，我们按照同样的方式遍历，直到遍历完整棵树。

```
  /**
  * Definition for a binary tree node.
  * function TreeNode(val, left, right) {
  *     this.val = (val===undefined ? 0 : val)
  *     this.left = (left===undefined ? null : left)
  *     this.right = (right===undefined ? null : right)
  * }
  */
  /**
  * @param {TreeNode} root
  * @return {number[]}
  */
  var preorderTraversal = function(root) {
    let result = []
    this.preOrder = (root) => {
      if (root === null) return
      result.push(root.val)
      preOrder(root.left)
      preOrder(root.right)
    }
    preOrder(root)
    return result
  };
```

---

#### 二叉树的后序遍历 -- 递归

给定一个二叉树，返回它的 后序 遍历。

二叉树的后序遍历：按照访问左子树——右子树——根节点的方式遍历这棵树，而在访问左子树或者右子树的时候，我们按照同样的方式遍历，直到遍历完整棵树。

```
  /**
  * Definition for a binary tree node.
  * function TreeNode(val, left, right) {
  *     this.val = (val===undefined ? 0 : val)
  *     this.left = (left===undefined ? null : left)
  *     this.right = (right===undefined ? null : right)
  * }
  */
  /**
  * @param {TreeNode} root
  * @return {number[]}
  */
  var postorderTraversal = function(root) {
    let result = []
    this.lastOrder = (root) => {
      if (root === null) return
      lastOrder(root.left)
      lastOrder(root.right)
      result.push(root.val)
    }
    lastOrder(root)
    return result
  };
```

---

#### 相同的树 -- 递归(深度优先遍历)

给你两棵二叉树的根节点 p 和 q ，编写一个函数来检验这两棵树是否相同。

如果两个树在结构上相同，并且节点具有相同的值，则认为它们是相同的。

```
  /**
  * Definition for a binary tree node.
  * function TreeNode(val, left, right) {
  *     this.val = (val===undefined ? 0 : val)
  *     this.left = (left===undefined ? null : left)
  *     this.right = (right===undefined ? null : right)
  * }
  */
  /**
  * @param {TreeNode} p
  * @param {TreeNode} q
  * @return {boolean}
  */
  var isSameTree = function(p, q) {
    if (p === null && q === null) {
      return true
    } else if (p === null || q === null) {
      return false
    } else if (p.val !== q.val) {
      return false
    } else {
      return isSameTree(p.left, q.left) && isSameTree(p.right, q.right)
    }
  };
```

---

#### 对称二叉树 -- 递归

给定一个二叉树，检查它是否是镜像对称的。

```
  /**
  * Definition for a binary tree node.
  * function TreeNode(val, left, right) {
  *     this.val = (val===undefined ? 0 : val)
  *     this.left = (left===undefined ? null : left)
  *     this.right = (right===undefined ? null : right)
  * }
  */
  /**
  * @param {TreeNode} root
  * @return {boolean}
  */
  var isSymmetric = function(root) {
    return check(root, root)
  };
  const check = (p, q) => {
    if (!p && !q) return true
    if (!p || !q) return false
    return p.val === q.val && check(p.left, q.right) && check(p.right, q.left)
  }
```

---

#### 二叉树的最小深度 -- 递归

给定一个二叉树，找出其最小深度。

最小深度是从根节点到最近叶子节点的最短路径上的节点数量。

说明：叶子节点是指没有子节点的节点。

```
  /**
  * Definition for a binary tree node.
  * function TreeNode(val, left, right) {
  *     this.val = (val===undefined ? 0 : val)
  *     this.left = (left===undefined ? null : left)
  *     this.right = (right===undefined ? null : right)
  * }
  */
  /**
  * @param {TreeNode} root
  * @return {number}
  */
  var minDepth = function(root) {
    if (!root) return 0
    if (!root.left) return minDepth(root.right) + 1
    if (!root.right) return minDepth(root.left) + 1
    return (Math.min(minDepth(root.left), minDepth(root.right))) + 1
  };
```

---

#### 二叉树的最大深度 -- 递归

给定一个二叉树，找出其最大深度。

二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。

说明: 叶子节点是指没有子节点的节点。

```
  /**
  * Definition for a binary tree node.
  * function TreeNode(val, left, right) {
  *     this.val = (val===undefined ? 0 : val)
  *     this.left = (left===undefined ? null : left)
  *     this.right = (right===undefined ? null : right)
  * }
  */
  /**
  * @param {TreeNode} root
  * @return {number}
  */
  var maxDepth = function(root) {
    if (!root) {
      return 0
    } else {
      let left = maxDepth(root.left)
      let right = maxDepth(root.right)
      return Math.max(left, right) + 1
    }
  };
```

---

#### 将有序数组转换为二叉搜索树 -- 递归，二分法

给你一个整数数组 nums ，其中元素已经按 升序 排列，请你将其转换为一棵 高度平衡 二叉搜索树。

高度平衡 二叉树是一棵满足「每个节点的左右两个子树的高度差的绝对值不超过 1 」的二叉树。

```
  /**
  * Definition for a binary tree node.
  * function TreeNode(val, left, right) {
  *     this.val = (val===undefined ? 0 : val)
  *     this.left = (left===undefined ? null : left)
  *     this.right = (right===undefined ? null : right)
  * }
  */
  /**
  * @param {number[]} nums
  * @return {TreeNode}
  */
  var sortedArrayToBST = function (nums) {
    if (!nums.length) return null;
    const mid = nums.length >>> 1;
    let cur = new TreeNode(nums[mid]);
    cur.left = sortedArrayToBST(nums.slice(0, mid));
    cur.right = sortedArrayToBST(nums.slice(mid+1));
    return cur;
  };

```

---

#### 平衡二叉树 -- 递归

给定一个二叉树，判断它是否是高度平衡的二叉树。

本题中，一棵高度平衡二叉树定义为：

一个二叉树每个节点 的左右两个子树的高度差的绝对值不超过 1 。

```
  var isBalanced = function (root) {
  if(!root) return true
  return Math.abs(depth(root.left) - depth(root.right)) <= 1
    && isBalanced(root.left)
    && isBalanced(root.right)
  }
  var depth = function (node) {
    if(!node) return -1
    return 1 + Math.max(depth(node.left), depth(node.right))
  }
```

---

#### 路径总和 -- 递归

给你二叉树的根节点  root 和一个表示目标和的整数  targetSum ，判断该树中是否存在 根节点到叶子节点 的路径，这条路径上所有节点值相加等于目标和  targetSum 。

叶子节点 是指没有子节点的节点。

```
  /**
  * Definition for a binary tree node.
  * function TreeNode(val, left, right) {
  *     this.val = (val===undefined ? 0 : val)
  *     this.left = (left===undefined ? null : left)
  *     this.right = (right===undefined ? null : right)
  * }
  */
  /**
  * @param {TreeNode} root
  * @param {number} targetSum
  * @return {boolean}
  */
  var hasPathSum = function(root, targetSum) {
    if (!root) return false
    if (!root.left && !root.right) return root.val === targetSum
    return hasPathSum(root.left, targetSum - root.val) || hasPathSum(root.right,   targetSum - root.val)
  };
```

---

#### 杨辉三角 -- 数学

给定一个非负整数 numRows，生成「杨辉三角」的前 numRows 行。

在「杨辉三角」中，每个数是它左上方和右上方的数的和。

```
  /**
  * @param {number} numRows
  * @return {number[][]}
  */
  var generate = function(numRows) {
    let ret = []
    for(let i = 0; i < numRows; i++) {
      const row = new Array(i + 1).fill(1)
      for (let j = 1; j < row.length - 1; j++) {
        row[j] = ret[i - 1][j - 1] + ret[i - 1][j]
      }
      ret.push(row)
    }
    return ret
  };
```

---

#### 买卖股票的最佳时机 -- 遍历

给定一个数组 prices ，它的第  i 个元素  prices[i] 表示一支给定股票第 i 天的价格。

你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回 0 。

```
  /**
  * @param {number[]} prices
  * @return {number}
  */
  var maxProfit = function(prices) {
    let res = 0
    let buy = Number.MAX_VALUE
    for (let price of prices) {
      buy = Math.min(buy, price)
      res = Math.max(res, price - buy)
    }
    return res
  };
```

---

#### 验证回文串 -- 双指针

给定一个字符串，验证它是否是回文串，只考虑字母和数字字符，可以忽略字母的大小写。

说明：本题中，我们将空字符串定义为有效的回文串

```
  /**
  * @param {string} s
  * @return {boolean}
  */
  var isPalindrome = function(s) {
    s = s.toLowerCase();
    const reg = /^[0-9a-z]*$/;
    let left = 0
    let right = s.length - 1
    while(left < right) {
      if (!reg.test(s[left])) {
        left++
        continue
      }
      if (!reg.test(s[right])) {
        right--
        continue
      }
      if (s[left] === s[right]) {
        left++
        right--
      } else {
        return false
      }
    }
    return true
  };
```

---

#### 只出现一次的数字 -- 位运算

给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

```
  /**
  * @param {number[]} nums
  * @return {number}
  */
  var singleNumber = function(nums) {
    let a = 0
    for (const item of nums) {
      a ^= item
    }
    return a
  };
```

---

#### 环形链表 -- 哈希/快慢指针

给定一个链表，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。

如果链表中存在环，则返回 true 。 否则，返回 false 。

```
  /**
  * Definition for singly-linked list.
  * function ListNode(val) {
  *     this.val = val;
  *     this.next = null;
  * }
  */

  /**
  * @param {ListNode} head
  * @return {boolean}
  */
  var hasCycle = function(head) {
    let obj = new Map()
    while(head) {
      if (obj.has(head)) return true
      obj.set(head, true)
      head = head.next
    }
    return false
  };
```

```
  /**
  * Definition for singly-linked list.
  * function ListNode(val) {
  *     this.val = val;
  *     this.next = null;
  * }
  */

  /**
  * @param {ListNode} head
  * @return {boolean}
  */
  var hasCycle = function(head) {
    let fast = head
    let slow = head
    while(fast) {
      if (fast.next === null) return false
      slow = slow.next
      fast = fast.next.next
      if (slow === fast) return true
    }
    return false
  };
```
