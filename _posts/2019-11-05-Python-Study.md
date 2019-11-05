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

```