# Actuator

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

打开所有监控端点（默认就 Health 等几个）
```
management.endpoints.web.exposure.include=*
```
其他（之前的版本）
```
#management.security.enabled=false
#management.endpoints.enabled-by-default=true
management.endpoints.web.exposure.include=*
#endpoints.docs.curies.enabled=true
```

访问 http://localhost:8080/actuator

# Spring Boot Admin (SBA)

[Spring Boot Admin](https://github.com/joshiste/spring-boot-admin) 是一个第三方的 Spring Boot 运行监控和配置服务。目前版本 v2.0.0，支持 Spring Boot 2.0.x

## 服务端

```xml
<dependency>
	<groupId>de.codecentric</groupId>
	<artifactId>spring-boot-admin-starter-server</artifactId>
	<version>2.0.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

声明 `@EnableAdminServer`

换个端口 `server.port=8099`

### 邮件通知

[TBD](http://codecentric.github.io/spring-boot-admin/2.0.0/#mail-notifications)

```
#spring.mail.host=smtp.example.com
#spring.boot.admin.notify.mail.to=admin@example.com
```

### 集群

[TBD](http://codecentric.github.io/spring-boot-admin/2.0.0/#clustering-support)

## 客户端

```xml
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.0.0</version>
</dependency>
```

配置服务端地址
```
spring.boot.admin.client.url=http://localhost:8099
```

打开所有监控端点（默认就 Health 等几个）
```
management.endpoints.web.exposure.include=*
```

用 IP 注册（默认是机器名）
```
spring.boot.admin.client.instance.prefer-ip=true

```

设置一下自己的应用名
```
spring.application.name=WilmarBoot
```
还可以加上额外的 info
```
info.app.name=Wilmar Boot
info.app.version=1.0.0
info.build.artifactId=@project.artifactId@
info.build.version=@project.version@
spring.boot.admin.client.instance.name=${spring.application.name}
```

不需要了的话，关闭注册
```
spring.boot.admin.client.enabled=false
```

### 安全

#### 客户端
客户端如果用了 Security，客户端里要么把 "/actuator/**" 设置为不需要权限限制，
```
.antMatchers("/actuator/**").permitAll().and()
```
要么设置授权给 SBA（一定要打开 .httpBasic()）
```
.httpBasic()
```
配置用户名密码
```
spring.boot.admin.client.instance.metadata.user.name=yinguowei
spring.boot.admin.client.instance.metadata.user.password=111111
```

> 注：如果使用了客户端权限访问，SBA 会持续不断的访问用户登入，可能会影响性能，需要加上缓存（getUserByUsername）

#### 服务端
如果需要设置在 SBA 上设置登入认证
```xml
<dependency>
	<groupId>de.codecentric</groupId>
	<artifactId>spring-boot-admin-server-ui-login</artifactId>
	<version>1.5.7</version>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

配置 Security
```java
/**
 * @author Yin Guo Wei 2018/5/28.
 */
@Configuration
public class SecuritySecureConfig extends WebSecurityConfigurerAdapter {
    private final String adminContextPath;

    public SecuritySecureConfig(AdminServerProperties adminServerProperties) {
        this.adminContextPath = adminServerProperties.getContextPath();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // @formatter:off
        SavedRequestAwareAuthenticationSuccessHandler successHandler = new SavedRequestAwareAuthenticationSuccessHandler();
        successHandler.setTargetUrlParameter("redirectTo");

        http.authorizeRequests()
                .antMatchers(adminContextPath + "/assets/**").permitAll()
                .antMatchers(adminContextPath + "/login").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin().loginPage(adminContextPath + "/login").successHandler(successHandler).and()
                .logout().logoutUrl(adminContextPath + "/logout").and()
                .httpBasic().and()
                .csrf().disable();
        // @formatter:on
    }
}
```

配置用户名密码
```
@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
    auth
        .inMemoryAuthentication()
        .withUser(User.withDefaultPasswordEncoder().username("sbaadmin").password("password").roles("ADMIN"));
}
```

就需要在客户端设置：
```
spring.boot.admin.client.username=sbaadmin
spring.boot.admin.client.password=password
```


其他
```
# spring boot admin
#spring.boot.admin.client.url=http://localhost:8080/admin
#spring.boot.admin.url=http://localhost:8080/admin
#spring.boot.admin.context-path=/admin
```