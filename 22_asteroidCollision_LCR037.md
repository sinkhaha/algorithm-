# 题目

**LCR 037. 行星碰撞**

> 中等

给定一个整数数组 `asteroids`，表示在同一行的小行星。



对于数组中的每一个元素，其绝对值表示小行星的大小，正负表示小行星的移动方向（正表示向右移动，负表示向左移动）。每一颗小行星以相同的速度移动。



找出碰撞后剩下的所有小行星。碰撞规则：两个行星相互碰撞，较小的行星会爆炸。如果两颗行星大小相同，则两颗行星都会爆炸。两颗移动方向相同的行星，永远不会发生碰撞。

 

**示例 1：**

```
输入：asteroids = [5,10,-5]
输出：[5,10]
解释：10 和 -5 碰撞后只剩下 10 。 5 和 10 永远不会发生碰撞。
```

**示例 2：**

```
输入：asteroids = [8,-8]
输出：[]
解释：8 和 -8 碰撞后，两者都发生爆炸。
```

**示例 3：**

```
输入：asteroids = [10,2,-5]
输出：[10]
解释：2 和 -5 发生碰撞后剩下 -5 。10 和 -5 发生碰撞后剩下 10 。
```

**示例 4：**

```
输入：asteroids = [-2,-1,1,2]
输出：[-2,-1,1,2]
解释：-2 和 -1 向左移动，而 1 和 2 向右移动。 由于移动方向相同的行星不会发生碰撞，所以最终没有行星发生碰撞。 
```

 

**提示：**

- `2 <= asteroids.length <= 104`
- `-1000 <= asteroids[i] <= 1000`
- `asteroids[i] != 0`

> leetcode 地址 https://leetcode.cn/problems/XagZNi/description/

注意：本题与主站 735 题相同： https://leetcode-cn.com/problems/asteroid-collision/



# 思路

**碰撞的情况分析**

1、碰撞的情况：当两颗行星相向移动时，左边的行星向右移动，右边的行星向左移动，才会碰撞（它们可以表示为` [正数，负数]`）

2、不碰撞的情况：

* 两颗行星的移动方向相同，则不会发生碰撞（它们可以表示为 `[正数，正数]` 或  `[负数，负数]`）
* 两颗行星的移动方向相反，其中左边的行星向左移动，右边的行星向右移动，也不会发生碰撞（它们可以表示为` [负数，正数]`）



因为碰撞只会发生在相邻的行星之间，因此可以使用辅助栈存储未爆炸的行星，用栈模拟整个碰撞的过程。



**算法如下**

1、从左到右遍历数组 `asteroids`

2、如果当前元素是正数（即向右移动的行星），则将其入栈

3、如果当前元素是负数（即向左移动的行星）则需要考虑该行星可能和栈顶的行星发生碰撞，用一个 `curAlive` 表示当前元素是是否活着（即是否未爆炸），未爆炸的需要入栈 ，只有当栈顶的行星为正数时才会发生碰撞，即相邻元素是`[正数，负数]` 的序列，此时要继续比较它们的大小即可判断出碰撞后谁消失，如果当前元素消失，则不入栈即可，如果栈顶元素消失，则栈顶元素出栈即可，可能的情况如下：

* 当前的行星比栈顶的行星小，栈顶的行星不变，当前的行星爆炸不入栈，即 `curAlive` 为 `false`
* 当前行星比栈顶的行星大，栈顶的行星爆炸，所以将栈顶的行星出栈，即 `stack.pop()`，然后将当前的行星继续和新的栈顶的行星比较，重复上述操作，直到栈为空为止
* 当前的行星和栈顶的行星大小相同，两颗行星都会爆炸，将栈顶的行星出栈，即 `stack.pop()`，当前的行星爆炸不入栈，即 `curAlive` 为 `false`
* 最后判断当前的行星没有爆炸，即 `curAlive` 为 `true`，将当前的行星入栈

4、最后返回 `stack` 数组即可，这些是未爆炸的行星



# 代码

**rust 代码**

```rust
impl Solution {
 
    pub fn asteroid_collision(asteroids: Vec<i32>) -> Vec<i32> {
        let mut stack: Vec<i32> = vec![];

        for i in 0..asteroids.len() {
            let num = asteroids[i];

            if num > 0 {
                stack.push(num);
            } else {
                let mut cur_alive = true;
                // 此时顺序是[正，负]
                while !stack.is_empty() && *stack.last().unwrap() > 0 {
                    let left = *stack.last().unwrap();
                     // 正 > ｜负｜，此时负数的行星爆炸了，所以标识为不存活，也不入栈
                    if left > num.abs() {
                        cur_alive = false;
                        break;
                    } else if left < num.abs() {
                        //  正 < ｜负｜，栈顶的行星爆炸，继续循环拿当前元素跟栈顶元素比较
                        stack.pop();
                    } else {
                        // 正 == ｜负｜，两个都爆炸
                        stack.pop(); // 栈顶爆炸
                        cur_alive = false;
                        break;
                    }
                }

                if cur_alive {
                    stack.push(num);
                }
            }
        }

        stack
    }


}
```



**js 代码**

```js
/**
 * @param {number[]} asteroids
 * @return {number[]}
 */
var asteroidCollision = function(asteroids) {
    let stack = [];

    for (let num of asteroids) {
        if (num > 0) {
            stack.push(num);
        } else {
            let curAlive = true;
            
            // 此时顺序是[正，负]
            while (stack.length && stack[stack.length - 1] > 0) {
                let left = stack[stack.length - 1];
                // 正 > ｜负｜，此时负数的行星爆炸了，所以标识为不存活，也不入栈
                if (left > Math.abs(num)) {
                    curAlive = false;
                    break;
                } else if (left < Math.abs(num)) {
                    //  正 < ｜负｜，栈顶的行星爆炸，继续循环拿当前元素跟栈顶元素比较
                    stack.pop();                  
                } else {
                    // 正 == ｜负｜，两个都爆炸
                    stack.pop(); // 栈顶爆炸
                    curAlive = false;
                    break;
                }
            }

            if (curAlive) {
                stack.push(num);
            } 
        }
    }

    return stack;
};
```

# 复杂度

* 时间复杂度：`O(n)`
* 空间复杂度：`O(n)`