---
layout:     post
title:      实现strStr()--KMP
subtitle:   leetcode刷题记录
date:       2018-09-21
author:     BY kylin
header-img: img/post-bg-cook.jpg
catalog: true
tags:
    - leetcode
    - 算法
    - 字符串
---
实现strStr()--KMP

题目

实现 strStr() 函数。

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。



思路

暴力可解，用KMP的话是一道模板题。

从大二开始看KMP，理解一次忘一次。又重新理解了一遍，手敲了一遍代码，写下来自己的理解，希望能记住吧。(:з)∠)



代码

    class Solution {
    public:
        void getNext(vector<int> &next,const string needle)
        {
            int lp=next.size();
            int k=-1;
            int j=0;
            next[0]=-1;
            while(j<lp-1)
            {
                if(k==-1||needle[j]==needle[k])
                {
                    if(needle[++j]==needle[++k])
                    {
                        next[j]=next[k];
                    }
                    else
                    {
                        next[j]=k;
                    }
                }
                else
                {
                    k=next[k];
                }
            }
        }
        int strStr(string haystack, string needle) {
            int ls=haystack.length();
            int lp=needle.length();
            if(lp==0)
                return 0;
            int i=0;
            int j=0;
            vector<int> next(lp,0);
            getNext(next,needle);
            while(i<ls && j<lp)
            {
                if(j==-1 || haystack[i]==needle[j])
                {
                    i++;
                    j++;
                }
                else
                {
                    j=next[j];
                }
            }
            
            if(j>=lp)
                return i-lp;
            else
                return -1;
        }
    };



KMP

暴力匹配的规律



如图，S为待匹配串，P为模式串，i,j分别指向S,P当前的位置。

暴力匹配方案中，若当前字符匹配失败，i,j需要分别向前回溯j个字符，整个算法是O(mn)的。那么有没有方法可以通过之前匹配得到的信息，减少匹配的字符次数呢。

如图所示，在i=j发生不匹配时，存在两种情况：

1. 下次可以匹配到的位置是i。
2. 下次可以匹配到的位置在i前，设它为i-k。

 

 











其中，情况1如上图，意味着，s串中，[i-j,i-1]的部分，都无法与p串匹配。如图，此时我们可以直接把i不动，j移动到0。因为我们已经知道，p串的前三个字符是匹配的，而在p[0,j]部分中，只有p[0]是能与s[i]匹配到的'A'字符。换言之，因为匹配过前面三个字符，我们可以得到的信息是：



 





情况2如下图，意味着，s[i-k,i-1]和p[0,k]的部分是匹配的，也就是图中的AB两个字符。试想，如果从i-k开始匹配，最终也会匹配到上图的情况，也就是说，可以直接从j=k,i不动的状态开始再次匹配。(其中细节可以再一步论证，这里不细说了)。



这两种情况可以归结为一种，即P串存在一个j前的指针k，指向的位置使得p[0,k-1]和s[i-k,i-1]的部分匹配。这样我们可以直接从j=k,i=i的位置开始匹配，省去了s串i前j个字母逐个匹配一遍。



这个结论可以推倒一蛤：

设P中存在一个指针k，使得：

- p[0,k-1]=p[j-k,j-1]

当s[i]!=p[j]时，我们已知如下结论:

- s[i-j,i-1]=j[0,j-1]

- s[i-k,i-1]=p[j-k,j-1]

可以得到：

- p[0,k-1]=s[i-k,i-1]
  

这意味着，S中i指针往前的k个字符，和P中的开头k个字符，是完全匹配的！再次匹配的时候，就可以固定i指针不动，P串则直接从k位置开始匹配。

我们使用一个next数组来表示它，即：next[j]=k表示，当P[j] != S[i]时，j指针的下一步移动位置。 也就是说，P串的前几个字符，能够和S串i位置之前匹配上。



如何计算next数组 

我们要找的是，P串中j位置，和从头往后数到第几个字符的字串能够完全匹配上。让我们分三部分解读代码:

1. 初始化部分

因为next指向的字串一定在j前，所以初始化j=0，k=-1，让他们始终错开一位匹配。next[0]设为-1，表示串首没有可以匹配的字串了。

2.匹配部分

若p[k]==p[j]，说明字串当前字符能够匹配，则j,k++。这点很好理解。

另外一个判断条件k==-1，意味着是否从头开始匹配。若是，则next[++j]=++k=0，即把指针指向串首。

此外，多判断了p[++j]是否等于p[++k]。

 

如图所示，若j是从串最后移动到当前=1的位置，即移动前：j=3,k=1，可以发现，p[1]==p[3]，而由于j=3时已经判断过不匹配，那么移动到1的位置也没有意义，可以再向前跳一步。

1. 不匹配部分
   寻找next数组可以看作：[0,k]的字串和[j-k,j]的字串是否匹配。而当p[++j]不于p[++k]时，问题转做[0,k]中有没有字串和[j-k,j]能匹配上，k=next[k]将k挪到k匹配到的更小字串的位置，这个操作可以看作k指针向前“跳跃”，不断寻找能够匹配的更短字串，直到找到的过程。



 

 


