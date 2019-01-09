> 一般数据库的表结构都会有update_time，修改时间，因为这个字段基本与业务没有太大关联，因此开发过程中经常会忘记设置这两个字段的值，本插件就是来解决这个问题。同样的想生成id，create_time等操作都是可以以同样的方式解决。想折腾的同学还可以通过这中方式自己写个分页插件。闲话少说上代码。


#### 1. 先写一个自定义注解标注是update_time

```
package com.zb.iscrm.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * @Auther: 杨红星
 * @Date: 2018/11/28 09:38
 * @Description:
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD})
public @interface UpdateTime {
    String value() default "";
}

```
 
#### 2. 写一个mybatis插件
> 使用@Intercepts标注这是个mybatis插件，@Signature标注要拦截的操作

```java
package com.zb.iscrm.mybatisInterceptor;

import com.zb.iscrm.annotation.UpdateTime;
import com.zb.iscrm.utils.DateUtils;
import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.executor.Executor;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.mapping.SqlCommandType;
import org.apache.ibatis.plugin.*;

import java.lang.reflect.Field;
import java.util.Properties;

/**
 * @Auther: 杨红星
 * @Date: 2018/11/28 09:41
 * @Description: mybatis插件 用于执行Update时将当前时间加入
 */
@Slf4j
@Intercepts({ @Signature(type = Executor.class, method = "update", args = { MappedStatement.class, Object.class }) })
public class UpdateTimeInterceptor  implements Interceptor {

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        MappedStatement mappedStatement = (MappedStatement) invocation.getArgs()[0];
        // 获取 SQL 命令
        SqlCommandType sqlCommandType = mappedStatement.getSqlCommandType();
        // 获取参数
        Object parameter = invocation.getArgs()[1];
        if (parameter != null) {
            // 获取成员变量
            Field[] declaredFields = parameter.getClass().getDeclaredFields();
            for (Field field : declaredFields) {
                if (field.getAnnotation(UpdateTime.class) != null) { // update 语句插入 updateTime
                    if (SqlCommandType.INSERT.equals(sqlCommandType) || SqlCommandType.UPDATE.equals(sqlCommandType)) {
                        field.setAccessible(true);
                        if (field.get(parameter) == null) {
                            field.set(parameter, DateUtils.dateTimeNow(DateUtils.YYYY_MM_DD_HH_MM_SS));
                        }
                    }
                }
            }
        }
        //同样的方式也可以在这里添加create_time或者是id的生成等处理
        
        return invocation.proceed();
    }

    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    @Override
    public void setProperties(Properties properties) {
    }
}

```
#### 最后在mybatis的配置文件中注册插件，然后就大功告成


```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--插件注册-->
    <plugins>
        <plugin interceptor="com.zb.iscrm.mybatisInterceptor.UpdateTimeInterceptor"/>
    </plugins>
</configuration>
```

