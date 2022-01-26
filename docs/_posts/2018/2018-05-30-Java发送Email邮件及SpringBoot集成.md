---
layout: post
title: Java发送Email邮件及SpringBoot集成
date: 2018-05-30 19:36:00
author: 薛师兄
tags:
  - Java
  - SpringBoot
---
# 一：普通方式发送

### 1.导包

```xml
<!--Java MAil 发送邮件API-->
<dependency>
    <groupId>javax.mail</groupId>
    <artifactId>javax.mail-api</artifactId>
    <version>1.6.1</version>
</dependency>
<!-- 真正的实现库 -->
<dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>javax.mail</artifactId>
    <version>1.6.1</version>
</dependency>
```

### 2.代码

*  实例：通过一个已知的163邮箱发送给他人邮件

```java
public static void main(String[] args) throws MessagingException {
    // 1.创建一个程序与邮件服务器会话对象 Session
    Properties props = new Properties();
    props.setProperty("mail.transport.protocol", "SMTP");
    props.setProperty("mail.smtp.host", "smtp.163.com");
    props.setProperty("mail.smtp.port", "25");
    // 指定验证为true
    props.setProperty("mail.smtp.auth", "true");
    props.setProperty("mail.smtp.timeout", "1000");
    // 验证账号及密码，密码对应邮箱授权码（163密码可行，qq必须授权码）
    Authenticator auth = new Authenticator() {
        public PasswordAuthentication getPasswordAuthentication() {
            return new PasswordAuthentication("xxxx@163.com", "xxxxx");
        }
    };
    Session session = Session.getInstance(props, auth);

    // 2.创建一个Message，它相当于是邮件内容
    Message message = new MimeMessage(session);
    // 设置发送者
    message.setFrom(new InternetAddress("xxx同上xxx@163.com"));
    // 设置发送方式与接收者
    message.setRecipient(MimeMessage.RecipientType.TO, new InternetAddress("xx10086xx@qq.com"));
    // 设置主题
    message.setSubject("邮件发送测试");
    // 设置内容
    message.setContent("Hello!", "text/html;charset=utf-8");

    // 3.创建 Transport用于将邮件发送
    Transport.send(message);
}
```


* 实例：通过一个已知的QQ邮箱发送给他人邮件

```java
public class QQMail {
    public static void main(String[] args) throws MessagingException {

        // 创建Properties 类用于记录邮箱的一些属性
        Properties props = new Properties();
        // 表示SMTP发送邮件，必须进行身份验证
        props.put("mail.smtp.auth", "true");
        //此处填写SMTP服务器
        props.put("mail.smtp.host", "smtp.qq.com");
        //端口号，QQ邮箱端口587
        props.put("mail.smtp.port", "587");
        // 此处填写，写信人的账号
        props.put("mail.user", "xxxxx@qq.com");
        // 此处填写16位STMP口令
        props.put("mail.password", "xxxx");

        // 构建授权信息，用于进行SMTP进行身份验证
        Authenticator authenticator = new Authenticator() {

            protected PasswordAuthentication getPasswordAuthentication() {
                // 用户名、密码
                String userName = props.getProperty("mail.user");
                String password = props.getProperty("mail.password");
                return new PasswordAuthentication(userName, password);
            }
        };
        // 使用环境属性和授权信息，创建邮件会话
        Session mailSession = Session.getInstance(props, authenticator);
        // 创建邮件消息
        MimeMessage message = new MimeMessage(mailSession);
        // 设置发件人
        InternetAddress form = new InternetAddress(props.getProperty("mail.user"));
        message.setFrom(form);

        // 设置收件人的邮箱
        String toMail = "xxxxx@qq.com";
        InternetAddress to = new InternetAddress(toMail);
        message.setRecipient(Message.RecipientType.TO, to);

        // 设置邮件标题
        message.setSubject("标题");

        // 设置邮件的内容体
        message.setContent("内容", "text/html;charset=UTF-8");

        // 最后当然就是发送邮件啦
        Transport.send(message);
    }
}
```

---

# 二：SpringBoot的JavaMailSenderImpl发送

##### 1.在resources下新建mailConfig.properties

```properties
mailConfig.properties
#服务器
mailHost=smtp.163.com
#端口号
mailPort=25
#邮箱账号(使用者改成自己的)
mailUsername=xxxxxx@163.com
#邮箱密码(使用者改成自己的)
mailPassword=xxxxx
#时间延迟
mailTimeout=25000
```

##### 2.新建一个类去读这个配置文件的内容

```java

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

/**
 * Create by Martin 2018/5/28/028 21:00
 */
public class MailConfig {
    private static final String PROPERTIES_DEFAULT = "mailConfig.properties";
    public static String host;
    public static Integer port;
    public static String userName;
    public static String passWord;
    public static String emailForm;
    public static String timeout;
    public static String personal;
    public static Properties properties;
    static{
        init();
    }

    /**
     * 初始化
     */
    private static void init() {
        properties = new Properties();
        try{
            InputStream inputStream = MailConfig.class.getClassLoader().getResourceAsStream(PROPERTIES_DEFAULT);
            properties.load(inputStream);
            inputStream.close();
            host = properties.getProperty("mailHost");
            port = Integer.parseInt(properties.getProperty("mailPort"));
            userName = properties.getProperty("mailUsername");
            passWord = properties.getProperty("mailPassword");
            emailForm = properties.getProperty("mailUsername");
            timeout = properties.getProperty("mailTimeout");
            personal = "来自星星的我";//发送人
        } catch(IOException e){
            e.printStackTrace();
        }
    }
}
```

##### 3.以上只是准备阶段，下面开始邮件编码

```java

import com.example.config.MailConfig;
import org.springframework.mail.javamail.JavaMailSenderImpl;
import org.springframework.mail.javamail.MimeMessageHelper;

import javax.mail.MessagingException;
import javax.mail.internet.MimeMessage;
import java.io.UnsupportedEncodingException;
import java.util.Properties;

/**
 * Create by Martin 2018/5/28/028 21:05
 */
public class MailUtil {
    private static final String HOST = MailConfig.host;
    private static final Integer PORT = MailConfig.port;
    private static final String USERNAME = MailConfig.userName;
    private static final String PASSWORD = MailConfig.passWord;
    private static final String emailForm = MailConfig.emailForm;
    private static final String timeout = MailConfig.timeout;
    private static final String personal = MailConfig.personal;
    private static JavaMailSenderImpl mailSender = createMailSender();
    /**
     * 邮件发送器
     *
     * @return 配置好的工具
     */
    private static JavaMailSenderImpl createMailSender() {
        JavaMailSenderImpl sender = new JavaMailSenderImpl();
        sender.setHost(HOST);
        sender.setPort(PORT);
        sender.setUsername(USERNAME);
        sender.setPassword(PASSWORD);
        sender.setDefaultEncoding("Utf-8");
        Properties p = new Properties();
        p.setProperty("mail.smtp.timeout", timeout);
        p.setProperty("mail.smtp.auth", "false");
        sender.setJavaMailProperties(p);
        return sender;
    }

    /**
     * 发送邮件
     *
     * @param to 接受人
     * @param subject 主题
     * @param html 发送内容
     * @throws MessagingException 异常
     * @throws UnsupportedEncodingException 异常
     */
    public static void sendMail(String to, String subject, String html) throws MessagingException,UnsupportedEncodingException {
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        // 设置utf-8或GBK编码，否则邮件会有乱码
        MimeMessageHelper messageHelper = new MimeMessageHelper(mimeMessage, true, "UTF-8");
        messageHelper.setFrom(emailForm, personal);
        messageHelper.setTo(to);
        messageHelper.setSubject(subject);
        messageHelper.setText(html, true);
        mailSender.send(mimeMessage);
    }
```

##### 4.开始发送一封邮件

```java
public static void main(String[] args){
    try {
        MailUtil.sendMail("xxxx@qq.com", "标题", "内容");
    } catch (MessagingException e) {
        e.printStackTrace();
    } catch (UnsupportedEncodingException e) {
        e.printStackTrace();
    }
}
```
