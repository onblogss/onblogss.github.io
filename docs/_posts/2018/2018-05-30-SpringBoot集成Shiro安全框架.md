---
layout: post
title: SpringBoot集成Shiro安全框架
date: 2018-05-30 13:37:00
author: 薛师兄
tags: SpringBoot
---
##### 跟着我的步骤：先运行起来再说

Spring集成Shiro的GitHub：[https://github.com/onblog/shiro-imooc](https://github.com/onblog/shiro-imooc)

---

# 一：导包

```xml
<!-- Shiro安全框架 -->
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.4.0</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-web</artifactId>
    <version>1.4.0</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.4.0</version>
</dependency>
```

# 二：ShiroConfig配置

```java

import com.example.shiro.CustomRealm;
import org.apache.shiro.authc.credential.HashedCredentialsMatcher;
import org.apache.shiro.spring.LifecycleBeanPostProcessor;
import org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor;
import org.apache.shiro.spring.web.ShiroFilterFactoryBean;
import org.apache.shiro.web.mgt.CookieRememberMeManager;
import org.apache.shiro.web.mgt.DefaultWebSecurityManager;
import org.apache.shiro.web.servlet.SimpleCookie;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * Create by Martin 2018/5/17/017 20:18
 */
@Configuration
public class ShiroConfig {
    /**
     * 记住我：自动登录-1
     */
    @Bean
    public SimpleCookie getSimpleCookie(){
        SimpleCookie simpleCookie = new SimpleCookie();
        simpleCookie.setName("rememberMe");
        simpleCookie.setMaxAge(20000000);
        return simpleCookie;
    }

    /**
     * 记住我：自动登录-2
     */
    @Bean
    public CookieRememberMeManager getCookieRememberMeManager(){
        CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
        cookieRememberMeManager.setCookie(getSimpleCookie());
        return cookieRememberMeManager;
    }

    /**
     * 开启MD5加密
     * @return
     */
    @Bean
    public HashedCredentialsMatcher getMatcher(){
        HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
        matcher.setHashAlgorithmName("md5");
        matcher.setHashIterations(1);
        return matcher;
    }

    /**
     * 自定义Realm密码验证与加密
     * @return
     */
    @Bean
    public CustomRealm getCustomRealm(){
      CustomRealm customRealm = new CustomRealm();
      customRealm.setCredentialsMatcher(getMatcher());
      return customRealm;
    }

    /**
     * 创建SecurityManager环境
     * @return
     */
    @Bean
    public DefaultWebSecurityManager getSecurityManager(){
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(getCustomRealm());
        securityManager.setRememberMeManager(getCookieRememberMeManager());
        return securityManager;
    }

    /**
     * Shiro在Web项目中的过滤
     * @return
     */
    @Bean
    public ShiroFilterFactoryBean getfilterFactoryBean(){
        ShiroFilterFactoryBean filterFactoryBean = new ShiroFilterFactoryBean();
        filterFactoryBean.setSecurityManager(getSecurityManager());
        /**
         *  只有在下面配置路径访问权限，Shiro才会执行自动跳转。
         *  如果使用Shiro注解权限，就只会报异常，
         *  就只能采用统一异常处理的方法。
         */
        //拦截器.
        Map<String,String> filterChainDefinitionMap = new LinkedHashMap<String,String>();
        // 配置不会被拦截的链接 顺序判断
        filterChainDefinitionMap.put("/static/**", "anon");
        //配置退出 过滤器,其中的具体的退出代码Shiro已经替我们实现了
        filterChainDefinitionMap.put("/user/exit", "logout");
        //<!-- 过滤链定义，从上向下顺序执行，一般将/**放在最为下边 -->:这是一个坑呢，一不小心代码就不好使了;
        //<!-- authc:所有url都必须认证通过才可以访问; anon:所有url都都可以匿名访问-->
        filterChainDefinitionMap.put("/user/user", "authc");
        // 如果不设置默认会自动寻找Web工程根目录下的"/login.jsp"页面
        filterFactoryBean.setLoginUrl("/user/login");
        // 登录成功后要跳转的链接
        filterFactoryBean.setSuccessUrl("/");
        //未授权界面;
        filterFactoryBean.setUnauthorizedUrl("/user/login");
        filterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return filterFactoryBean;
    }


    /**
     * 保证Shiro的声明周期
     * @return
     */
    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor(){
        return new LifecycleBeanPostProcessor();
    }

    /**
     * 开启Shiro授权生效
     * @return
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(){
        return new AuthorizationAttributeSourceAdvisor();
    }

}
```

# 三：自定义授权类

```java
import com.example.service.user.UserService;
import org.apache.shiro.authc.AuthenticationException;
import org.apache.shiro.authc.AuthenticationInfo;
import org.apache.shiro.authc.AuthenticationToken;
import org.apache.shiro.authc.SimpleAuthenticationInfo;
import org.apache.shiro.authz.AuthorizationInfo;
import org.apache.shiro.authz.SimpleAuthorizationInfo;
import org.apache.shiro.realm.AuthorizingRealm;
import org.apache.shiro.subject.PrincipalCollection;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Lazy;

import java.util.Set;

/**
 * Create by Martin 2018/5/17/017 20:38
 */
public class CustomRealm extends AuthorizingRealm {

    private String ClassName =this.getClass().getName();

    @Autowired
    @Lazy   //必须懒加载，否则Ehcache缓存注解及事务管理注解无效
    private UserService userService;

    {
        super.setName(ClassName);
    }

    /**
     * 权限处理
     * @param principals
     * @return
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        String username = (String) principals.getPrimaryPrincipal();
        // 从数据库或者缓存中获得角色数据
        Set<String> roles = userService.getRolesByUserName(username);
        Set<String> permissions = userService.getPermissionsByRoles(roles);
        //上面的service层方法需要自己写
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        simpleAuthorizationInfo.setStringPermissions(permissions);
        simpleAuthorizationInfo.setRoles(roles);

        return simpleAuthorizationInfo;
    }

    /**
     * 认证处理
     * @param token
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        // 1.从主体传过来的认证信息中，获得用户名
        String username = (String) token.getPrincipal();

        // 2.通过用户名到数据库中获取凭证
        String password = userService.getPasswordByUsername(username);
        if(password == null) {
            return null;
        }
        SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(username, password, ClassName);
        return simpleAuthenticationInfo;
    }

}
```

# 四：使用

## 1）登录验证：

```java
@RequestMapping(value = "/login/result", method = RequestMethod.POST)
public String userLogin(User user) {
    String error = null;
    Subject subject = SecurityUtils.getSubject();
    UsernamePasswordToken token = new UsernamePasswordToken(user.getMail(), user.getPassword());
    try {
        token.setRememberMe(user.isRememberMe());//记住我
        subject.login(token);
    } catch (UnknownAccountException e) {
        error = "用户名/密码错误";
    } catch (IncorrectCredentialsException e) {
        error = "用户名/密码错误";
    } catch (AuthenticationException e) {
        //其他错误，比如锁定，如果想单独处理请单独catch处理
        error = "其他错误：" + e.getMessage();
    }
    if(error != null) {//出错了，返回登录页面

    } else {//登录成功

    }
}
```

## 2）权限验证：

*  可以在配置文件中配置Url权限等，参考步骤二。

*  使用诸如    @RequiresRoles(“user”) 注解在controller层的方法上，进行角色验证，或者使用@RequiresPermissions(“index:hello”)进行权限验证，不过使用注解Shiro就只抛出异常，无法使用shiro设置自动跳转到页面等。针对这个问题，可以用@ControllerAdvice统一异常处理。

```java
@RequiresRoles("user")
@RequestMapping(value = "/up")
public String up(){

}
```

---

本实例旨在讲解SpringBoot集成，Shiro教程推荐：[http://wiki.jikexueyuan.com/project/shiro/](http://wiki.jikexueyuan.com/project/shiro/)