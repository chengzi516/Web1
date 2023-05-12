---
title: 【牛客】与时间相关的程序设计题目
date: 2023-05-11 16:29:02
tags: 
- 牛客网
- c++
categories:
- 刷题
ai: ture
cover: https://tuchuang-1317757279.cos.ap-chengdu.myqcloud.com/%E7%89%9B%E5%AE%A2%E7%BD%91.png
---

> 本篇一共有四道来自牛客网的关于日期计算的题目。

# :dog: 题1.HJ73 计算日期到天数转换

难度：`简单`
描述
根据输入的日期，计算是这一年的第几天。
保证年份为4位数且日期合法。

输入描述：
输入一行，每行空格分割，分别是年，月，日

输出描述：
输出是这一年的第几天
>示例1
输入：2012 12 31
输出：366
示例2
输入：1982 3 4
输出：63

本题考察日期的转换，难度简单。只需要从1月开始计算即可，需要注意的是`闰年的2月有29天`。

```c++
int main()
{
    int year,mon,day;
    int data[]={31,28,31,30,31,30,31,31,30,31,30,31};
    while(cin>>year>>mon>>day)
    {
        int sum;
        for(int i=0;i<mon-1;i++)
        {
              sum=sum+data[i];
        }
        if((year%400==0||(year%4==0&&year%100!=0))&&mon>2)
            sum=sum+1+day;
        else 
            sum=sum+day;
        cout<<sum<<endl;

    }
    return 0;
}
```

# :hamster: 题2.KY222 打印日期
难度：`较难`
描述
给出年分m和一年中的第n天，算出第n天是几月几号。
输入描述：
输入包括两个整数y(1<=y<=3000)，n(1<=n<=366)。
输出描述：
可能有多组测试数据，对于每组数据， 按 yyyy-mm-dd的格式将输入中对应的日期打印出来。
>示例1
输入：
2000 3
2000 31
2000 40
2000 60
2000 61
2001 60
输出：
2000-01-03
2000-01-31
2000-02-09
2000-02-29
2000-03-01
2001-03-01

在做日期相关的题目的时候，因为存在闰年与平年的干扰，所以常常考虑将两种年份的月历表分开来存储，然后通过bool返回值的隐式转换来巧取对应的月份。
例如：

```c++
int dayofmonth[2][13]={
     {0,31,28,31,30,31,30,31,31,30,31,30,31},
     {0,31,29,31,30,31,30,31,31,30,31,30,31}
 };

bool isleapyear(int year){            //是否为闰年
    return (year%4==0&&year%100!=0 )||(year%400==0);
}

```

比如我要取1994年的2月份，就可以这样写：
```c++
dayofmonth[isleapyear(1994)][2];
```
虽然isleapyear返回的是bool值，但c++支持这种`隐式的类型转换`,true则被转为了1，false则转为0。那么如果1994不是闰年，则返回0，也就调用了dayofmonth[0][2]的值，也就是28，同理是闰年则调用下一行的。这种巧用常常出现在日期类的题目中，运用十分灵活。

隐式类型转换在c++编程中还是很常见的。
例如：
```c++
A aa1(1);
A aa2=2;
```
这也是一种隐式类型转换，将2转换为A类型，再拷贝给aa2。
再比如：
```c++
void push(const String & str);
push("i'm chengzi");
```

那么有了以上的铺垫，本题也就很好解决了。
```c++
int main(){
    int year,month,day;
    int number=0;
    while(cin>>year>>number){
        month=0;
        int row=isleapyear(year);
        while(number>dayofmonth[row][month]){          //用number来递减确定确定月份
            number-=dayofmonth[row][month];
            month++;
        }
        day=number;                //不足月剩下的number确定日
        printf("%04d-%02d-%02d\n",year,month,day);
    }
    return 0;
}
```

# :mouse: 题3.KY258日期累加

难度：`中等`
描述
设计一个程序能计算一个日期加上若干天后是什么日期。
输入描述：
输入第一行表示样例个数m，接下来m行每行四个整数分别表示年月日和累加的天数。
输出描述：
输出m行，每行按yyyy-mm-dd的个数输出。
>示例1
输入：
1
2008 2 3 100
输出：
2008-05-13

题3是题2的一种变种，与题2不同的是，题3可以加的数字是可以大于365的。这里我使用的办法是：`number只要可以使得day位上的值足够向month位进1则循环，不够则加到day位上借宿循环即可`。

```c++
bool isleapyear(int y) {
    return y % 400 == 0 || (y % 4 == 0 && y % 100 != 0);
}

int print(int year, int month, int day, int number) {
    int dayofmonth[2][13] = {
        {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31},
        {0, 31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}
    };
     while(number){
        if(number>dayofmonth[isleapyear(year)][month]-day+1){
            number-= dayofmonth[isleapyear(year)][month]-day+1;
          month++;
          day=1;
        if(month>12){
            month=1;
            year++;
        }
        }else{
            day+=number;
            number=0;
        }
     }
   
    printf("%04d-%02d-%02d\n", year, month, day); // 按照指定格式输出
    return 0;
}

int main() {
    int m = 0;
    cin >> m;
    while (m) {
        int year, month, day, number;
        cin >> year >> month >> day >> number;
        print(year, month, day, number);
        m--;
    }
}
```


# :rabbit: 题4.KY111 日期差值

难度：`中等`
描述
有两个日期，求两个日期之间的天数，如果两个日期是连续的我们规定他们之间的天数为两天
输入描述：
有多组数据，每组数据有两行，分别表示两个日期，形式为YYYYMMDD
输出描述：
每组数据输出一行，即日期差值
>示例1
输入：
20110412
20110422
输出：
11

这里对于闰年平年的处理也可以采用上文提到的二维数组加bool，但我用的是函数的形式。
同时，题目输入的是年月日所组成的一个数字，所以我们可以考虑将其拆成三部分，这样求取二者差值就容易多了，首先将两个年份相减，再减去多余的月份，加上没考虑的月份即可。比如：20110120与20001230，先将2011与2000相减，得到11年（有闰有平，计算天数时要分开考虑），此时就多了一个1月20天，也少算了一个12月30日，分别加减即可。
```c++
//判断是否为闰年
bool isleapyear(int y){
  return y%400==0 || (y%4==0 && y%100 !=0);
}
//取当前月的天数
int monthDays(int year, int month) { // 每个月的天数
    int days[] = {0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
    if (month == 2 && isleapyear(year)) { // 闰年的二月有29天
        return 29;
    }
    return days[month];
}
int getdays(int date1,int date2){
   //使得date1最小
   if(date1>date2)
   swap(date1,date2);
   //将输入的数据转化为三个部分，年，月，日
   int year1=date1/10000, month1=date1%10000/100, day1=date1%100;
   int year2=date2/10000, month2=date2%10000/100, day2=date2%100;
   //days即为要返回的数据
   int days = 0;
   for (int i = year1; i < year2; i++) { // 统计整年的天数
        days += 365 + isleapyear(i); //bool隐式转换为int
   }

   for (int i = 1; i < month1; i++) { // 减去date1所在月之前的天数
        days -= monthDays(year1, i);
   }
   days -= day1; // 减去date1当月已经过去的天数

   for (int i = 1; i < month2; i++) { // 加上date2所在月之前的天数
        days += monthDays(year2, i);
   }
   days += day2; // 加上date2当月已经过去的天数

   days++;

   return days;

}
int main(){
    int a,b;
    cin>>a>>b;
    cout<<getdays(a,b)<<endl;
}
```