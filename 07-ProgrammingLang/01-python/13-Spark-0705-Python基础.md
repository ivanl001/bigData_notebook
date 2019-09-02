## 1，python的基本数据类型
* Numbers
  * int
  * long
  * float
  * complex #复数
* String
  * 切片用:分割，而不是,[0: 4]，前包后不包
  * 切片：[5:20:2]表示从5个字符开始，到20个字符结束，每隔两个字符取一次， 这里20不是长度
  * str01 = r"ivan is the king of world \n and so cool"  # 这里是原样输出，不会转义
  * print(str01)
* List
  * list01 = [1, 2, 34, "ivanl001", "kkkk"]
  * 可变，任何元素都可以二次赋值
* Tuple
  * tuple01 = (1, 2, "zhang", 444)
  * print(tuple01[0])
  * 注意：感觉和数组差不多，但是有一点就是不可以二次赋值
* Dictionary
  * dictionary01 = {1:10, "name":"ivan", 2:333, "key":"king"}
  * print(dictionary01)
  * print(dictionary01.get("key"))

## 2, 类型转换
`规则：目标数据类型(需要转换的数据)`
* age = 10
* ageStr = str(age)

`eval求值`
* print(eval("10+20"))

`tuple((1, 2, 3, 4)):将序列转成元祖，因为接受一个参数，也就是徐磊，所以内部需要加上括号`
* seq01 = 1, 3, 5, 7, 9;
* d = tuple(seq01)

## 3，分支语句
### 3.1，for循环
  ```python
  for i in range(10):
    print(i)
  
  for i in 1,3,4,5,9:
    print(i)
  ```
### 3.2，if循环
  ```python
  age = 10
  if age < 16:
    print("You are too young")
  elif age < 40:
    print("You are cool")
  else:
    print("sorry, you are old enough")
  ```

## 4，文件操作
### 4.1，文件拷贝

```python
# 1, 一次性读
print("# 1, 一次性读")
f = open("/Users/ivanl001/Desktop/bigData/input/ivanl001.txt")
lines = f.readlines()
for line in lines:
    # 这个意思是打印但是不换行
    print(line, end='')

# 2, 每次读取一行
print()
print("2, 每次读取一行")
f = open("/Users/ivanl001/Desktop/bigData/input/ivanl001.txt")
while True:
    line = f.readline()
    if  line != '':
        print(line, end="")
    else:
        # 这里就是换个行
        print()
        break

# 3,字节读取，文件拷贝等
print("# 3,字节读取，文件拷贝等")
input01 = open("/Users/ivanl001/Pictures/test01.jpg", "rb")
output01 = open("/Users/ivanl001/Pictures/test01-copy.jpg","wb")

print("拷贝开始")
while True:
    print("---0000")
    # 这里是设置缓存，一次缓存多少字节
    data = input01.read(1024*1024) # 这里是1024*1024byte，也就是1M
    print(len(data))
    if len(data) == 0:
        break
    else:
        output01.write(data)

input01.close()
output01.close()
print("拷贝结束")
```

### 4.2, os系统操作

```python
import os

# 修改文件名
os.rename("/Users/ivanl001/Desktop/01.jpg", "/Users/ivanl001/Desktop/01-rename.jpg")
```





