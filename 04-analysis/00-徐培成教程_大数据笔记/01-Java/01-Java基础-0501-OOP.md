* 查看class代码
  
  > javap dog.class

## 1，Adaptor：适配器
> 在java框架中，经常看到有些类名字是**Adaptor，就是适配器。比如spring中的HandlerInterceptorAdapter拦截适配器。

> 适配器的原理大概如下：最底层是一个接口，如果我们需要实现这些方法，如果直接实现接口：很明显接口的所有的方法都需要实现，但是有时候我们只需要实现个别方法，所以通过一个抽象类实现接口，然后抽象类中空实现那些我们有时候不想重写的方法，空实现即可。这个时候再通过继承抽象类的话，必须实现那些抽象类中没有实现的方法，同时还可以重写抽象类中实现的方法

> 下面是boss项目中使用的案例：
```java
//1，我们直接继承抽象类
public class AuthorityInterceptor extends HandlerInterceptorAdapter，这里继承了抽象类，实现了一个preHandle方法，这里不贴代码了，知道就行

//2，抽象类实现接口
public abstract class HandlerInterceptorAdapter implements AsyncHandlerInterceptor {
    public HandlerInterceptorAdapter() {
    }

    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    }

    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    }

    public void afterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    }
}


//3，接口上包装另外一层接口
public interface AsyncHandlerInterceptor extends HandlerInterceptor {
    void afterConcurrentHandlingStarted(HttpServletRequest var1, HttpServletResponse var2, Object var3) throws Exception;
}


//4，最底层的接口
public interface HandlerInterceptor {
    boolean preHandle(HttpServletRequest var1, HttpServletResponse var2, Object var3) throws Exception;

    void postHandle(HttpServletRequest var1, HttpServletResponse var2, Object var3, ModelAndView var4) throws Exception;

    void afterCompletion(HttpServletRequest var1, HttpServletResponse var2, Object var3, Exception var4) throws Exception;
}
```

## 2，关于OOP中this字段
> 首先先解释以下下面这段代码执行过程：
1. 首先main方法压入栈内存
2. main方法的方法帧中创建变量dog，在堆内存中创建Dog对象，并把dog的指针指向堆内存中刚刚创建的Dog对象
3. main方法帧中执行setName方法，在栈中压入setName方法帧，这个方法帧位于main方法上面。根据setName方法内部的内容进行相应的设置
4. 设置完毕后，主线程关闭之前，先把setName方法帧弹出，然后在把main方法弹出，先进后出，完成调用
```java
public static void main(String[] args) {
    Dog dog = new Dog();
    dog.setName("ivanl001");
}
```


> this其实是对象中的一个隐藏的成员变量，指向自己。比如下面的这个构造方法中，如果没有this的话，name则会直接从方法帧，也就是栈中找，而不是从堆中的对象找
```java
public void setName(String name) {
    this.name = name;
}
```