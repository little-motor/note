[toc]
# 1. 引言
Jackson是一个简单基于Java应用库，Jackson可以轻松的将Java对象转换成json对象和xml文档，同样也可以将json、xml转换成Java对象。
Jackson主要有三种方式处理JSON：
1. 数据绑定，它有两个类型，简单的数据绑定可以将JSON与Java Maps, Lists, Strings, Numbers, Booleans，null进行转换。全部数据绑定可以将JSON与任意JAVA类型转换。通过ObjectMapper类读/写JSON。数据绑定是最方便的方式是类似XML的JAXB解析器。
2. 流式API，读取并将JSON内容写入作为离散事件。 JsonParser读取数据，而JsonGenerator写入数据。它是三者中最有效的方法，是最低的开销和最快的读/写操作。它类似于Stax解析器XML。
3. 树模型，将JSON文件在内存里以树形式表示， ObjectMapper构建JsonNode节点树，这是最灵活的方法，它类似于XML的DOM解析器。	

# 2. ObjectMapper类
需要记住的大致过程事首先创建ObjectMapper类，readValue()方法可以实现从json文件到实例到反序列化过程，writeValueAsString()方法可以实现从实例到json文件的序列化过程。需要注意的是Mapper实例是完全线程安全的，前提是实例的所有配置都发生在任何读或写调用之前。
```
public class JacksonTester {
   public static void main(String args[]){
      ObjectMapper mapper = new ObjectMapper();
      String jsonString = "{\"name\":\"Mahesh\", \"age\":21}";

      //map json to student
      try {
         //将json字符串反序列化为对象实例
         Student student = mapper.readValue(jsonString, Student.class);
         System.out.println(student);
         //格式化输出字符串
         mapper.enable(SerializationConfig.Feature.INDENT_OUTPUT);
         //将实例序列化为字符串
         jsonString = mapper.writeValueAsString(student);
         System.out.println(jsonString);

      } catch (JsonParseException e) {
         e.printStackTrace();
      } catch (JsonMappingException e) {
         e.printStackTrace();
      } catch (IOException e) {
         e.printStackTrace();
      }
   }

   //将对象实例序列化为json文件
   private void writeJSON(Student student) throws JsonGenerationException, JsonMappingException, IOException{
      ObjectMapper mapper = new ObjectMapper();	
      mapper.writeValue(new File("student.json"), student);
   }

   //将json文件反序列化为对象实例
   private Student readJSON() throws JsonParseException, JsonMappingException, IOException{
      ObjectMapper mapper = new ObjectMapper();
      Student student = mapper.readValue(new File("student.json"), Student.class);
      return student;
   }
}

class Student {
   private String name;
   private int age;
   public Student(){}
   public String getName() {
      return name;
   }
   public void setName(String name) {
      this.name = name;
   }
   public int getAge() {
      return age;
   }
   public void setAge(int age) {
      this.age = age;
   }
   public String toString(){
      return "Student [ name: "+name+", age: "+ age+ " ]";
   }	
}
```


						
	

参考易百教程https://www.yiibai.com/jackson/jackson_objectmapper.html