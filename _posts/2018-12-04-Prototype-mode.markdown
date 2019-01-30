#### 原型模式

##### 定义
指原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。
##### 特点
不需要知道创建的细节，不调用构造函数
##### 类型
创建型
##### 适用场景
1. 类初始化消耗较多资源
2. new产生的一个对象需要非常繁琐的过程（数据准备、访问权限等）
3. 构造函数比较复杂
4. 循环体中生产大量对象时

##### 优点
1. 原型模式性能比直接new 一个对象性能高
2. 简化创建过程

##### 缺点
1. 必须配备克隆方法
2. 对克隆复杂对象或克隆出的对象进行复杂改造时，容易引入风险。
3. 深拷贝、浅拷贝必须引用得当

下面看代码，写代码之前我们先假设一个业务场景，假设我们现在要发一个构建一个邮件对象给别人发送邮件告诉别人中奖了，这个对象构建起来非常麻烦，当然我们的代码不一定会构建非常复杂的对象。

邮件工具类
```java
public class MailUtil {
    public static void sendMail(Mail mail){
        String outputContent = "向{0}同学,邮件地址:{1},邮件内容:{2}发送邮件成功";
        System.out.println(MessageFormat.format(outputContent,mail.getName(),mail.getEmailAddress(),mail.getContent()));
    }
    public static void saveOriginMailRecord(Mail mail){
        System.out.println("存储originMail记录,originMail:"+mail.getContent());
    }
}
```
邮件实体类
```java
public class Mail implements Cloneable{
    private String name;
    private String emailAddress;
    private String content;
    public Mail(){
        System.out.println("Mail Class Constructor");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmailAddress() {
        return emailAddress;
    }

    public void setEmailAddress(String emailAddress) {
        this.emailAddress = emailAddress;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    @Override
    public String toString() {
        return "Mail{" +
                "name='" + name + '\'' +
                ", emailAddress='" + emailAddress + '\'' +
                ", content='" + content + '\'' +
                '}'+super.toString();
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        System.out.println("clone mail object");
        return super.clone();
    }
}
```
这个实体类注意要实现Cloneable接口，然后重写Object的clone方法，直接返回父类的的clone方法就可以了，我们在里面添加了一个输入方便等会儿我们测试。下面看测试类。

测试类
```java
public class PrototypeTest {
    public static void main(String[] args) throws CloneNotSupportedException {
        Mail mail = new Mail();
        mail.setContent("初始化模板");
        System.out.println("初始化mail:"+mail);
        for(int i = 0;i < 10;i++){
            Mail mailTemp = (Mail) mail.clone();
            mailTemp.setName("姓名"+i);
            mailTemp.setEmailAddress("姓名"+i+"@redstar.com");
            mailTemp.setContent("恭喜您，你中奖了");
            MailUtil.sendMail(mailTemp);
            System.out.println("克隆的mailTemp:"+mailTemp);
        }
        MailUtil.saveOriginMailRecord(mail);
    }
}
```
然后我们看看测试类运行结果。

```java
克隆的mailTemp:Mail{name='姓名4', emailAddress='姓名4@redstar.com', content='恭喜您，你中奖了'}com.design.pattern.creational.prototype.Mail@312b1dae
clone mail object
向姓名5同学,邮件地址:姓名5@redstar.com,邮件内容:恭喜您，你中奖了发送邮件成功
克隆的mailTemp:Mail{name='姓名5', emailAddress='姓名5@redstar.com', content='恭喜您，你中奖了'}com.design.pattern.creational.prototype.Mail@7530d0a
clone mail object
向姓名6同学,邮件地址:姓名6@redstar.com,邮件内容:恭喜您，你中奖了发送邮件成功
克隆的mailTemp:Mail{name='姓名6', emailAddress='姓名6@redstar.com', content='恭喜您，你中奖了'}com.design.pattern.creational.prototype.Mail@27bc2616
clone mail object
向姓名7同学,邮件地址:姓名7@redstar.com,邮件内容:恭喜您，你中奖了发送邮件成功
克隆的mailTemp:Mail{name='姓名7', emailAddress='姓名7@redstar.com', content='恭喜您，你中奖了'}com.design.pattern.creational.prototype.Mail@3941a79c
clone mail object
向姓名8同学,邮件地址:姓名8@redstar.com,邮件内容:恭喜您，你中奖了发送邮件成功
克隆的mailTemp:Mail{name='姓名8', emailAddress='姓名8@redstar.com', content='恭喜您，你中奖了'}com.design.pattern.creational.prototype.Mail@506e1b77
clone mail object
向姓名9同学,邮件地址:姓名9@redstar.com,邮件内容:恭喜您，你中奖了发送邮件成功
克隆的mailTemp:Mail{name='姓名9', emailAddress='姓名9@redstar.com', content='恭喜您，你中奖了'}com.design.pattern.creational.prototype.Mail@4fca772d
存储originMail记录,originMail:初始化模板
```
注意看打印结果的内存地址，他们打印的地址是不一样的，他们不是同一个对象。使用clone方式操作的是内存中的二进制文件，比直接new一个对象性能好的多。

##### 深克隆和浅克隆
既然讲到了clone那我们就来说说深克隆和浅克隆吧。关于这个我们看一段代码。


```java
public class Pig implements Cloneable{
    private String name;
    private Date birthday;

    public Pig(String name, Date birthday) {
        this.name = name;
        this.birthday = birthday;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }

    @Override
    public String toString() {
        return "Pig{" +
                "name='" + name + '\'' +
                ", birthday=" + birthday +
                '}'+super.toString();
    }
}
```
这段代码我们定义了一个pig小猪佩奇类，有两个属性名字和生日，我们写一下他的测试类。

```java
public class PigTest {
    public static void main(String[] args) throws CloneNotSupportedException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Date birthday = new Date(0L);
        Pig pig1 = new Pig("佩奇",birthday);
        Pig pig2 = (Pig) pig1.clone();
        System.out.println(pig1);
        System.out.println(pig2);
    }
}
```
我们运行一下看看结果
```
Pig{name='佩奇', birthday=Thu Jan 01 08:00:00 CST 1970}com.design.pattern.creational.prototype.clone.Pig@27973e9b
Pig{name='佩奇', birthday=Thu Jan 01 08:00:00 CST 1970}com.design.pattern.creational.prototype.clone.Pig@312b1dae
```
注意看内存地址，好像是符合我们的预期，内存地址是不一样的，那我们修改一下，这个生日属性看看会发生什么

```java
public class PigTest {
    public static void main(String[] args) throws CloneNotSupportedException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Date birthday = new Date(0L);
        Pig pig1 = new Pig("佩奇",birthday);
        Pig pig2 = (Pig) pig1.clone();
        System.out.println(pig1);
        System.out.println(pig2);
        pig1.getBirthday().setTime(666666666666L);
        System.out.println(pig1);
        System.out.println(pig2);
    }
}
```
运行结果：

```
Pig{name='佩奇', birthday=Thu Jan 01 08:00:00 CST 1970}com.design.pattern.creational.prototype.clone.Pig@27973e9b
Pig{name='佩奇', birthday=Thu Jan 01 08:00:00 CST 1970}com.design.pattern.creational.prototype.clone.Pig@312b1dae
Pig{name='佩奇', birthday=Sat Feb 16 09:11:06 CST 1991}com.design.pattern.creational.prototype.clone.Pig@27973e9b
Pig{name='佩奇', birthday=Sat Feb 16 09:11:06 CST 1991}com.design.pattern.creational.prototype.clone.Pig@312b1dae
```
仔细看虽然他们的内存地址不一样但是在修改了其中一个生日时发现两个都改变了。这是因为我们的这种方式实现的是一种浅拷贝，生日使用的是date类型是一个引用类型，这两个对象引用的都是同一个地址。为了得到我们期待的结果我们就要修改为深拷贝的方式。
```java
public class Pig implements Cloneable{
    private String name;
    private Date birthday;

    public Pig(String name, Date birthday) {
        this.name = name;
        this.birthday = birthday;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        Pig pig = (Pig)super.clone();

        //深克隆
        pig.birthday = (Date) pig.birthday.clone();
        return pig;
    }

    @Override
    public String toString() {
        return "Pig{" +
                "name='" + name + '\'' +
                ", birthday=" + birthday +
                '}'+super.toString();
    }
}

```
此时我们再看看测试类的运行结果：

```
Pig{name='佩奇', birthday=Thu Jan 01 08:00:00 CST 1970}com..design.pattern.creational.prototype.clone.Pig@27973e9b
Pig{name='佩奇', birthday=Thu Jan 01 08:00:00 CST 1970}com.design.pattern.creational.prototype.clone.Pig@312b1dae
Pig{name='佩奇', birthday=Sat Feb 16 09:11:06 CST 1991}com.design.pattern.creational.prototype.clone.Pig@27973e9b
Pig{name='佩奇', birthday=Thu Jan 01 08:00:00 CST 1970}com.design.pattern.creational.prototype.clone.Pig@312b1dae
```
此时结果就是我们想要的了。对于深拷贝还有一种方式就是通过json序列化一次成json再讲json转成对象，这样也可以实现深拷贝，但是我们说使用拷贝的目的是拷贝是直接修改二进制文件效率高，至于使用转json的方式效率怎么样我不太清楚，欢迎小伙伴们自己试验一下把结果和我交流一下。今天的原型型模式就到这里。