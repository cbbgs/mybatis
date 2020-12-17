

# 什么是mybatis

​		MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。



# mybatis配置

mybatis官网:https://mybatis.org/mybatis-3/zh/getting-started.html

在resourse文件下新建一个mybatisconfig

.xml文件

xml配置文件的头部:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
```

xml配置文件的内容:

```xml

<!--配置数据库的链接信息-->
<configuration>
    <!--配置环境链接-->
    <environments default="">
        <!--单个数据库配置信息-->
        <environment id="">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="POOLED">
                <!--配置用户名-->
                <property name="username" value="root"/>
               <!-- 配置密码-->
                <property name="password" value="123"/>
                <!--配置驱动-->
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <!--地址-->
                <property name="url" value="jdbc:mysql://localhost:3306/test?serverTimezone=UTC"/>
            </dataSource>
        </environment>
    </environments>

```

# mybatis操作数据库

### 创建一个mybatis的工具类

```java
public class MybatisConfig {
    public static SqlSessionFactory sqlSessionFactory = null;

    static {
        Reader reader = null;
        try {
            /*
            * 获取mybatis的配置文件
            * */
            reader = Resources.getResourceAsReader("MybatisConfig.xml");
            /*
            * 通过SqlSessionFactoryBuilder.build();来获得sqlsession,传入的参数是读取的mybatis的配置文件
            * */
            sqlSessionFactory =new SqlSessionFactoryBuilder().build(reader);
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            /*
            * 关闭文件流
            * */
            if(reader!=null){
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    }


    /*
    * 写一个函数来的到sqlsession
    * */
    public static SqlSession getSqlSession(){
        return  sqlSessionFactory.openSession();
    }

    /*
    * 关闭sqlsession
    *
    * */

    public static  void close(SqlSession sqlSession){
        if(sqlSession !=null){
            sqlSession.close();
        }
    }
}
```

### 创建一个dao

```java

/*
* 数据访问对象
* */
public class StudentDao {
    public int save(Student student){
        /*
        * 得到sqlsession
        * */
        SqlSession sqlSession = MybatisConfig.getSqlSession();

        /*
        * 添加语句,调用sqlsession中的函数,类似于jpa仓库
        * 第一个参数就是和studentMapper.xml中的ID里的一样
        * 第二个参数就是这个方法传递的参数
        * */
        Integer flag = sqlSession.insert("save",student);
        /*
        * 插入数据对数据库有改动,提交事务
        * */

        sqlSession.commit();
        /*
        * 关闭sqlsession
        * */
        MybatisConfig.close(sqlSession);
        return flag;
    }
}

```

### 创建一个实体类



安装了Lombok

```java

/*
* Serializable实现序列化接口,在分布式架构中必须实现序列化
* */
@Data
public class Student  implements Serializable {
    private String studentName;
    private Integer studentId;


}


```

### 创建mapper

在resouse下创建一个mapper文件夹,在mapper文件夹下创建一个studentMapper.xml

mapper的头

```xml
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
```

mapper的体

```xml
<!--namespace绑定dao的类名-->
<mapper namespace="cn.cbbgs.dao.StudentDao">
    <!--id就是到中的方法-->
    <insert id="save">
        insert into student(studentName,studentId) values(#{studentName},#{studentId});

    </insert>
</mapper>
```

### 添加映射

在mybatis.xml里面添加

```
  <!--数据库的映射文件,可以配置多个-->
    <mappers>
        <mapper resource="mapper/studentMapper.xml"/>
    </mappers>
```

### 测试

添加junit4的

```
 <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
```

编写测试类

```java
public class TestMybatis {
    @Test
    public void testSave(){
        Student student = new Student();
        student.setStudentName("张三");
        student.setStudentId(1222);
        StudentDao.save(student);
    }
}

```

## 查询

#### 单个查询

dao

```java

    /*
    * 查询一个
    * */

    public static  Student getStudentById(Integer studentId){
        /*
         * 得到sqlsession
         * */

        SqlSession sqlSession = MybatisConfig.getSqlSession();


        /*
         * 添加语句,调用sqlsession中的函数,类似于jpa仓库
         * 第一个参数就是和studentMapper.xml中的ID里的一样
         * 第二个参数就是这个方法传递的参数
         * */

       Student student =  sqlSession.selectOne("getStudentById",studentId);
       /*
       * 查询不用提交事务,直接关闭
       * */

        MybatisConfig.close(sqlSession);
        return  student;

    }

```

studentmapper.xml

```xml
  <select id="getStudentById" resultType="cn.cbbgs.entity.Student">
        select * from student where studentId=#{studentId};
    </select>
```

测试

```java
  @Test
  public void testGetStudentByid(){
       Student student =  StudentDao.getStudentById(12);
        System.out.println(student.toString());
    }
```

#### 查询所有

dao

```java
/*
    * 查找所有用户
    * */

    public static List<Student> showAll(){
        SqlSession sqlSession = MybatisConfig.getSqlSession();
       List<Student> students =  sqlSession.selectList("showall");
       MybatisConfig.close(sqlSession);
       return  students;
    }
```

studentMapper.xml

```xml
 <!--查询所有,注意返回值还是实体类-->
    <select id="showall" resultType="cn.cbbgs.entity.Student">
        select * from student;
    </select>
```







## 删除

#### 删除单个



dao

```java 

    /*
    * 删除单个
    * */

    public static  int deleteById(Integer studentId){
        /*
         * 得到sqlsession
         * */

        SqlSession sqlSession = MybatisConfig.getSqlSession();

       /*
         * 添加语句,调用sqlsession中的函数,类似于jpa仓库
         * 第一个参数就是和studentMapper.xml中的ID里的一样
         * 第二个参数就是这个方法传递的参数
         * */
       int f =  sqlSession.delete("deleteById",studentId);

       /*
       * 提交事务
       * */
        sqlSession.commit();
        /*
        * 关闭流
        * */
        MybatisConfig.close(sqlSession);
        return  f;


    }
```

studentmapper.xml

```xml
<delete id="deleteById">
        delete from student where studentId=#{studentId};
</delete>
```

测试

```java
 @Test
 public void testDeleteById(){

        int student =  StudentDao.deleteById(12);
        System.out.println(student);
    }
```

## 修改

#### 修改单个

dao

```java
 /*
    * 更新操作
    * */


    public static  int updateStudent(Student student){
        /*
        * 得到sqlsession
        * */
        SqlSession sqlSession = MybatisConfig.getSqlSession();

        /*
         * 添加语句,调用sqlsession中的函数,类似于jpa仓库
         * 第一个参数就是和studentMapper.xml中的ID里的一样
         * 第二个参数就是这个方法传递的参数
         * */
       int f =  sqlSession.update("updateStudent",student);

       /*
       * 提交事务
       * */

        sqlSession.commit();

        /*
        * 关闭流
        * */

        MybatisConfig.close(sqlSession);
        return f;
    }
```

studentmapper.xml

```xml
   <delete id="deleteById">
        delete from student where studentId=#{studentId};
    </delete>
```

测试类

```java
 @Test
    public void testUpdateStudent(){

        Student student = new Student();
        student.setStudentName("李四");
        student.setStudentId(1222);
        StudentDao.updateStudent(student);

    }
```

## 数据库链接优化

* 在resourse下创建一个文件db.properties,把数据库链接的参数写在里面

  ```properties
  username=root
  password=123
  driver=com.mysql.cj.jdbc.Driver
  url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC
  ```

  

* 修改mybatisConfig.xml

  ```xml
              <dataSource type="POOLED">
                  <!--配置用户名-->
                  <property name="username" value="${username}"/>
                  <!-- 配置密码-->
                  <property name="password" value="${password}"/>
                  <!--配置驱动-->
                  <property name="driver" value="${driver}"/>
                  <!--地址-->
                  <property name="url" value="${url}"/>
              </dataSource>
          </
  ```

  在environments前添加

  ```xml
   <!--引入外部数据源-->
      <properties resource="db.properties"></properties>
  ```

  

## 日志配置

* 在mybatisConfig.xml中添加

```xml
<!--配置日志-->
<settings>
    <!--在控制台打印sql-->
    <setting name="logImpl" value="LOG4J"/>
</settings>
```

* 然后在resoues下创建一个文件log4j.properties

```properties
# 全局日志配置
log4j.rootLogger=ERROR, stdout
# MyBatis 日志配置,这个cn.cbbgs.dao 是自己的dao 包的路径
log4j.logger.cn.cbbgs.dao=TRACE
# 控制台输出
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

导入日志jar包

```xml
 <!--开启日志-->
    <!-- https://mvnrepository.com/artifact/log4j/log4j -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
```



## mybatis接口编程

* 将数据访问层的dao写成接口,创建一个StudentMapper接口

```java

public interface StudentMapper {
    /*
     * 插入数据
     * */
    public   int save(Student student);

    /*
     * 查询一个
     * */
    public   Student getStudentById(Integer studentId);

    /*
     * 删除单个
     * */

    public   int deleteById(Integer studentId);

    /*
     * 更新操作
     * */
    public   int updateStudent(Student student);



    /*
     * 查找所有用户
     * */

    public  List<Student> showAll();

    /*
     * 按条件查找
     * */
    public   List<Student> findById(int studentId);
}

```

* 编写 studentMapper.xml

```xml


<!--namespace接口,这里的id接口的方法名一致-->
<mapper namespace="cn.cbbgs.dao.StudentMapper">
    <insert id="save">
        insert into student(studentId,studentName) values (#{studentId},#{studentName});
    </insert>

    <select id="getStudentById" resultType="cn.cbbgs.entity.Student">
        select * from student where studentId=#{studentId};
    </select>

    <delete id="deleteById">
        delete from student where studentId=#{studentId};
    </delete>

    <update id="updateStudent" >
       update student set studentName=#{studentName} where studentId=#{studentId};
   </update>

   <!-- 查询所有,返回值还是实体类-->
    <select id="showAll" resultType="cn.cbbgs.entity.Student">
        select * from student;
    </select>

   <!-- 按条件查找-->
    <select id="findbyid" resultType="cn.cbbgs.entity.Student">

    </select>
</mapper>

```

* 创建service层

在service 包下创建一个类

```java
public class StudentServiceImpl {

    /*添加*/
    public static  int save(Student student){
        /*得到sqlsession*/
        SqlSession sqlSession = MybatisConfig.getSqlSession();
        /*通过反射得到studentMapper*/
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
       int flag =  mapper.save(student);
       sqlSession.commit();
       MybatisConfig.close(sqlSession);
       return  flag;
    }
    /*通过Id查询学生*/
    public static  Student getStudentById(int studentId){
        /*得到sqlsession*/
        SqlSession sqlSession = MybatisConfig.getSqlSession();
        /*通过反射得到studentMapper*/
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
        Student student = mapper.getStudentById(studentId);
        MybatisConfig.close(sqlSession);
        return  student;
    }

    /*通过Id删除学生*/
    public static  int deleteById(int studentId){
        /*得到sqlsession*/
        SqlSession sqlSession = MybatisConfig.getSqlSession();
        /*通过反射得到studentMapper*/
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
        int flag = mapper.deleteById(studentId);
        sqlSession.commit();
        MybatisConfig.close(sqlSession);
        return  flag;
    }
    /*更新操作*/
    public static  int updateStudent(Student student){
        /*得到sqlsession*/
        SqlSession sqlSession = MybatisConfig.getSqlSession();
        /*通过反射得到studentMapper*/
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
        int flag = mapper.updateStudent(student);
        sqlSession.commit();
        MybatisConfig.close(sqlSession);
        return  flag;
    }


    //*查询所有*/
    public static List<Student> showall(){
        /*得到sqlsession*/
        SqlSession sqlSession = MybatisConfig.getSqlSession();
        /*通过反射得到studentMapper*/
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
        List<Student> students = mapper.showAll();
        MybatisConfig.close(sqlSession);
        return  students;
    }

}

```

* 创建新的测试类测试一下

```java

public class TestImpel {

    @Test
    public void testSave(){
        Student student = new Student();
        student.setStudentName("陈斌");
        student.setStudentId(6);
        StudentServiceImpl.save(student);
    }


    @Test
    public void testGetStudentByid(){

        Student student =  StudentServiceImpl.getStudentById(4);
        System.out.println(student.toString());
    }

    @Test
    public void testDeleteById(){

        int student =  StudentServiceImpl.deleteById(4);
        System.out.println(student);
    }

    @Test
    public void testUpdateStudent(){

        Student student = new Student();
        student.setStudentName("fuxiaobo");
        student.setStudentId(2);
        StudentServiceImpl.updateStudent(student);

    }

    @Test
    public void testShowAll() {
        List<Student> students = StudentServiceImpl.showall();
        for (Student student : students) {
            System.out.println(student.toString());
        }
    }

}

```

## mybatis中传递多个参数



* #### 顺序传参

  ```java
  
  public User select(String name,int id);
  
  
  <select id="select" resultMap="UserResultMap">
  select * from user where name=#{0} and id=#{1}
  </select>
  ```

\#{}里面的数字代表你传入参数的顺序

* #### @Param注解传参

  ```java
  public User select(@Param("paramName") String name,@Param("paramId") int id);
  
  
  <select id="select" resultMap="UserResultMap">
  select * from user where name=#{paramName} and id=#{paramId}
  </select>
  ```

\#{}里面的名称对应的是注解 @Param括号里面修饰的名称。

* #### Map传参

  ```java
  Map<String,Object> map = new HashMap<String, Object>();
  map.put("mapName","zhangsan");
  map.put("mapId",2);
  public User select(Map<String,Object> map);
  
  
  <select id="select" parameterType="java.util.Map" resultMap="UserResultMap">
  select * from user where name=#{mapName} and id=#{mapId}
  </select>
  ```

\#{}里面的名称对应的是 Map里面的key名称。

* #### Java Bean传参

  ```java
  User user = new User();
  user.setName("zhangsan");
  user.setId(2);
  public User select(User user);
  
  
  <select id="select" parameterType="com.pojo.User" resultMap="UserResultMap">
  select * from user where name=#{name} and id=#{id}
  </select>
  
  ```

  \#{}里面的名称对应的是 User类里面的成员属性。



## 分页

* 准备接口中的方法

StudentMapper.java

```java 
  /*分页查询*/
    public  List<Student> listPage(@Param("page") int page, @Param("size") int size);

    /*
    * 统计总数*/
    public int total();
```

* 准备sql语句

studentmapper.xml

```xml

   <!-- 分页查找-->
    <select id="listPage" resultType="cn.cbbgs.entity.Student">
            select  * from student limit #{page},#{size}
    </select>

    <!--统计总数-->
    <select id="total" resultType="int">
        select count(*) from student
    </select>
```

* service中的实现

StudentServiceImpi.java

```java
 /*分页查询*/
    public static  List<Student> listPage(int page,int size){
        SqlSession sqlSession = MybatisConfig.getSqlSession();
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
        List<Student> students = mapper.listPage(page, size);
        sqlSession.close();
        return students;
    }

    /*统计总数
    * */
    public static  int total(){
        SqlSession sqlSession = MybatisConfig.getSqlSession();
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
        int total = mapper.total();
        return total;
    }

```

* servlet中

```java

@WebServlet("/studentServlet")
public class StudentServlet extends HttpServlet {


    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setCharacterEncoding("UTF-8");
        resp.setCharacterEncoding("UTF-8");
        PrintWriter out = resp.getWriter();
        String m = req.getParameter("m");
       if("showall".equals(m)){
           out.print(new Gson().toJson(StudentServiceImpl.showall()));
       }else  if("save".endsWith((m))){
           Student student = new Student();
           student.setStudentId(Integer.parseInt(req.getParameter("studentId")));
           student.setStudentName(req.getParameter("studentName"));
           StudentServiceImpl.save(student);
           out.print("添加成功!");

       }else  if("getStudentById".equals(m)){
           Student student = StudentServiceImpl.getStudentById(Integer.parseInt(req.getParameter("studentId")));
           out.print(new Gson().toJson(student));
           
           //分页部分
       }else if("listPage".equals(m)){
           /*前台的数据，由essayui提供*/
           int page = Integer.parseInt(req.getParameter("page"));//必须是page
           int size = Integer.parseInt(req.getParameter("rows"));//必须是rows
           /*分页查询出记录*/
           List<Student> students = StudentServiceImpl.listPage((page-1)*size, size);
           int total = StudentServiceImpl.total();
           /*定义一个map,将总记录数和当前记录封装*/
           Map<String,Object> map = new HashMap<>();
           map.put("total",total);//建必须是total
           map.put("rows",students);//键必须是rows
            /*把这个map给出去*/
           out.print(new Gson().toJson(map));
       }
       out.flush();
       out.close();
    }
}

```

* 前端部分利用easyUi

![](img/easyUi.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <!--主题皮肤-->
    <link rel="stylesheet" type="text/css" href="essayui/themes/default/easyui.css">
    <!--图片样式-->
    <link rel="stylesheet" type="text/css" href="essayui/themes/icon.css">
    <!--jquery-->
    <script type="text/javascript" src="essayui/jquery.min.js"></script>
    <script type="text/javascript" src="essayui/jquery.easyui.min.js"></script>

    <script type="text/javascript" src="essayui/easyui-lang-zh_CN.js"></script>

</head>
<!--用js渲染表格-->
<script type="text/javascript">
    $(function(){
        $('#dg').datagrid({
            /*访问的对象*/
            url:'studentServlet?m=listPage',
           /*让表格铺满浏览器*/
            fit:true,
            fitColumns:true,/*设置为true将自动使列适应表格宽度以防止出现水平滚动*/
            pagination:true,/*开启分页*/
            pagePosition:'both',/*分页显示位置*/
            toolbar:'#tb',/*搜索框*/

            /*表头*/
            columns:[[
                {field:'studentId',title:'学号',width:100},/*checkbox,批量操作*/
                {field:'studentName',title:'姓名',width:100},

            ]]
        });

    })



</script>
<body>
<!--搜索栏-->
<div>
    <table>
        <tr>
            <td>
                <input class="easyui-validatebox" type="text" name="name" data-options="required:true" />
                <a id="btn" href="#" class="easyui-linkbutton" data-options="iconCls:'icon-search'">查询</a>
            </td>
        </tr>
    </table>

</div>


<div id="tb">
    <a href="#" class="easyui-linkbutton" data-options="iconCls:'icon-add',plain:true">添加</a>
    <a href="#" class="easyui-linkbutton" data-options="iconCls:'icon-edit',plain:true">修改</a>
    <a href="#" class="easyui-linkbutton" data-options="iconCls:'icon-remove',plain:true">删除</a>

</div>
<!-- 准备一个表格显示数据-->
<table id="dg"></table>

</body>
</html>
```



## 多表查询

* 数据库表如下

![](/img/b1.png)

![](img/b2.png)

* 定义的两个实体类

在学生实体类中定义一个班级的自定义类型的属性

```java
public class Student {
    private Integer studentId;
    private String studentName;
    private Grade grade;

    public Integer getStudentId() {
        return studentId;
    }

    public void setStudentId(Integer studentId) {
        this.studentId = studentId;
    }

    public String getStudentName() {
        return studentName;
    }

    public void setStudentName(String studentName) {
        this.studentName = studentName;
    }

    public Grade getGrade() {
        return grade;
    }

    public void setGrade(Grade grade) {
        this.grade = grade;
    }

    @Override
    public String toString() {
        return "Student{" +
                "studentId=" + studentId +
                ", studentName='" + studentName + '\'' +
                '}';
    }
}
```

```java
public class Grade implements  Serializable {
    private Integer classId;
    private String className;

    public Integer getClassId() {
        return classId;
    }

    public void setClassId(Integer classId) {
        this.classId = classId;
    }

    public String getClassName() {
        return className;
    }

    public void setClassName(String className) {
        this.className = className;
    }
}

```



* 主要是studentMapper.xml里面

```
在单表操作中，我们使用**resultType**来指定返回类型，并将得到的值依次类型传递到业务逻辑中去，而在多变关联查询中，我们则不使用resultType，而是使用**resultMap**将返回值以Map的形式传递到resultMap操作中。
```







```xml

    <!--多表查询,采用-->
    <select id="listPage" resultMap="studentMapper">
        SELECT * FROM student s LEFT JOIN grade  g ON
            s.classId=g.classId
        <where>
            <if test="studentName!=null and studentName !=''">
                studentName=#{studentName}
            </if>
        </where>
        limit #{page},#{size}
    </select>
    <!--多表映射操作-->
    <resultMap id="studentMapper" type="cn.cbbgs.entity.Student">
        <!--主键数据的填充-->
        <id property="studentId" column="studentId"></id>
        <!--其余属性-->
        <result property="studentName" column="studentName"></result>

        <!--引用数据类型-->
        <association property="grade" javaType="cn.cbbgs.entity.Grade">
            <id property="classId" column="classId"></id>
            <result property="className" column="className"></result>

        </association>
    </resultMap>


```

```
id与result，id用于设置主键的映射关系（这里也包括外联表的主键），而result则用于设置非主键的映射关系
olumn与property：column则是列名，即SQL语句返回出来的列名，property则是Bean对象的属性名
```

​	

