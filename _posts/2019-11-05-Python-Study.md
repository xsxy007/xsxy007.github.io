---
layout: post
title: Python-Study
date: 2019-11-05
tags: python
---

> python 学习

## python 100例练习

1、题目：有四个数字：1、2、3、4，能组成多少个互不相同且无重复数字的三位数？各是多少？

```python
for i in range(1,5):
    for j in range(1,5):
        for k in range(1,5):
            if (i != j) and (j != k) and (k != i):
                print(i,j,k)
```

2、企业发放的奖金根据利润提成。利润(I)低于或等于10万元时，奖金可提10%；利润高于10万元，低于20万元时，低于10万元的部分按10%提成，高于10万元的部分，可提成7.5%；20万到40万之间时，高于20万元的部分，可提成5%；40万到60万之间时高于40万元的部分，可提成3%；60万到100万之间时，高于60万元的部分，可提成1.5%，高于100万元时，超过100万元的部分按1%提成，从键盘输入当月利润I，求应发放奖金总数？

```python
i = int(input("输入净利润："))
arr = [1000000, 600000, 400000, 200000, 100000, 0]
rat = [0.01, 0.015, 0.03, 0.05, 0.075, 0.1]
r = 0
for index in range(1, 6):
    if i > arr[index]:
        r += (i - arr[index]) * rat[index]
        # print((i - arr[index]) * rat[index])
        i = arr[index]

print(r)
```

3、一个整数，它加上100后是一个完全平方数，再加上168又是一个完全平方数，请问该数是多少？

```python
for i in range(1,85):
    if 168 % i == 0:
        j = 168 / i
        if  i > j and (i + j) % 2 == 0 and (i - j) % 2 == 0 :
            m = (i + j) / 2
            n = (i - j) / 2
            x = n * n - 100
            print(int(x))
```

4、输入某年某月某日，判断这一天是这一年的第几天？

```python
year = int(input('year:\n'))
month = int(input('month:\n'))
day = int(input('day:\n'))

months = (0,31,59,90,120,151,181,212,243,273,304,334)
sum = 0

if 0 < month <= 12:
    sum += months[month - 1]
else:
    print('error')

sum += day

leap = 0
if (year % 400 == 0) or ((year % 4 ==0) and (year % 100 == 0)):
    leap = 1

if (leap == 1) and (month > 2):
    sum += 1

print('第%d天' % sum)

```

5、输入三个整数x,y,z，请把这三个数由小到大输出。

```python
l = []
for i in range(3):
    x = int(input('input:\n'))
    l.append(x)

l.sort()
print(l)
```

6、斐波那契数列。

```python
def fib(n):
    a, b = 1, 1
    for i in range(n - 1):
        a, b = b, a + b

    return a


i = int(input('input:\n'))

print('fib(%d) = %d' % (i, fib(i)))
```

```python
def fib(n):
    if n == 1:
        return [1]

    if n == 2:
        return [1,1]
    fibs = [1,1]
    for i in range(2, n):
        fibs.append(fibs[-1] + fibs[-2])
    return fibs

print(fib(10))
```

7、将一个列表的数据复制到另一个列表中。

```python
a = [1,2,3]
b = a[0:]
print(b)
```

8、输出 9*9 乘法口诀表。

```python
# print不换行 ：
# python3 end=" "
# python2 print x,  #只需要加一个逗号即可
for i in range(1,10):
    print()
    for j in range(1,i + 1):
        print('%d * %d = %d' % (i, j, i*j), end=" ")
```

9、暂停一秒输出。

```python
import time

myMap = {1 : 'aaaa', 2 : 'bbbb' }

for key,value in dict.items(myMap):
    print(key, value)
    time.sleep(1) # 暂停1s

```

10、暂停一秒输出，并格式化当前时间。

```python
import time

print(time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(time.time())))

time.sleep(1)

print(time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(time.time())))
```

11、古典问题：有一对兔子，从出生后第3个月起每个月都生一对兔子，小兔子长到第三个月后每个月又生一对兔子，假如兔子都不死，问每个月的兔子总数为多少？

```python
f1, f2 = 1, 1
for i in range(1,22):
    print('%12ld %12ld' % (f1,f2),end="  ")
    if (i % 3) == 0:
        print(" ")
    f1 = f1 + f2
    f2 = f1 + f2

```

12、判断101-200之间有多少个素数，并输出所有素数。

```python
from math import sqrt

leap = 1
for i in range(101,201):
    for j in range(2, int(sqrt(i + 1)) + 1):
        if i % j == 0:
            leap = 0
            break

    if leap == 1:
        print('%d' % i)

    leap = 1

```

13、打印出所有的"水仙花数"，所谓"水仙花数"是指一个三位数，其各位数字立方和等于该数本身。例如：153是一个"水仙花数"，因为153=1的三次方＋5的三次方＋3的三次方。

```python
for n in range(100,1000):
    i = int(n / 100)
    j = int(n / 10 % 10)
    k = int(n % 10)
    if n == (i ** 3 + j ** 3 + k ** 3):
        print(n)
```

14、将一个正整数分解质因数。例如：输入90,打印出90=2*3*3*5。

```python
def reduceNum(n):
    print('{} = '.format(n),end=" ")
    if not isinstance(n,int) or n < 0:
        print('请输入正确数字！')
        exit(0)
    elif n in [1]:
        print('{}'.format(n))

    while n not in [1]:
        for index in range(2,n+1):
            if n % index == 0:
                n = int(n/index)
                if n == 1:
                    print(index)
                else :
                    print('{} * '.format(index),end="")
                break

reduceNum(100)
reduceNum(90)
```

15、利用条件运算符的嵌套来完成此题：学习成绩>=90分的同学用A表示，60-89分之间的用B表示，60分以下的用C表示。

```python
score = int(input('输入分数：\n'))

if score >= 90:
    grade = 'A'
elif score >=60:
    grade = 'B'
else:
    grade = 'C'
print('{} 属于 {}'.format(score, grade))

```

16、输出指定格式的日期。

```python
import datetime

if __name__ == '__main__':

    # 输出今日日期，格式为dd/mm/yyyy。
    print(datetime.date.today().strftime('%d/%m/%Y'))

    # 创建日期对象
    selfDate = datetime.date(1992,2,15)

    print(selfDate.strftime('%d/%m/%Y'))

    # 日期算术运算
    selfDate = selfDate + datetime.timedelta(days=1)

    print(selfDate.strftime('%d/%m/%Y'))

    # 日期替换
    selfDate = selfDate.replace(year=selfDate.year+1)

    print(selfDate.strftime('%m/%d/%Y'))
```

17、输入一行字符，分别统计出其中英文字母、空格、数字和其它字符的个数。

```python
# 输入一行字符，分别统计出其中英文字母、空格、数字和其它字符的个数。

import string

str = input('输入字符串：\n')

letters = 0
space = 0
digit = 0
others = 0
i = 0

while i < len(str):
    c = str[i]
    i += 1
    if c.isalpha():
        letters += 1
    elif c.isspace():
        space += 1
    elif c.isdigit():
        digit += 1
    else:
        others += 1
print('char = {},space = {},digit = {}, others = {}'.format(letters,space,digit,others))

```

```python
import string

str = input('输入字符串：\n')

letters = 0
space = 0
digit = 0
others = 0
i = 0

for c in str:
    i += 1
    if c.isalpha():
        letters += 1
    elif c.isspace():
        space += 1
    elif c.isdigit():
        digit += 1
    else:
        others += 1
print('char = {},space = {},digit = {}, others = {}'.format(letters,space,digit,others))

```

18、求s=a+aa+aaa+aaaa+aa...a的值，其中a是一个数字。例如2+22+222+2222+22222(此时共有5个数相加)，几个数相加由键盘控制。

```python
from functools import reduce

temp = 0
arr = []

n = int(input("n=\n"))
a = int(input("a=\n"))

for count in range(n):
    temp = temp + a
    a = a * 10
    arr.append(temp)
    print(temp)

arr = reduce(lambda x,y : x + y, arr)

print("和为：{}".format(arr))
```

19、一个数如果恰好等于它的因子之和，这个数就称为"完数"。例如6=1＋2＋3.编程找出1000以内的所有完数。

```python
for i in range(2,1001):

    k = [] # 保存一个数的所有因子
    s = i
    for j in range(1,i):
        if i % j == 0:
            k.append(j)
            s -= j
    if s == 0 :
        print(i)
        print(k)

```

20、一球从100米高度自由落下，每次落地后反跳回原高度的一半；再落下，求它在第10次落地时，共经过多少米？第10次反弹多高？

```python
tour = []
height = []
hei = 100.0
tim = 10

for i in range(1,tim + 1):
    if i == 1:
        tour.append(hei)
    else:
        tour.append(2 * hei)
    hei /= 2
    height.append(hei)

print('总高度：tour = {0}'.format(sum(tour)))
print('第10次反弹的共度：height：{0}'.format(height[-1]))
```

21、猴子吃桃问题：猴子第一天摘下若干个桃子，当即吃了一半，还不瘾，又多吃了一个第二天早上又将剩下的桃子吃掉一半，又多吃了一个。以后每天早上都吃了前一天剩下的一半零一个。到第10天早上想再吃时，见只剩下一个桃子了。求第一天共摘了多少。

```python
last = 1
for i in range(9,0,-1):
    sum = (last + 1) * 2
    last = sum

print(sum)

```

22、两个乒乓球队进行比赛，各出三人。甲队为a,b,c三人，乙队为x,y,z三人。已抽签决定比赛名单。有人向队员打听比赛的名单。a说他不和x比，c说他不和x,z比，请编程序找出三队赛手的名单。

```python
for i in range(ord('x'),ord('z') + 1):
    for j in range(ord('x'),ord('z') + 1):
        if i != j:
            for k in range(ord('x'),ord('z') + 1):
                if (i != k) and (j != k):
                    if (i != ord('x')) and (k != ord('x')) and (k != ord('z')):
                        print ('order is a -- %s\t b -- %s\tc--%s' % (chr(i),chr(j),chr(k)))

```

23、打印出如下图案（菱形）:

   *
  ***
 *****
*******
 *****
  ***
   *


```python
from sys import stdout

for i in range(4):
    for j in range(2 - i + 1):
        stdout.write(" ");
    for j in range(i * 2 + 1):
        stdout.write("*")
    print()

for i in range(3):
    for j in range(i + 1):
        stdout.write(" ")
    for j in range(4 - 2 * i + 1):
        stdout.write("*")
    print()

```

24、有一分数序列：2/1，3/2，5/3，8/5，13/8，21/13...求出这个数列的前20项之和。

```python
sum = 0.0
a = 2.0
b = 1.0

for n in range(1,21):
    sum += a/b
    a,b = a+b,a

print(sum)

```

```python
from functools import reduce

a = 2.0
b = 1.0
l = []

l.append(a/b)

for n in range(1,20):
    b,a = a,a+b
    l.append(a/b)

print(reduce(lambda x,y: x + y,l))


```

25、求1+2!+3!+...+20!的和。

```python
s = 0
t = 1
for i in range(1,21):
    t *= i
    s += t

print('1! + 2! + 3! + ... + 20! = {}'.format(s))
```