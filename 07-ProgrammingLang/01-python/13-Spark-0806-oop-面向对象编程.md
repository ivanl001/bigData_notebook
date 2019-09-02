## 1， OOP基础
```python
#
# author      : ivanl001
# creator     : 2018-12-06 16:54
# description : 
# 
# 定义类
class Dog:

    # 01, 定义构造函数, 如果有了下面的带参数的构造器，这里的构造器就不能再写了
    def __init__(self):
        print("A:定义构造函数")

    def __init__(self, name, age, num):
        #下面这三个可以直接理解成定义成员变量
        self.name = name
        self.age = age
        self.num = num
        print("A:定义构造函数")

    def __del__(self):
        print("这个是对象销毁方法，也即是析构函数")

    # 02, 定义属性变量, 这里和静态成员变量还是有一定差别的， 这里的可以被更改的哈
    name = "ivanl001"

    # 03, 定义方法
    @staticmethod
    def play():
        print("C:定义并调用方法: dog is playing")

    @staticmethod
    def add(a:int, b:int):
        return a+b

    def print_name(self):
        print("打印成员变量：" + self.name)


# 创建对象(python和java的一个不同就是python可以直接在同一个文件中定义类后并创建
# dog01 = Dog()

# 因为后来的构造方法新增了变量，所以在构造dog的时候需要添加参数
dog = Dog("ivanl002", 10, 10120945)

print("B:打印静态成员变量name:" + dog.name)

# play是静态方法，可以直接对象调用，也可以类调用哈
dog.play()
Dog.play()

# 对象方法
dog.print_name()
# 注意：这里不是静态方法，所以不能直接用Dog类进行调用
# Dog.printName()


print("---------------对象属性-----------------")
# 判断对象是否有某个变量， 也可以判断是否有某个方法的
print(hasattr(dog, "name"))
print(hasattr(dog, "add"))
# 获取属性值
print(getattr(dog, "name"))
# 更改属性值
setattr(dog, "name", "ivanl003")
print(getattr(dog, "name"))
# 删除属性值
delattr(dog, "age")
print(hasattr(dog, "age"))


# 类属性
print("----------------类属性-----------------")
dogDict = Dog.__dict__
for key in dogDict.keys():
    print("key:" + key + ", value:" + str(dogDict[key]))
```

## 2, 多重继承
```python
#
# author      : ivanl001
# creator     : 2018-12-08 11:45
# description : 多重继承
# 
class Horse:

    name = ""

    # 这种是私有属性，子类是不能继承的
    __house_tailLength = 10

    def __init__(self):
        print("这是马--初始化")

    @staticmethod
    def run():
        print("马在跑----")

    # 如下：带有下划线的是私有方法
    @staticmethod
    def __house_do():
        print("这个是马私有的方法，只有本类可以使用，子类也不能用")

    # 如果有self做参数，并且方法内部中使用到self相关的属性，就是对象方法
    def house_work(self):
        print(self.name)
        print("这个是对象方法，因为有对象相关的属性：name:" + self.name)

class Donkey:

    def __init__(self):
        print("这是驴--初始化")

    @staticmethod
    def carry():
        print("驴子在驼东西----")

# 可以看到骡子可以同时继承马和驴
class Luozi(Horse, Donkey):

    def __init__(self):
        # 如果想要调用父类方法
        Donkey.__init__(self)
        Horse.__init__(self)
        print("这是骡子--初始化")

    @staticmethod
    def scream():
        print("骡子在叫----")

    # 重写父类方法
    @staticmethod
    def run():
        Horse.run()
        print("重写父类的方法，先实现父类调用，在调用自身， 骡子也要跑---")

    # 这里如果重写父类方法而不调用父类， 那么就相当与是覆盖了父类方法
    def house_work(self):
        print("lll" + self.name)

luozi = Luozi()
luozi.name = "ivanl001"

luozi.run()
luozi.carry()
luozi.scream()
luozi.house_work()
```