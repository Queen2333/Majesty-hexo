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
