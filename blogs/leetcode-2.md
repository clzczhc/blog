---
title: LeetCode刷题(二)    JavaScript
date: 2022-03-06 20:39:16
categories: 算法
tags:
  - LeetCode
---

# LeetCode 刷题(二) JavaScript

## [合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/description/)

通过递归实现：判断 list1 和 list2 那个链表的头结点的值更小，然后递归下去决定下一个添加到结果的节点，当两个链表中有为空时，递归结束。

```js
var mergeTwoLists = function (list1, list2) {
  if (list1 === null) {
    return list2;
  } else if (list2 === null) {
    return list1;
  } else if (list1.val < list2.val) {
    list1.next = mergeTwoLists(list1.next, list2);
    return list1;
  } else {
    list2.next = mergeTwoLists(list1, list2.next);
    return list2;
  }
};
```

## [删除有序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/description/)

### 1. 简单版本

遍历一遍，通过` indexOf`和` lastIndexOf`来判断是否有重复项，有的话，则不相等。然后通过` splice`删除掉重复项，删除后，因为少一位了，此时的索引也需要-1

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var removeDuplicates = function (nums) {
  for (let i = 0; i < nums.length; i++) {
    if (nums.indexOf(nums[i]) !== nums.lastIndexOf(nums[i])) {
      nums.splice(i, 1);
      i--;
    }
  }

  return nums.length;
};
```

### 2. 双指针法

定义两个指针 fast 和 slow，起始都为 1(因为 0 时不可能会有重复)，其中 fast 指针一直在走，而当` nums[fast]`不等于` nums[fast - 1]`时，即是不重复项，那么此时就让` nums[slow] = nums[fast]`，即存下不重复项，` slow`指针才继续走一步，遍历完后，`slow`就是删掉重复项后数组的长度。

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var removeDuplicates = function (nums) {
  let n = nums.length;
  if (n === 0) {
    return 0;
  }

  let fast = 1;
  let slow = 1;

  while (fast < n) {
    if (nums[fast] !== nums[fast - 1]) {
      nums[slow] = nums[fast];
      slow++;
    }
    fast++;
  }

  console.log(slow);

  return slow;
};
```

## [移除元素](https://leetcode-cn.com/problems/remove-element/description/)

### 1. 双指针法

和上面的做法一样，

```js
var removeElement = function (nums, val) {
  const n = nums.length;
  let left = 0;
  for (let right = 0; right < n; right++) {
    if (nums[right] !== val) {
      nums[left] = nums[right]; // 不等于要移除的元素，则存起来，等于则不存
      left++;
    }
  }

  return left;
};
```

### 2. 双指针优化

做法和双指针法类似，不同的是，初始时，左指针指向 0，右指针指向数组最后一位。

通过判断 left 指针指向的元素等不等于 val

- 等于的话，则把 right 指针指向的元素赋值给 left 指针，然后 right--，继续判断 left 指针指向的元素等不等于 left
- 不等于的话，则 left 指针前进

```js
/**
 * @param {number[]} nums
 * @param {number} val
 * @return {number}
 */
var removeElement = function (nums, val) {
  let left = 0,
    right = nums.length;

  while (left < right) {
    if (nums[left] === val) {
      nums[left] = nums[right - 1]; // 把后面的元素放到前面，避免重复赋值。如1,2,3,4,5要去掉1
      right--;
    } else {
      left++;
    }
  }

  return left;
};
```

<b style="color: red">可以避免元素的重复赋值操作</b>：

假设数组为` [1, 2, 3, 4]`，要删除的元素为 1

- 使用普通的双指针法：第一位等于要删的元素，跳过，后面一次赋值` nums[0]=2`，` nums[1]=3`，` nums[2]=4`
- 使用优化的双指针法：第一轮需要赋值` nums[0]=4`，后面就不需要在赋值了

## [实现 strStr()](https://leetcode-cn.com/problems/implement-strstr/description/)

### 1. 暴力匹配

就是直接嵌套循环，时间复杂度较大。KMP 算法比较麻烦，先留个坑(希望之后会补坑)

```js
/**
 * @param {string} haystack
 * @param {string} needle
 * @return {number}
 */
var strStr = function (haystack, needle) {
  let hLen = haystack.length;
  let nLen = needle.length;

  for (let i = 0; i + nLen <= hLen; i++) {
    let flag = true;
    for (let j = 0; j < nLen; j++) {
      if (haystack[i + j] !== needle[j]) {
        flag = false;
        break;
      }
    }
    if (flag) {
      return i;
    }
  }

  return -1;
};
```

## [搜索插入位置](https://leetcode-cn.com/problems/search-insert-position/description/)

有序插入：二分法：就是每次都取在中间的那个值，如果`target`小于等于中间那个值的话，就是说` target`在左半边，此时，让 right=mid-1，大于则让` left=mid+1`。最后返回` right+1`。（这里返回` right+1`的原因是`target`小于等于中间那个值都会继续往左移，所以`right`停的位置会比`target`小，那么插的位置就是` right+1`了）

```js
var searchInsert = function (nums, target) {
  let left = 0, right = nums.length - 1
  let ans = nums.length

  while (left <= right) {
    let mid = right - Number.parseInt((right - left) / 2)

    if (target <= nums[mid]) {
      right = mid - 1
    } else {
      left = mid + 1
    }
  }
  return right + 1
```
