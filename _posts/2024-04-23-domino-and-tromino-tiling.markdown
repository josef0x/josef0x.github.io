---
layout: post
title:  "790. Domino and Tromino Tiling"
date:   2022-12-25 21:26:54 +0200
# categories: dynamic-programming combinatorics recursion
---
In this blog post, we will discuss a nice counting problem - which is not hard [^1] - that I stumbled upon while doing Leetcode daily problems.

[^1]: depending on how comfortable you are with counting, Leetcode considers this <a href="https://leetcode.com/problems/domino-and-tromino-tiling/description/">medium</a>.

In brief, you are given two tiles, a $1 \times 2$ tile (i.e. a domino), and an L-shaped tile which they refer to as a "tromino", the goal is to find (or count) the number of ways of tiling a $2 \times n$ board, since this number gets very big when $n$ is large, the answer is expected to be modulo $10^9 + 7$. The full problem description can be found <a href="https://leetcode.com/problems/domino-and-tromino-tiling/description/">here</a>.

### Preliminaries

Before we go into more complicated cases, let us take a simple case, that of a board of size $2 \times 3$ for example and try to solve it manually to see if there is anything we can learn from such a small instance that would allow us to generalize.

{:refdef: style="text-align: center;"}
![](/images/board.svg)
{: refdef}
*Figure 1: A $2 \times 3$ board, a domino and a tromino*
{: refdef}

The board above ($n = 3$) can be tiled in five ways, these are enumerated below.

![](/images/3-board_tiling.svg)
{: refdef}
*Figure 2: Valid tilings for a $2 \times 3$ board (w/ colors are there for easy visualization)* 
{: refdef}

I really encourage you to grab a pencil and a piece of paper and make sure that these are the only possible ways to tile a $2 \times 3$ board.

For $n = 4$, there are 11 ways:

![](/images/4-board_tiling.svg)
{: refdef}
*Figure 3: Possible tilings of a $2 \times 4$ board - 11 in total* 
{: refdef}
### A bit of maths

Now, if we let $T_k$ be the number of ways of tiling a $2 \times k$ board, we can think of $T_k$ in terms of smaller instances (i.e. in terms of $T_{k-i}$ where $i \in 1, \dots, k-1$).

This leads us to the following recursive formula:
$T_k = T_{k-1} + T_{k-2} + 2 \times T_{k-3} + 2 \times T_{k-4} + \dots + 2 \times T_1 :(1)$

$\space \space \space \space = T_{k-1} + T_{k-2} + 2 \times \sum_{i=3}^{k-1}T_{k-i} :(2)$

If you don't get how equation $(1)$ was obtained, the following figures could help visualizing it.

There is only one way of tiling a board of $2 \times 1$ so, $T_1 = 1$
(we don't consider the 180-degree rotation of the domino a different way).
If we were to abstract away the numbers by hiding them inside the $T_{k}$'s, and just count the number of ways using elementary counting rules, we get $T_1 \times T_{k-1}$

![](/images/formula_1-1.svg)
{: refdef}

Next, as you may know $T_2 = 2$, but since all the configurations starting with a vertical domino were counted in the previous expression (i.e. $T_1 \times T_{k-1}$) we should subtract 1 from $T_2$ to avoid counting it twice.
This means $1 \times T_{k-2}$

![](/images/formula_1-2.svg)
{: refdef}
The factor of $2$ that is in the 3rd term of equation $(1)$ (i.e. $2 \times T_{k-3}$) comes from the following configurations:

![](/images/formula_1-3.svg)
{: refdef}
If you still don't see why, recall the solutions for $n=3$ (see figure 2) using the same trick (or observation if you want), all configurations starting with a vertical and horizontal dominos are already counted (not colored on the figure) the only ones that are not yet counted are the ones starting with trominos. Since trominos can start in two possible ways (figure 2 again) we only need to multiply by $2$ the rest of possible configurations $T_{k-3}$.

I will let you continue the same process until you reach the last $2 \times 1$ block of the board, which would look like:

![](/images/formula_1-last.svg)
{: refdef}
And I will let you figure out why it is $2 \times T_1$


If we replace $k$ by $k-1$ in equation $(1)$, we get:
$T_{k-1} = T_{k-2} + T_{k-3} + 2 \times T_{k-4} + 2 \times T_{k-5} + \dots + 2 \times T_1$

Hence, $T_k = T_{k-1} + T_{k-3} + T_{k-1}$ and consequently $ T_k = 2 \times T_{k-1} + T_{k-3}$

At this stage, we have done most of the work and the rest is merely a *straightforward* translation of the last equation into code.

In order to avoid recomputing the $T_k$'s each time, we could do a simple memoization thanks to a simple array.

### Solution
Below is an implementation in C++ of the result we got from the previous section. We assume $T_0 = 1$ (by extension). `cache` is a vector (resizable array) where smaller results are saved.

{% highlight c++ %}
class Solution {
public:
    int numTilings(int n)
    {
        if (n == 1 || n == 2)
            return n;
        
        vector<int> cache(n+1);
        cache[0] = 1;
        cache[1] = 1;
        cache[2] = 2;

        for (int i = 3; i <= n ; i++)
        {
            cache[i] = (((2*cache[i-1]) % 1000000007) + cache[i-3]) % 1000000007;
        }

        return cache[n];
    }
};
{% endhighlight %}
