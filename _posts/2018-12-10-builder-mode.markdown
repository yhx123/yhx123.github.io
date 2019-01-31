---
layout:     post
title:      " 建造者模式 "
subtitle:   " 设计模式之建造者模式 "
date:       2019-01-9 12:00:00
author:     "redstar"
header-img: ""
tags:
    - Design Patterns
---



#### 建造者模式
##### 定义

  * 讲一个复杂的对象的构建与他的表示分离，使得同样的构建过程可以创建不同的表示。
  * 用户只需指定需要建造的类型就可以得到他们，建造过程及其细节不需要知道
##### 类型
    创建型
##### 适用场景
* 如果一个对象有非常复杂的内部结构（很多属性）
* 想把复杂对象的创建和使用分离
##### 优点
* 封装性好，创建和使用分离
* 拓展性好、建造类之间独立、一定程度上解耦
##### 缺点
* 产生多余的Builder对象
* 产品内部发生变化、建造者都要修改、成本较大。

> 建造者模式和工厂模式比较接近，其中建造者模式强调的方法的调用顺序。而工厂模式强调的是创建产品。另外他们的创建粒度也不同。建造者模式创建出来的产品复杂的，而工厂模式创建出出来的都是一个样。工厂模式注重的是把这个对象创建出来就可以。建造者模式不仅要创建产品还要知道这个产品的部件组成。

下面开始简单粗暴看代码
业务场景假设我们现在要制作课程，这个课程需要视频、ppt、文章的信息。同时我们在制作课程的时候还需要一个指导老师。

首先我们来创建一个课程类，把属性，set get toString 方法等填充一下。
```java
public class Course {
    private String courseName;
    private String coursePPT;
    private String courseVideo;
    private String courseArticle;

    //question & answer
    private String courseQA;

    public String getCourseName() {
        return courseName;
    }

    public void setCourseName(String courseName) {
        this.courseName = courseName;
    }

    public String getCoursePPT() {
        return coursePPT;
    }

    public void setCoursePPT(String coursePPT) {
        this.coursePPT = coursePPT;
    }

    public String getCourseVideo() {
        return courseVideo;
    }

    public void setCourseVideo(String courseVideo) {
        this.courseVideo = courseVideo;
    }

    public String getCourseArticle() {
        return courseArticle;
    }

    public void setCourseArticle(String courseArticle) {
        this.courseArticle = courseArticle;
    }

    public String getCourseQA() {
        return courseQA;
    }

    public void setCourseQA(String courseQA) {
        this.courseQA = courseQA;
    }

    @Override
    public String toString() {
        return "Course{" +
                "courseName='" + courseName + '\'' +
                ", coursePPT='" + coursePPT + '\'' +
                ", courseVideo='" + courseVideo + '\'' +
                ", courseArticle='" + courseArticle + '\'' +
                ", courseQA='" + courseQA + '\'' +
                '}';
    }
}
```
紧接着我们创建一个抽象建造方法
```java
public abstract class CourseBuilder {

    public abstract void buildCourseName(String courseName);
    public abstract void buildCoursePPT(String coursePPT);
    public abstract void buildCourseVideo(String courseVideo);
    public abstract void buildCourseArticle(String courseArticle);
    public abstract void buildCourseQA(String courseQA);

    public abstract Course makeCourse();

}
```
实体建造方法

```java

public class CourseActualBuilder extends CourseBuilder {

    private Course course = new Course();


    @Override
    public void buildCourseName(String courseName) {
        course.setCourseName(courseName);
    }

    @Override
    public void buildCoursePPT(String coursePPT) {
        course.setCoursePPT(coursePPT);
    }

    @Override
    public void buildCourseVideo(String courseVideo) {
        course.setCourseVideo(courseVideo);
    }

    @Override
    public void buildCourseArticle(String courseArticle) {
        course.setCourseArticle(courseArticle);
    }

    @Override
    public void buildCourseQA(String courseQA) {
        course.setCourseQA(courseQA);
    }

    @Override
    public Course makeCourse() {
        return course;
    }
}
```
我们前面说了这个课程我们还有个指导老师，下面我们来创建一个指导老师类。
通过这个指导老师我们对属性进行注入同时返回建造好了的对象。

```java
public class Coach {
    private CourseBuilder courseBuilder;

    public void setCourseBuilder(CourseBuilder courseBuilder) {
        this.courseBuilder = courseBuilder;
    }

    public Course makeCourse(String courseName,String coursePPT,
                             String courseVideo,String courseArticle,
                             String courseQA){
        this.courseBuilder.buildCourseName(courseName);
        this.courseBuilder.buildCoursePPT(coursePPT);
        this.courseBuilder.buildCourseVideo(courseVideo);
        this.courseBuilder.buildCourseArticle(courseArticle);
        this.courseBuilder.buildCourseQA(courseQA);
        return this.courseBuilder.makeCourse();
    }
}
```
最后是测试类
```java
public class BuilderTest {
    public static void main(String[] args) {
        CourseBuilder courseBuilder = new CourseActualBuilder();
        Coach coach = new Coach();
        coach.setCourseBuilder(courseBuilder);

        Course course = coach.makeCourse("Java设计模式精讲",
                "Java设计模式精讲PPT",
                "Java设计模式精讲视频",
                "Java设计模式精讲笔记",
                "Java设计模式精讲问答");
        System.out.println(course);

    }
}
```
这个上面的代码是可以优化的，我们可以将课程类和课程的类的建造者放在同一个类中，将建造写成课程类的内部类。下面开始给出代码。
这里同时也使用了链式调用的方法。
```java
public class Course {

    private String courseName;
    private String coursePPT;
    private String courseVideo;
    private String courseArticle;

    //question & answer
    private String courseQA;

    public Course(CourseBuilder courseBuilder) {
        this.courseName = courseBuilder.courseName;
        this.coursePPT = courseBuilder.coursePPT;
        this.courseVideo = courseBuilder.courseVideo;
        this.courseArticle = courseBuilder.courseArticle;
        this.courseQA = courseBuilder.courseQA;
    }


    @Override
    public String toString() {
        return "Course{" +
                "courseName='" + courseName + '\'' +
                ", coursePPT='" + coursePPT + '\'' +
                ", courseVideo='" + courseVideo + '\'' +
                ", courseArticle='" + courseArticle + '\'' +
                ", courseQA='" + courseQA + '\'' +
                '}';
    }

    public static class CourseBuilder{
        private String courseName;
        private String coursePPT;
        private String courseVideo;
        private String courseArticle;

        //question & answer
        private String courseQA;

        public CourseBuilder buildCourseName(String courseName){
            this.courseName = courseName;
            return this;
        }


        public CourseBuilder buildCoursePPT(String coursePPT) {
            this.coursePPT = coursePPT;
            return this;
        }

        public CourseBuilder buildCourseVideo(String courseVideo) {
            this.courseVideo = courseVideo;
            return this;
        }

        public CourseBuilder buildCourseArticle(String courseArticle) {
            this.courseArticle = courseArticle;
            return this;
        }

        public CourseBuilder buildCourseQA(String courseQA) {
            this.courseQA = courseQA;
            return this;
        }

        public Course build(){
            return new Course(this);
        }

    }
}
```
此时应用的使用即为如下


```java
public class V2BuilderTest {
    public static void main(String[] args) {
        Course course = new Course.CourseBuilder().buildCourseName("Java设计模式精讲")
                .buildCoursePPT("Java设计模式精讲PPT").buildCourseVideo("Java设计模式精讲视频").build();
        System.out.println(course);
    }
}
```
下面我们一起来看一下mybatis中对于建造者模式的应用吧，首先我们打开SqlSessionFactoryBuilder这个类。


![](https://user-gold-cdn.xitu.io/2018/12/6/16783fae0e0380ee?w=893&h=533&f=png&s=517440)

从图中我们很明显可以看出这些方法都是返回这个SqlSessionFactory对象
还有这个XMLConfigBuilder(reader, environment, properties)明显就是解析mybatis xml配置的。
这里核心就是builder方法我们来看一下默认的builder方法。

```java
public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
  }
```
这里将配置传给默认的工厂进行构造。而Configuration是怎么来的呢，我们查看一下这个方法build(Reader reader, String environment, Properties properties)

```java
public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        reader.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }

```
XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);这个方法是在建造这种再使用建造者。对xml文件解析，这里调用了parse方法，我们再查看一下这个parse方法看看。

```java
  public Configuration parse() {
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }
```
这里调用了parseConfiguration，我们再进去看一下这个里面parseConfiguration方法。

```java
private void parseConfiguration(XNode root) {
    try {
      Properties settings = settingsAsPropertiess(root.evalNode("settings"));
      //issue #117 read properties first
      propertiesElement(root.evalNode("properties"));
      loadCustomVfs(settings);
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      reflectorFactoryElement(root.evalNode("reflectorFactory"));
      settingsElement(settings);
      // read it after objectFactory and objectWrapperFactory issue #631
      environmentsElement(root.evalNode("environments"));
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
```
看到这里就非常清晰了，他主要负责个个组件的构建和装配。从上到下就是整个装配的流程。这才是复杂整个构建的核心，而SqlSessionFactory只是对构建进行封装而已，所以我们这个是建造者里面使用建造者。
