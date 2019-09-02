## 1，Protobuf的使用
* 01, 如果没有安装protoc，需要先安装protoc
* 02, 首先定义模型文件

  ```protobuf
  syntax = "proto2";

  option java_package = "im.ivanl001.bigData.Protobuf";
  option java_outer_classname = "AddressBookProtos";
  message Person {
  
  	required string name = 1;
  	required int32 id = 2; 
  	optional string email = 3; 
  
  	enum PhoneType {
  		MOBILE = 0; 
  		HOME = 1; 
  		WORK = 2;
  	}
  
  	message PhoneNumber {
  		required string number = 1;
  		optional PhoneType type = 2 [default = HOME];
  	}
  	repeated PhoneNumber phone = 4;
  }
  
  message AddressBook {
  	repeated Person person = 1;
  }
  ```
  
* 03，通过protoc编译模型文件为java模型
  
> protoc --java_out=. addressbook.proto
  
* 04, 在03的指定位置，会生成java文件
  ```java
  //超级长，我这里不放代码了
  ```
  
* 05，把代码放进项目中，然后使用模型，写入到文件

  ```java
   @Test
    public void protobufWriteTest() throws IOException {
  
  
        Long current = System.currentTimeMillis();
        AddressBookProtos.Person person = AddressBookProtos.Person.newBuilder()
                .setName("ivanl001")
                .setId(10120945)
                .setEmail("ivanl001@163.com")
                .addPhone(AddressBookProtos.Person.PhoneNumber.newBuilder()
                        .setNumber("13918667287")
                        .setType(AddressBookProtos.Person.PhoneType.HOME)
                        .build()
                ).build();
  
        person.writeTo(new FileOutputStream("/Users/ivanl001/Desktop/bigData/serialization/pro_person.data"));
  
        Long current01 = System.currentTimeMillis();
        System.out.println("duration:----:" + (current01 - current));
        //duration:----:83
  
    }
  ```
  
* 06, 读取文件

  ```java
  @Test
  public void protobufReadTest() throws Exception {
  
      Long current = System.currentTimeMillis();
      AddressBookProtos.Person person = AddressBookProtos.Person.parseFrom(new FileInputStream("/Users/ivanl001/Desktop/bigData/serialization/pro_person.data"));
      System.out.println(person.getName());
      System.out.println(person.getId());
  
      Long current01 = System.currentTimeMillis();
      System.out.println("duration:----:" + (current01 - current));
      //duration:----:86
      //duration:----:68
  }
  ```

## 2, 和java的序列化对比一下
* 01, 定义模型

  ```java
  package im.ivanl001.bigData.Protobuf;

  import java.io.Serializable;
  
  /**
   * #author      : ivanl001
   * #creator     : 2018-11-06 19:52
   * #description : java序列化的演示模型
   **/
  public class Person implements Serializable {
  
      private int id;
      private String name;
      private String phoneNum;
      private String email;
  
  
      public String getEmail() {
          return email;
      }
  
      public void setEmail(String email) {
          this.email = email;
      }
  
      public int getId() {
          return id;
      }
  
      public void setId(int id) {
          this.id = id;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public String getPhoneNum() {
          return phoneNum;
      }
  
      public void setPhoneNum(String phoneNum) {
          this.phoneNum = phoneNum;
      }
  }
  ```
  
* 02, 写入读取文件

  ```java
  //------------------------------下面是java的序列化和反序列化的对比---------------------------------------
    @Test
    public void javaWriteTest() throws Exception {
  
        Long current = System.currentTimeMillis();
        Person person = new Person();
        person.setId(10120945);
        person.setName("ivanl001");
        person.setEmail("ivanl001@163.com");
        person.setPhoneNum("13918667287");
  
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("/Users/ivanl001/Desktop/bigData/serialization/java_person.data"));
        objectOutputStream.writeObject(person);
  
        objectOutputStream.close();
  
        Long current01 = System.currentTimeMillis();
        System.out.println("duration:----:" + (current01 - current));
        //duration:----:29
    }
  
  
    @Test
    public void javaReadTest() throws Exception {
        Long current = System.currentTimeMillis();
  
        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("/Users/ivanl001/Desktop/bigData/serialization/java_person.data"));
        Object object = objectInputStream.readObject();
        Person person = (Person) object;
        System.out.println(person.getId());
        System.out.println(person.getName());
  
        Long current01 = System.currentTimeMillis();
        System.out.println("duration:----:" + (current01 - current));
  
        //duration:----:104
    }
  ```