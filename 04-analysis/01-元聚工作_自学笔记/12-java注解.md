##### 1,  reflection

```java
@Test
public void  reflect_test() throws IllegalAccessException {
    IMModel imModel = new IMModel();
    imModel.setAge(10);
    imModel.setName("ivanl001");
    imModel.setNum(1000010L);

    Field[] fields = imModel.getClass().getDeclaredFields();

    for (Field field : fields) {
        //这个需要设定一下，保证可以取出私有变量，要不然会报错IllegalAccessException
        field.setAccessible(true);
        //先获取字段，然后通过字段取出字段对应的值
        Object obj = field.get(imModel);
        System.out.println(field.getName() + "-----------" + obj.toString());
    }

    System.out.println("ivanl001----test-----done");
}
```

