# Actuator

```
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

```
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

```
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
#info.build.artifactId=@project.artifactId@
#info.build.version=@project.version@

```

### 安全
如果用了 Security，客户端里要么把 "/actuator/**" 设置为不需要权限限制，
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

如果需要设置在 SBA 上访问客户端配置页面权限，配置：（注意这和上面的访问访客端的账号不一样）
```
spring.boot.admin.client.username=sba
spring.boot.admin.client.password=password
```

> 注：如果使用了权限访问，SBA 会持续不断的访问用户登入，可能会影响性能，需要加上缓存（getUserByUsername）

其他
```
# spring boot admin
#spring.boot.admin.client.url=http://localhost:8080/admin
#spring.boot.admin.url=http://localhost:8080/admin
#spring.boot.admin.context-path=/admin
```

