## 针对springcloud框架的微服务，提供服务访问的版本控制，实现灰度发布

### 服务提供者

#### 1.pom引入依赖

    <dependency>
        <groupId>com.graydeploy.springcloud.versioning</groupId>
        <artifactId>springcloud-starter-versioning</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    
#### 2.配置文件配置

    eureka:
        instance:
            metadata-map:
                versions: release-20180301
    
至此服务提供者启动时以release-20180301版本提供服务
    
### 服务消费者

#### 1.pom引入依赖和添加启动类配置

    <dependency>
        <groupId>com.graydeploy.springcloud.versioning</groupId>
        <artifactId>springcloud-starter-versioning</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
    
    启动类添加注解，指定每个feignClient的ribbon配置，主要是rule规则
    @RibbonClients(value = {
        @RibbonClient(name = "provider",configuration = FeignClientConfigduration.class),
        @RibbonClient(name = "provider2",configuration = FeignClientConfigduration.class)
    })
    或通过配置指定  
        provider: 
            ribbon: 
                NFLoadBalancerRuleClassName: VersioningZoneAvoidanceRule（类全路径）  
        provider2: 
            ribbon: 
                NFLoadBalancerRuleClassName: VersioningZoneAvoidanceRule（类全路径）  
                
#### 2.提供多种版本控制策略

##### a. URL query参数控制版本 
配置：

    @Bean
    public RequestVersionExtractor requestVersionExtractor(){
        return new RequestVersionExtractor.Default();
    }
    
如http://127.0.0.1:8080/springcloud-provider/hello?rq_version=release-20180301
##### b. Header参数控制版本
配置：

    @Bean
    public RequestVersionExtractor requestVersionExtractor(){
        return new RequestVersionExtractor.RequestHeaderVersionExtractor();
    }
        
如请求服务时header添加 rq_version=release-20180301
            
##### c. 配置文件映射服务名和版本（默认这个策略）
配置文件添加配置文件：

    versioning:
      server-version:
        metadata:
          product-provider: 1.0
          dealer-provider: 2.0
          order-provider: 3.0
          
metadata后面的格式为{服务提供者的服务名}:{版本号}，如order-provider服务请求3.0版本的。
{服务提供者的服务名}即为provider的application-name也即@FeignClient的value，
@FeignClient(value = "order-provider",fallbackFactory = MyFallBackFactory.class)
                
策略a使用场景：通过API网关给第三方调用，放在URL上，清晰明了  
策略b使用场景：网页请求，只要塞过这个header后面的服务路由就会受版本号控制，缺陷是如果consumer需要调用N个服务且版本号各不相同，则无法满足  
策略c使用场景：版本号控制到服务应用级别，不同服务指定不同版本，更符合生产情况。
            

        
