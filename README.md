# 如何在GitHub上上传文件
## 进入GitHub主页，点击右上角头像旁+，选择new repository
![image](https://user-images.githubusercontent.com/34829040/126111911-52ee4e72-c270-47c8-bdb2-719af09da9c9.png)
## 输入仓库名以及一些其他设置，仓库名最好不要用中文，当然自己学习测试也就无所谓了
![image](https://user-images.githubusercontent.com/34829040/126112056-03884afd-139d-4c38-bf45-9da898ed977f.png)
## 仓库建立完成后，为了能够从本地上传文件，还需要下载安装git工具，下载连接如下：
https://gitforwindows.org/  
完成后傻瓜式安装即可
## 在建立好仓库的界面，找到标注的地址，复制备用
![image](https://user-images.githubusercontent.com/34829040/126112778-0973aac0-04e3-4b67-bef7-2967f626aabe.png)
## 然后是本地操作，采用git工具的bash或者windows控制台操作均可
### 在本地建立一个用作提交的文件夹，cd进入
### 建立git仓库
git init
### 复制欲提交的文件进这个文件夹，添加所有文件进仓库
git add .
### 提交到仓库
git commit -m "注释信息"（注释可不写，主要是为了表明这次提交的意图）
### 将本地仓库关连到GitHub
git remote add origin 刚才复制的链接
### 上传前pull一下
git pull origin master
### 上传
git push -u origin master

# SpringBoot+Mybatis+Vue.js+element-ui实现简单的数据库操作前后端
## SpringBoot+MyBatis（使用IDEA）
### 建立项目，File->New->Project，选择Spring Initializr，其他设置可以参考下图，点击Next进行下一步
![new_project](https://user-images.githubusercontent.com/34829040/126280550-b3ea3223-8678-4610-b7ed-63837e3ee03d.PNG)
### 项目构建前的依赖选择，参考下图，其他依赖可以稍后在pom.xml中自行添加
![dependencies](https://user-images.githubusercontent.com/34829040/126280571-e245d4b8-cbc4-4438-8243-eef09bd1a471.PNG)
### 项目构建完成后，目录结构如下图所示
![file_structure](https://user-images.githubusercontent.com/34829040/126282635-e26cfe47-34e6-4a34-86bd-92ebfab992b8.PNG)
### 各个目录存放代码的作用
#### config：配置类，处理前后端的跨域请求
#### controller：前端控制器，处理前后端的交互
#### entity：数据实体类，确定从数据库中查询得到数据的结构
#### mapper：数据接口访问，在xml文件或类中实现具体的sql操作
#### service：数据服务接口层
##### -serviceImpl：数据服务接口实现层
#### utils：工具类
#### resources：资源目录，存放项目配置文件和资源文件等
#### test：与测试相关的文件
#### target：经过编译的文件的存放位置
#### pom.xml：进行依赖的相关配置
#### 其余文件暂时无需关心
### 建立数据表teacher（仅作参考，可自行建立）
```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for teacher
-- ----------------------------
DROP TABLE IF EXISTS `teacher`;
CREATE TABLE `teacher`  (
  `teacher_id` int NOT NULL AUTO_INCREMENT,
  `teacher_name` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `teacher_dept_id` int NULL DEFAULT NULL,
  `teacher_title` varchar(10) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `teacher_tel` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `teacher_address` varchar(200) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `teacher_email` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`teacher_id`) USING BTREE,
  INDEX `fk_teacher_dept_id`(`teacher_dept_id`) USING BTREE,
  CONSTRAINT `fk_teacher_dept_id` FOREIGN KEY (`teacher_dept_id`) REFERENCES `department` (`dept_id`) ON DELETE SET NULL ON UPDATE CASCADE
) ENGINE = InnoDB AUTO_INCREMENT = 57 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

SET FOREIGN_KEY_CHECKS = 1;
```
### 编写代码
#### 首先配置数据库连接，在resources文件夹下application.properties中写入如下基本配置内容
```xml
spring.datasource.url=jdbc:mysql://localhost:3306/addresslist?characterEncoding=UTF-8&serverTimezone=GMT%2B8&useServerPrepStms=true&cachePrepStms=true
<!--连接数据库的端口号、名字等信息-->
spring.datasource.username=root<!--访问数据库的用户名-->
spring.datasource.password=Xiang136<!--访问数据的密码-->
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver<!--数据库访问驱动-->
mybatis.mapper-locations=mapper/**.xml<!--mapper文件的位置-->
server.port= 6934<!--后端服务的端口号，可自行设置-->
```
#### 编写entity层的代码Teacher.class（代码编写顺序可按个人习惯）
```java
package com.example.springboot_mybatis.entity;
public class Teacher {
    public Integer teacher_id;
    public String teacher_name;
    public Integer teacher_dept_id;
    public String teacher_title;
    public String teacher_tel;
    public String teacher_address;
    public String teacher_email;
    //在IDEA选中以上字段，按alt+insert键可以生成相应的getter和setter，此处省略
}
```
#### 编写mapper层的代码（代码编写顺序可按个人习惯）
##### TeacherMapper.java接口文件
```java
package com.example.springboot_mybatis.mapper;
import com.example.springboot_mybatis.entity.Teacher;
import org.apache.ibatis.annotations.*;
import java.util.List;

@Mapper//该注解声明一个mybatis的数据操作接口，会被SpringBoot扫描到
public interface TeacherMapper {
    List<Teacher> findByName(String name);
    List<Teacher> findAll();
    List<Teacher> findAllwithDeptName();
}

```
##### TeacherMapper.xml mapper方法的具体sql语句
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.springboot_mybatis.mapper.TeacherMapper">
    <resultMap id="teacher" type="com.example.springboot_mybatis.entity.Teacher">
        <id column="teacher_id" jdbcType="INTEGER" property="teacher_id"/>
        <result column="teacher_name" jdbcType="VARCHAR" property="teacher_name"/>
        <result column="teacher_dept_id" jdbcType="INTEGER" property="teacher_dept_id"/>
        <result column="teacher_tel" jdbcType="VARCHAR" property="teacher_tel"/>
        <result column="teacher_title" jdbcType="VARCHAR" property="teacher_title"/>
        <result column="teacher_address" jdbcType="VARCHAR" property="teacher_address"/>
        <result column="teacher_email" jdbcType="VARCHAR" property="teacher_email"/>
    </resultMap>
    <!--resuleMap中内容指定了查询结果的数据类型-->
    <!--本次只尝试了数据查询操作，故只有select标签，注意标签中id要与mapper接口中方法名对应-->
    <select id="findByName" resultType="com.example.springboot_mybatis.entity.Teacher">
        select * from teacher where teacher_name like concat('%',#{name},'%')
    </select>
    <select id="findAll" resultType="com.example.springboot_mybatis.entity.Teacher">
        select * from teacher
    </select>
    <select id="findAllwithDeptName" resultType="com.example.springboot_mybatis.entity.Teacher">
        SELECT
            t1.teacher_id,
            t1.teacher_name,
            d1.dept_name,
            t1.teacher_title,
            t1.teacher_tel,
            t1.teacher_address,
            t1.teacher_email
        FROM
            teacher t1
                JOIN department d1 ON t1.teacher_dept_id = d1.dept_id
    </select>
</mapper>
```
#### 编写service层具体代码
##### 服务接口TeacherService.java
```java
package com.example.springboot_mybatis.service;

import com.example.springboot_mybatis.entity.Teacher;
import org.springframework.stereotype.Service;

import java.util.List;


public interface TeacherService {
    public List<Teacher> findByName(String name);

    public List<Teacher> findAll();

    public List<Teacher> findAllwithDeptName();
}
```
##### impl包下TeacherServicesImpl.java，服务接口实现
```java
package com.example.springboot_mybatis.service.impl;

import com.example.springboot_mybatis.entity.Teacher;
import com.example.springboot_mybatis.mapper.TeacherMapper;
import com.example.springboot_mybatis.service.TeacherService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service//该注解表明该类用于数据服务，可以被扫描到
public class TeacherServiceImpl implements TeacherService {

    @Autowired
    private TeacherMapper teacherMapper;

    @Override
    public List<Teacher> findByName(String name) {
        return teacherMapper.findByName(name);
    }

    @Override
    public List<Teacher> findAll() {
        return teacherMapper.findAll();
    }

    @Override
    public List<Teacher> findAllwithDeptName() {
        return teacherMapper.findAllwithDeptName();
    }

}
```
#### controller层代码编写
##### TeacherController.java
```java
package com.example.springboot_mybatis.controller;

import com.example.springboot_mybatis.entity.Teacher;
import com.example.springboot_mybatis.service.TeacherService;
import org.apache.ibatis.annotations.Param;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController//将方法返回的对象展示为json格式
@RequestMapping("/teacher")//该注解会将HTTP请求映射到controller的处理方法上
public class TeacherController {

    @Autowired
    private TeacherService teacherService;

    @RequestMapping("/getByName",method = RequestMethod.GET)
    public List<Teacher> getMsg(@Param("name") String name){
        List<Teacher> teacherList = teacherService.findByName(name);
        return teacherList;
    }

    @RequestMapping(value = "/getAll",method = RequestMethod.GET)
    public List<Teacher> getAll(){
        List<Teacher> teacherList = teacherService.findAll();
        return teacherList;
    }

}
```
### 编写到这里，其实已经可以看看效果了，config层的代码可以先按下不表
#### 启动项目，打开浏览器，在地址栏输入localhost:6934/teacher/getAll，可以看到被解析为json格式的查询结果
![image](https://user-images.githubusercontent.com/34829040/126291059-6f9998da-00ff-40da-adeb-554683eec59f.png)
#### 显然，数据的展示形式并不是多美观，渲染到比较好看的前端界面就比较必要了，于是我们进行到下一部分，运用Vue.js框架编写前端页面
## Vue.js + element-ui（使用vscode编写）

