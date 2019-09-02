```java
BufferedReader reader = new BufferedReader(new InputStreamReader(requestWrapper.getInputStream()));

String str = "";
String wholeStr = "";
while((str = reader.readLine()) != null){//一行一行的读取body体里面的内容；
wholeStr += str;
}
JSONObject jsonObj = JSON.parseObject(wholeStr);
String version = jsonObj.getString("client_version");
System.out.println(wholeStr);
```

