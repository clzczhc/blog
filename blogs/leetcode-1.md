---
title: LeetCode刷题(一)
date: 2022-02-28 00:13:26
categories: 算法
tags:
  - LeetCode
---

# LeetCode 刷题(一)

LeetCode 刷题，提升算法能力(dddd)

## 两数之和

[两数之和](https://leetcode-cn.com/problems/two-sum/)

初级版本( 时间复杂度`O(n2)` )

没什么要在意的，就是有种情况：nums=[3, 4, 5]， target=6，所以需要限制两次的下标不能相同

```js
var twoSum = function (nums, target) {
  for (let i = 0; i < nums.length; i++) {
    for (let j = 0; j < nums.length && i !== j; j++) {
      if (nums[i] + nums[j] === target) {
        return [i, j];
      }
    }
  }
};
```

**进阶版本**(时间复杂度小于 `O(n2)` )：

利用 map，把之前出现过的数字，以及下标存储起来，只需要遍历一次，如果有出现过符合条件的，直接返回结果，没有则继续存，继续遍历。

```js
var twoSum = function (nums, target) {
  let map = {}; // 用来存出现过的数字，其中键是出现过的数字，值是数组下标

  for (let i = 0; i < nums.length; i++) {
    let result = target - nums[i];
    if (map[result] !== undefined) {
      // 之前有出现过符合条件的
      return [map[result], i];
    } else {
      map[nums[i]] = i;
    }
  }
};
```

## 回文数

[回文数](https://leetcode-cn.com/problems/palindrome-number/)

初级版本：转化成字符串再判断

```js
var isPalindrome = function (x) {
  let str = x.toString(); // 转换成字符串
  let reverseStr = str.split("").reverse().join(""); // 先转换成数组，反转后再变回字符串

  if (str === reverseStr) {
    return true;
  }

  return false;
};
```

**进阶版本**：

- 定义一个除数 div，主要用于获取首位。

- 循环遍历 div 每次乘以 10，直到 x / div >= 10
  - 例如，x=1234，要获取首位，那么就需要先得到 1000，然后通过 1234 / 1000（整除），就能获取首位
- 通过循环，一直获取首尾，直到符合条件，或 x===0,即是回文数后退出循环

```js
var isPalindrome = function (x) {
  if (x < 0) {
    // 小于0直接返回
    return false;
  }

  let div = 1;
  while (x / div >= 10) {
    div *= 10; // 用于x / div来获取第一位
  }

  while (x > 0) {
    let left = Number.parseInt(x / div); // 获取第一位
    let right = x % 10; // 获取最后一位
    if (left !== right) {
      return false;
    }

    x = Number.parseInt((x % div) / 10); // 新的x去掉已经符合条件的首尾
    div = Number.parseInt(div / 100); // 新的x变化了，对应的div也要变化，除以100是因为x去掉了首尾
  }

  return true;
};
```

## 罗马数字转整数

[罗马数字转整数](https://leetcode-cn.com/problems/roman-to-integer/)

手动建一个 map，键是罗马数字，值是对应的整数，遍历一遍，遇到特殊情况，如 IX，则减，否则加

```js
var romanToInt = function (s) {
  let luoma = {
    I: 1,
    V: 5,
    X: 10,
    L: 50,
    C: 100,
    D: 500,
    M: 1000,
  };

  let fin = 0;
  let len = s.length;
  for (let i = 0; i < len; i++) {
    let left = luoma[s[i]];
    if (i + 1 < len && left < luoma[s[i + 1]]) {
      fin -= left; // 特殊情况是减
    } else {
      fin += left;
    }
  }

  return fin;
};
```

## 最长公共前缀

[最长公共前缀](https://leetcode-cn.com/problems/longest-common-prefix/)

解法较弱：通过两个循环，第一层循环获取字符串，第二层循环获取字符，而公共前缀一开始默认是第一个字符串，通过循环遍历，得到最长公共前缀

```js
var longestCommonPrefix = function (strs) {
  let common = strs[0];

  for (let i = 1; i < strs.length; i++) {
    common = common.slice(0, strs[i].length); // 用于排除第一个字符串较长，后面的字符串较短的情况
    for (let j = 0; j < strs[i].length; j++) {
      if (common[j] !== strs[i][j]) {
        common = common.slice(0, j);
      }
    }
  }
  return common;
};
```

## 有效的括号

[有效的括号](https://leetcode-cn.com/problems/valid-parentheses/)：先建立一个对象和一个空栈，键是右括号，值是左括号，然后遍历字符串。如果是左括号，则入栈。如果是右括号，则判断与栈顶括号是否是一对，是则出栈，否则直接返回 false，因为没有成功匹配。遍历完之后，如果栈不为空，即没有完全匹配，返回 false。

```js
var isValid = function (s) {
  let arr = [];
  let len = s.length;
  let pair = {
    ")": "(",
    "]": "[",
    "}": "{",
  };

  if (len % 2 !== 0) {
    return false;
  }

  for (let i = 0; i < s.length; i++) {
    if (s[i] === "(" || s[i] === "[" || s[i] === "{") {
      arr.push(s[i]);
    } else {
      if (pair[s[i]] === arr[arr.length - 1]) {
        arr.pop();
      } else {
        return false;
      }
    }
  }

  if (arr.length === 0) {
    return true;
  }

  return false;
};
```

## 实现 strStr

[实现 strStr](https://leetcode-cn.com/problems/implement-strstr/)

暴力解法：

```js
var strStr = function (haystack, needle) {
  let hLen = haystack.length;
  let nLen = needle.length;

  for (let i = 0; i + nLen <= hLen; i++) {
    // 不需要考虑后面的长度不够匹配需要的情况
    let flag = true; // 每一次从新的地方开始匹配，都需要重置flag为true
    for (let j = 0; j < nLen; j++) {
      if (haystack[i + j] !== needle[j]) {
        // 不匹配，直至跳出循环，并置flag为false
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
