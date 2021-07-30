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
