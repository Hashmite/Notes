# Notes
Persion's wiki



## yml 配置 log4j2
```
logging:
  config: classpath:log4j2-spring.xml
  level:
    root: INFO
    javax.activation: info
    org.apache.catalina: INFO
    org.apache.commons.beanutils.converters: INFO
    org.apache.coyote.http11.Http11Processor: INFO
    org.apache.http: INFO
    org.apache.tomcat: INFO
    org.springframework: INFO
    com.chinamobile.cmss.bdpaas.resource.monitor: DEBUG
 ```

