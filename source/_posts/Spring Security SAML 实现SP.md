---
title: Spring Security SAML 实现SP
date: 2017-10-20 10:56:00
categories:
- WEB技术
tags:
- Spring Security
- SAML
---

### Spring Security SAML 实现SP

#### 下载 sample application

	https://github.com/spring-projects/spring-security-saml

#### 配置IDP metadata

	修改 /src/main/webapp/WEB-INF/securityContext.xml，告诉系统下载IDP metadata 从给定的url, 超时5s

``` xml
<bean id="metadata" class="org.springframework.security.saml.metadata.CachingMetadataManager"> 
  <constructor-arg> 
    <list> 
      <bean class="org.opensaml.saml2.metadata.provider.HTTPMetadataProvider"> 
        <constructor-arg> 
          <value type="java.lang.String">http://idp.ssocircle.com/idp-meta.xml</value> 
        </constructor-arg>  
        <constructor-arg> 
          <value type="int">5000</value> 
        </constructor-arg>  
        <property name="parserPool" ref="parserPool"/> 
      </bean> 
    </list> 
  </constructor-arg> 
</bean>
```
<!--more-->
#### 配置SP metadata
	修改  src/main/webapp/WEB-INF/securityContext.xml 
``` xml
<bean id="metadataGeneratorFilter" class="org.springframework.security.saml.metadata.MetadataGeneratorFilter"> 
  <constructor-arg> 
    <bean class="org.springframework.security.saml.metadata.MetadataGenerator"> 
      <property name="entityId" value="urn:test:winchannel:beijing"/>  
      <!-- 文档中是 <property name="signMetadata" value="false"/>,需改为requestSigned -->  
      <property name="requestSigned" value="false"/> 
    </bean> 
  </constructor-arg> 
</bean>
```

#### 上传SP metadata到IDP


启动项目后,访问http://localhost/spring-security-saml2-sample/saml/metadata下载生成好的 sp metadata，访问 https://idp.ssocircle.com/sso/UI/Login (建议使用IE，其他浏览器有可能注册不成功) 注册一个新账号，Manage Metadata  -> Add new Service Provider Enter the FQDN of the ServiceProvider :  输入 entityId  urn:test:winchannel:beijing 输入 metadata information。

#### 测试

访问 http://localhost/spring-security-saml2-sample/ 即被要求跳转到 https://idp.ssocircle.com/ 认证。

> 参考: http://docs.spring.io/spring-security-saml/docs/1.0.0.RELEASE/reference/html/chapter-quick-start.html