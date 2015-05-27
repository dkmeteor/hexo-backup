title: 九度OJ 题目1384：二维数组中的查找
id: 104
categories:
  - Algorithm
date: 2014-09-24 02:52:48
tags:
---

http://ac.jobdu.com/problem.php?pid=1384

题目描述：在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

输入：输入可能包含多个测试样例，对于每个测试案例，
输入的第一行为两个整数m和n(1&lt;=m,n&lt;=1000)：代表将要输入的矩阵的行数和列数。
输入的第二行包括一个整数t(1&lt;=t&lt;=1000000)：代表要查找的数字。
接下来的m行，每行有n个数，代表题目所给出的m行n列的矩阵(矩阵如题目描述所示，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。

输出：对应每个测试案例，
输出”Yes”代表在二维数组中找到了数字t。
输出”No”代表在二维数组中没有找到数字t。

样例输入：
3 3
5
1 2 3
4 5 6
7 8 9
3 3
1
2 3 4
5 6 7
8 9 10
3 3
12
2 3 4
5 6 7
8 9 10

样例输出：
Yes
No
No

答疑：解题遇到问题?分享解题心得?讨论本题请访问：http://t.jobdu.com/thread-8107-1-1.html

AC Code:

#include &lt;stdio.h>

using namespace std;

int main()
{
    int rows,cols;
     while (scanf("%d %d", &amp;rows, &amp;cols) != EOF)
    {
        int find;
        scanf("%d", &amp;find);

        int ** arr = new int *[rows];
        for (int r = 0; r &lt; rows; r ++)
        {
            arr[r] = new int [cols];
            for (int c = 0; c &lt; cols; c ++)
            {
                scanf("%d", &amp;arr[r][c]);
            }
        }

        if(find &lt;arr[0][0] || find&gt;arr[rows-1][cols-1])
        {
            printf("%s", "No\n");
        }
        else
        {
            bool flag_find=0;
            for(int i=0;i&lt;rows &amp;&amp; !flag_find;i++)
            {
                if(find&lt;arr[i][0])
                {
                    for(int j=0;j&lt;cols &amp;&amp; !flag_find;j++)
                    {
                        if(find==arr[i-1][j])
                        {
                            flag_find=1;
                            break;
                        }
                    }
                    break;
                }
            }

            for(int j=0;j&lt;cols &amp;&amp; !flag_find;j++)
            {
                if(find==arr[rows-1][j])
                {
                    flag_find=1;
                }
            }

        if(flag_find)
        {
            printf("%s", "Yes\n");
        }
        else
         {
            printf("%s", "No\n");
         }
        }

        for(int j = 0; j &lt; rows; j++)
            delete arr[j];
        delete arr;

    }

    return 0;

}

/**************************************************************
    Problem: 1384
    User: dkmeteor
    Language: C++
    Result: Accepted
    Time:700 ms
    Memory:4852 kb
****************************************************************/

TODO:
第一次用OJ,用最简单的写法做个测试,等会更新下二分查找.