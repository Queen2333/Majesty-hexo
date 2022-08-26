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
