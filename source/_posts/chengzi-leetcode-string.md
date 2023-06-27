---
title: 【leetcode】string的相关oj
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/1610173947-QjrHlC-LeetCode.png
tags:
- 刷题
- c语言
date: 2023-06-06 17:45:18
categories: 
- 刷题
ai: ture
---

# :movie_camera:字符串相加

给定两个字符串形式的非负整数 num1 和num2 ，计算它们的和并同样以字符串形式返回。

你不能使用任何內建的用于处理大整数的库（比如 BigInteger）， 也不能直接将输入的字符串转换为整数形式。

 

>示例 1：
输入：num1 = "11", num2 = "123"
输出："134"
示例 2：
输入：num1 = "456", num2 = "77"
输出："533"
示例 3：
输入：num1 = "0", num2 = "0"
输出："0"

链接：https://leetcode.cn/problems/add-strings
思路：模拟简单的加法计算。设置`value1，value2，以及carry(进位)`三个变量来实现此过程。
代码：
```c++
class Solution {
public:
    string addStrings(string num1, string num2) {
        int end1=num1.size()-1;
        int end2=num2.size()-1;  //两数加法计算是从末尾开始的。注意size取值和末尾下标的大小差异！
        string news;
        int value1=0,value2=0,carry=0;
        while(end1>=0||end2>=0){ 
                 if(end1>=0)
                  value1=num1[end1--]-'0'; //一定要减'0'，才能将string的数字转为int的数字。
                  else
                   value1=0;
                 if(end2>=0)
                 value2=num2[end2--]-'0';
                 else 
                 value2=0;
                 int sum=value1+value2+carry;
                 if(sum>9){
                     carry=1;
                     sum-=10;
                 }else{
                    carry=0;
                 }
                 news+=sum+'0';
        }
        if(carry==1)  //最高位的carry可能会被遗忘！
           news+='1';
        reverse(news.begin(),news.end());
        return news;
    }
};
```
关于为何 value1=num1[end1--]-'0'中要减去一个'0':
在C++中，字符类型和数字类型之间存在一个ASCII码的`映射关系`。例如，字符'0'的ASCII码值是`48`，字符'1'的ASCII码值是`49`，以此类推。在字符串中，每个数字字符都对应着一个ASCII码值。例如，字符'0'对应着ASCII码值48，字符'1'对应着ASCII码值49，以此类推。当将一个数字字符转换为数字类型时，需要将其对应的ASCII码值减去字符'0'的ASCII码值，才能得到数字类型的值。例如，字符'1'对应的ASCII码值是49，减去字符'0'的ASCII码值48后，得到数字类型的值1。

# :movie_camera:反转字符串
编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 s 的形式给出。

不要给另外的数组分配额外的空间，你必须`原地修改`输入数组、使用 O(1) 的额外空间解决这一问题。

 
>示例 1：
输入：s = ["h","e","l","l","o"]
输出：["o","l","l","e","h"]
示例 2：
输入：s = ["H","a","n","n","a","h"]
输出：["h","a","n","n","a","H"]

链接：https://leetcode.cn/problems/reverse-string

思路：简单的双指针问题。

代码：
```c++
class Solution {
public:
    void reverseString(vector<char>& s) {
      int l=0;
      int r=s.size()-1;
      //判空
      if(s.empty())
        return;
      while(l<r){
          int tmp =s[l];
          s[l]=s[r];
          s[r]=tmp;
          l++;
          r--;
      }  
    }
};
```


# :movie_camera:字符串中的第一个唯一字符

给定一个字符串 s ，找到 它的第一个`不重复的字符`，并返回它的`索引` 。如果不存在，则返回 -1 。

 

>示例 1：
输入: s = "leetcode"
输出: 0
示例 2:
输入: s = "loveleetcode"
输出: 2
示例 3:
输入: s = "aabb"
输出: -1

链接：https://leetcode.cn/problems/first-unique-character-in-a-string
代码：
```c++
class Solution {
public:
    int firstUniqChar(string s) {
       //ASCII码表中只有256个字符，所以可以将count数组的大小设置为256。这样就可以将每个字符映射到count数组的一个位置上
       int count[256]={0};
       int size=s.size();
       for(int i=0;i<size;i++)
         count[s[i]]+=1;
       for(int i=0;i<size;i++){
           if(count[s[i]]==1)
            return i;
       }
       return -1;
    }
};
```




# :movie_camera: 反转字符串的单词

给定一个字符串 s ，你需要反转字符串中每个单词的字符顺序，同时仍保留空格和单词的初始顺序。

 
>示例 1：
输入：s = "Let's take LeetCode contest"
输出："s'teL ekat edoCteeL tsetnoc"
示例 2:
输入： s = "God Ding"
输出："doG gniD"

思路：使用两个for循环，第一个循环确定需要反转单词的最后一个坐标（需要减1），第二个循环用于进行反转。j是从需要反转的单词的第一位开始往后，循环操作至单词中间即可。需要注意循环的控制条件`(k+i-1)/2`以及swap的坐标控制。

```c++
class Solution {
public:
    string reverseWords(string s) {
        int k=0;
        for(int i=0;i<=s.length();i++){
            if(s[i]==' '||i==s.length()){
               for(int j=k;j<=(k+i-1)/2;j++){
                   swap(s[j],s[i-1-j+k]);
               }
               k=i+1;
            }
        }        
        return s;
    }
   
};

```