---
layout: post
title: Python-Study
date: 2019-11-05
tags: python
---

> python 学习(python3)

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

26、利用递归方法求5!

```python
sum = 0
def fact(n):
    if n == 0:
        sum = 1
    else:
        sum = n * fact(n - 1)
    return sum

print(fact(5))
```

27、利用递归函数调用方式，将所输入的5个字符，以相反顺序打印出来。

```python

def output(s,n):
    if n == 0:
        return
    print(s[n-1],end=" ")
    output(s,n-1)

s = input("输入字符串：\n")

l = len(s)

output(s,l)

```

28、有5个人坐在一起，问第五个人多少岁？他说比第4个人大2岁。问第4个人岁数，他说比第3个人大2岁。问第三个人，又说比第2人大两岁。问第2个人，说比第一个人大两岁。最后问第一个人，他说是10岁。请问第五个人多大？

```python
def age(n):
    if n == 1:
        c = 10
    else:
        c = age(n - 1) + 2
    return c

print(age(5))

```

29、给一个不多于5位的正整数，要求：一、求它是几位数，二、逆序打印出各位数字。

```python
x = int(input("输入一个数字：\n"))

if int(x / 10000) != 0:
    print("五位数")
elif int(x % 10000 / 1000) != 0:
    print("四位数")
elif int(x % 1000 / 100) != 0:
    print("三位数")
elif int(x % 100 / 10) != 0:
    print("二位数")
else:
    print("一位数")
```

30、一个5位数，判断它是不是回文数。即12321是回文数，个位与万位相同，十位与千位相同。

```python
x = int(input("输入一个数字：\n"))

str = str(x)
len = len(str)
flag = True
for i in range(int(len / 2)):
    if(str[i] != str[- i - 1]):
        flag = False
        break


if flag:
    print("是回文数")
else:
    print("不是回文数")

```

31、请输入星期几的第一个字母来判断一下是星期几，如果第一个字母一样，则继续判断第二个字母。

```python
str = input("please input:")

if str == 'S':
    print('please input second letter:')
    str = input("please input:")
    if str == 'a':
        print('Saturday')
    elif str == 'u':
        print('Sunday')
    else:
        print('data error')

elif str == 'F':
    print('Friday')

elif str == 'M':
    print('Monday')

elif str == 'T':
    print('please input second letter')
    str = input("please input:")

    if str == 'u':
        print('Tuesday')
    elif str == 'h':
        print('Thursday')
    else:
        print('data error')

elif str == 'W':
    print('Wednesday')
else:
    print('data error')

```

32、按相反的顺序输出列表的值。

```python
a = ['aaa','bbb','ccc']

for i in a[::-1]:
    print(i)

```

33、按逗号分隔列表。

```python
list = [1,2,3,4,5]

s1 = ','.join(str(n) for n in list)

print(s1)
```

35、文本颜色设置。

```python
class bcolors:
    HEADER = '\033[95m'
    OKBLUE = '\033[94m'
    OKGREEN = '\033[92m'
    WARNING = '\033[93m'
    FAIL = '\033[91m'
    ENDC = '\033[0m'
    BOLD = '\033[1m'
    UNDERLINE = '\033[4m'

print( bcolors.HEADER + "字体?" + bcolors.ENDC)
print( bcolors.OKBLUE + "蓝色字体?" + bcolors.ENDC)
print( bcolors.OKGREEN + "绿色字体?" + bcolors.ENDC)
print( bcolors.WARNING + "警告的颜色字体?" + bcolors.ENDC)
print( bcolors.FAIL + "失败?" + bcolors.ENDC)
print( bcolors.ENDC + "结束符号" + bcolors.ENDC)
print( bcolors.BOLD + "加粗?" + bcolors.ENDC)
print( bcolors.UNDERLINE + "下划线?" + bcolors.ENDC)
```

36、求100之内的素数。

```python
for num in range(1,101):
    for i in range(2,num):
        if (num % i) == 0:
            break
    else:
        print(num)
```

37、对10个数进行排序。

```python
nums = [2,1,3,2,4,5,3,2,7,0]

nums.sort()

print("-----")

for i in nums:
    print(i, end=" ")

```

38、求一个3*3矩阵主对角线元素之和。

```python
a = [[1,1,2],[3,3,3],[1,2,3]]

sum = 0
for i in range(3):
    for j in range(3):
        sum += a[i][i]

print(sum)
```

39、有一个已经排好序的数组。现输入一个数，要求按原来的规律将它插入数组中。

```python

if __name__ == '__main__':
    # 0作为加入数字的占位
    a = [1,3,4,5,6,7,8,13,15,100,0]
    print("原始列表")
    for i in a:
        print(i, end=" ")

    number = int(input("\n 插入一个数字：\n"))
    end = a[9]
    if number > end:
        a[10] = number
    else:
        for i in  range(10):
            if a[i] > number:
                temp = a[i]
                a[i] = number
                for j in range(i + 1, 11):
                    temp1 = a[j]
                    a[j] = temp
                    temp = temp1
                break

    print("插入后")
    for i in a:
        print(i,end=" ")

```

40、将一个数组逆序输出。

```python
a = [1,2,3,4,5,6]

print("原数组:" + str(a))

for i in a[::-1]:
    print(i,end=" ")

```

41、模仿静态变量的用法。

```python
def varfunc():
    var = 0
    print('var = %d' % var)
    var += 1
if __name__ == '__main__':
    for i in range(3):
        varfunc()

# 类的属性
# 作为类的一个属性吧
class Static:
    StaticVar = 5
    def varfunc(self):
        self.StaticVar += 1
        print(self.StaticVar)

print(Static.StaticVar)
a = Static()
for i in range(3):
    a.varfunc()
```

43、模仿静态变量(static)另一案例。

```python
class Num:
    nNum = 1

    def inc(self):
        self.nNum += 1
        print('class Num中的：nNum = %d' % self.nNum)


if __name__ == '__main__':
    nNum = 2
    inst = Num()
    for i in range(3):
        nNum += 1
        print('外边循环中的num：The num = %d' % nNum)
        inst.inc()

```

44、两个 3 行 3 列的矩阵，实现其对应位置的数据相加，并返回一个新矩阵：

X = [[12,7,3],
    [4 ,5,6],
    [7 ,8,9]]

Y = [[5,8,1],
    [6,7,3],
    [4,5,9]]

```python
X = [[12,7,3],
    [4,5,6],
    [7,8,9]]

Y = [[5,8,1],
    [6,7,3],
    [4,5,9]]

res = [[0,0,0],
       [0,0,0],
       [0,0,0]]

for x in range(len(X)):
    for y in range(len(X[0])):
        res[x][y] = X[x][y] + Y[x][y]

for i in res:
    print(i)
```

45、统计 1 到 100 之和。

```python
res = 0

for i in range(1,101):
    res += i

print("和：{}".format(res))
```

46、求输入数字的平方，如果平方运算后小于 50 则退出。

```python
def SQ(x):
    return x * x

again = True

while again:
    num = int(input("请输入一个数字："))
    print("运算结果为：{}".format(SQ(num)))
    if SQ(num) >= 50:
        again = True
    else:
        again = False

```

47、两个变量值互换。

```python
def exchage(x,y):
    x,y = y,x
    return (x,y)

if __name__ == '__main__':
    x = 10
    y = 20
    print("x = {}, y = {}".format(x,y))
    x,y = exchage(x,y)
    print("x = {}, y = {}".format(x,y))

```

49、使用lambda来创建匿名函数。

```python
MAXIMUM = lambda x, y: (x > y) * x + (x < y) * y
MINIMUM = lambda x, y: (x > y) * y + (x < y) * x

if __name__ == '__main__':
    a = 10
    b = 20
    print( 'The largar one is %d' % MAXIMUM(a, b))

    print('The lower one is %d' % MINIMUM(a, b))
```

50、输出一个随机数。

```python
import random

# 生成0,1的随机数
print(random.random())
# 生成10到20中间的随机数
print(random.uniform(10,20))
# 生成10到20的随机整数
print(random.randint(10,20))
# 输出0,到99间的随机数
print(random.choice([x for x in range(1,100)]))
```

55、按位取反~。

```python
if __name__ == '__main__':
    a = 234
    b = ~a
    print('The a\'s 1 complement is %d' % b) # -235
    a = ~a
    print('The a\'s 2 complement is %d' % a) # -235
```

61、打印出杨辉三角形（要求打印出10行如下图）
1 
1 1 
1 2 1 
1 3 3 1 
1 4 6 4 1 
1 5 10 10 5 1 
1 6 15 20 15 6 1 
1 7 21 35 35 21 7 1 
1 8 28 56 70 56 28 8 1 
1 9 36 84 126 126 84 36 9 1

```python
if __name__ == '__main__':
    a = []
    for i in range(10):
        a.append([])
        for j in range(10):
            a[i].append(0)
    for i in range(10):
        a[i][0] = 1
        a[i][i] = 1
    for i in range(2,10):
        for j in range(1,i):
            a[i][j] = a[i - 1][j-1] + a[i - 1][j]
    from sys import stdout
    for i in range(10):
        for j in range(i + 1):
            stdout.write(str(a[i][j]))
            stdout.write(' ')
        print()
```

62、查找字符串。　

```python
sStr1 = 'abcdefg'
sStr2 = 'cde'
print(sStr1.find(sStr2))
```

63、