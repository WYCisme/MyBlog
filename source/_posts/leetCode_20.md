---
title: leeCode刷题笔记	#文章标题
reward: false			#打赏开关
copyright: false		#版权声明
tags:					#标签
  - LeetCode
  - 有效的括号
categories:
  - 学习
  - 算法
date: 2020-03-23 19:24:40
---

# [有效的括号](https://leetcode-cn.com/problems/valid-parentheses/description/)

|  Category  |  Difficulty   | Likes | Dislikes |
| :--------: | :-----------: | :---: | :------: |
| algorithms | Easy (41.26%) | 1469  |    -     |

<details style="color: rgb(248, 248, 242); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe WPC&quot;, &quot;Segoe UI&quot;, system-ui, Ubuntu, &quot;Droid Sans&quot;, sans-serif, &quot;Microsoft Yahei UI&quot;; font-size: 14px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;"><summary><strong>Tags</strong></summary></details>

<details style="color: rgb(248, 248, 242); font-family: -apple-system, BlinkMacSystemFont, &quot;Segoe WPC&quot;, &quot;Segoe UI&quot;, system-ui, Ubuntu, &quot;Droid Sans&quot;, sans-serif, &quot;Microsoft Yahei UI&quot;; font-size: 14px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-style: initial; text-decoration-color: initial;"><summary><strong>Companies</strong></summary></details>

给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串，判断字符串是否有效。

有效字符串需满足：

1. 左括号必须用相同类型的右括号闭合。
2. 左括号必须以正确的顺序闭合。

注意空字符串可被认为是有效字符串。

**示例 1:**

```
输入: "()"
输出: true
```

**示例 2:**

```
输入: "()[]{}"
输出: true
```

**示例 3:**

```
输入: "(]"
输出: false
```

**示例 4:**

```
输入: "([)]"
输出: false
```

**示例 5:**

```
输入: "{[]}"
输出: true
```



### 代码：

```cpp
#include<vector>
#include<string>
using namespace std;
class Solution {
public:
    bool isValid(string s) {
        std::vector<char> as;
        for(char a : s){
            if (a == ')')
            {
                if (as.size()==0)
                {
                    return false;
                }
                if(as.back() == '('){
                    as.pop_back();

                }else{
                    return false;
                }
            }else
            if (a == ']')
            {
                if (as.size()==0)
                {
                    return false;
                }
                if(as.back() == '['){
                    as.pop_back();

                }else{
                    return false;
                }
            }else
            if (a == '}')
            {
                if (as.size()==0)
                {
                    return false;
                }
                if(as.back() == '{'){
                    as.pop_back();

                }else{
                    return false;
                }
            }else {
                as.push_back(a);
            }
        }
        if (as.size() == 0)
        {
            return true;
        }else{
            return false;
        }
    }
};
```

**结果**

<img src="https://gitee.com/wycisme/imageBed/raw/master/img/QQ截图20200812144948.png" alt="QQ截图20200812144948" style="zoom: 150%;" />