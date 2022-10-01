---
layout: post
title: change netflix-ribbon to spring-cloud-loadbalancer
date: 2022-10-01 09:48:00
last_modified_at : 2022-10-01 09:48:00
parent: SpringCloud
grand_parent: Msa
nav_exclude: true
tags: [spring cloud, spring boot]
---

# summary

- 기존에 eureka에 등록되지 않은 서비스에 대한 loadbalance처리를 ribbon을 통해 처리하고 있었는데, eos되면서 spring-cloud-loadbalancer로 대체하게 되었습니다.
- 상세 코드는 github에 올려두겠습니다(아직 작업중..)

# 내용

1. sprig-cloud-gateway는 eureka에 올라간 서비스만 가능하기 때문에, registry에 등록되지 않은 서비스는 spring-cloud-loadbalancer를 통해 서비스 등록을 시켜서 loadbalancing시킵니다.
2. ribbon으로 구성했던 작업내용에서 변경작업합니다

[Feign LB using nginx (without Eureka)](https://tnfhrnsss.github.io/docs/msa/feign/feignandribbon/)

## register service

- nginx를 통해서 외부API를 호출하는 서비스를 springbootapplication이 올라갈 때 등록하는 작업입니다.

### 1. replace dependency

- 스프링 버전 변경
- netflix-ribbon 제거
- loadbalancer dependency 추가
    
    ```yaml
    <dependency>
    	  <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>
    ```
    

### 2. register LoadBalancerClient

1. `LoadBalancerClientConfigurationRegistrar`에 서비스를 등록시키기 위해 @LoadBalancerClient를 선언한 클래스 생성
    1. @FeignClient 인터페이스 파일에 직접 선언해도 되지만, 저는 여러개의 FeignClient에서 공통으로 동작시키기 위해 별도의 클래스로 분리했습니다. 
        
        ```prolog
        import org.springframework.cloud.loadbalancer.annotation.LoadBalancerClient;
        
        @LoadBalancerClient(name="${crema.gateway.name}", configuration = {FeignLBInstanceConfigurationWithHealthCheck.class})
        public class FeignLBClientConfiguration {
        
        }
        ```
        
    2. LoadBalancerClient를 열어보면 @Configuration이 선언되어있는데요. 그래서 굳이 @FeignClient클래스에 개별적으로 선언할 필요없이 특정 클래스에 별도로 분리해주면 빈에 자동으로 등록됩니다.
2. LoadBalancerClient에 configuration으로 설정한 FeignLBInstanceConfigurationWithHealthCheck.java입니다.
    1. serviceId는 여러군데에서 동일하게 사용하는 값이라서 yml로 옮겼습니다.
        
        ```java
        public class FeignLBInstanceConfigurationWithHealthCheck {
            @Value("${crema.gateway.name:}")
            private String serviceId;
        
            @Bean
            ServiceInstanceListSupplier serviceInstanceListSupplier(LoadBalancerClientFactory loadBalancerClientFactory, @Value("${crema.gateway.instances}") String instances) {
                return new HealthCheckServiceInstanceListSupplier(
                    InstanceConfigurationHelper.getServiceInstanceListSuppliers(serviceId, StringUtils.tokenizeToStringArray(instances, ",")),
                    loadBalancerClientFactory,
                    healthCheckFunction(new RestTemplate())
                );
            }
        ... (중략)...
        }
        
        public class InstanceConfigurationHelper {
            static ServiceInstanceListSupplier getServiceInstanceListSuppliers(String serviceId, String[] instances) {
                return ServiceInstanceListSuppliers.from(
                    serviceId,
                    Arrays.stream(instances)
                        .map(Host::new)
                        .map(host -> new DefaultServiceInstance(host.toString(), serviceId, host.host, host.port, false))
                        .toArray(ServiceInstance[]::new)
                );
            }
        }
        ```
        
    2. healthcheck로 supplier를 선택한 이유는 여러 대의 서버 중 가용한 instance로만 서비스하기 위함입니다.
    3. HealthCheckServiceInstanceListSupplier로 구현하지 않으면, 서버상태를 보지않고 round robin시키기 때문에 내부적으로는 실패가 발생한 이후에 retry하게 됩니다.
    4. 자세한 내용은 spring docs 확인 ([https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#instance-health-check-for-loadbalancer](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#instance-health-check-for-loadbalancer))
3. InstanceConfigurationHelper 에서 supplier에 lb시킬 서버정보를 등록합니다 
    1. 서버정보는 yml에 선언

## Add healthcheck api

- loadbalancer를 healthcheck를 통해서 사용하기 위해서는 healthcheck정보가 필요합니다
- 만약에 설정이 따로 없다면, actuator/healthcheck를 기본으로 호출하게 됩니다.
- 저는 기존에 ribbon으로도 제공했던 pingurl이 있었기 때문에 그 설정 그대로 옮기기만 했습니다.
    
    ```yaml
    spring:
      cloud:
        loadbalancer:
          health-check:
            path:
              crema-gateway: /ping
            crema-gateway:
              refetch-instances: false
              # refetch-instances-interval: 60
              # repeat-health-check: false
    ```
    
- 실패 로그
    - 만약에 등록한 instance로 전송이 실패한 경우 connect refuse로 로깅됩니다. 서비스로 등록되어있지만 전송에 실패해서 그렇습니다. 그리고 나서
        
        ```prolog
        14:09:30.829 7-thread-5 DEBUG ockingLoadBalancerClient:130 ambda$execute$2 Service instance retrieved from LoadBalancedRetryContext: was null. Reattempting service instance selection
        14:09:30.829 7-thread-5 DEBUG ockingLoadBalancerClient:138 ambda$execute$2 Selected service instance: DefaultServiceInstance{instanceId='localhost-8051', serviceId='crema-gateway', host='localhost', port=8051, secure=false, metadata={}}
        14:09:30.830 7-thread-5 DEBUG ockingLoadBalancerClient:156 ambda$execute$2 Using service instance from LoadBalancedRetryContext: DefaultServiceInstance{instanceId='localhost-8051', serviceId='crema-gateway', host='localhost', port=8051, secure=false, metadata={}}
        14:09:35.366 7-thread-5 DEBUG f.s.Slf4jLogger         : 72 log             [KakaoWriteClient#write] <--- ERROR HttpHostConnectException: Connect to localhost:8051 [localhost/127.0.0.1, localhost/0:0:0:0:0:0:0:1] failed: Connection refused: connect (4538ms)
        14:09:35.367 7-thread-5 DEBUG f.s.Slf4jLogger         : 72 log             [KakaoWriteClient#write] org.apache.http.conn.HttpHostConnectException: Connect to localhost:8051 [localhost/127.0.0.1, localhost/0:0:0:0:0:0:0:1] failed: Connection refused: connect
        	at org.apache.http.impl.conn.DefaultHttpClientConnectionOperator.connect(DefaultHttpClientConnectionOperator.java:156)
        	at org.apache.http.impl.conn.PoolingHttpClientConnectionManager.connect(PoolingHttpClientConnectionManager.java:376)
        	at org.apache.http.impl.execchain.MainClientExec.establishRoute(MainClientExec.java:393)
        	at org.apache.http.impl.execchain.MainClientExec.execute(MainClientExec.java:236)
        	at org.apache.http.impl.execchain.ProtocolExec.execute(ProtocolExec.java:186)
        	at org.apache.http.impl.execchain.RetryExec.execute(RetryExec.java:89)
        	at org.apache.http.impl.execchain.RedirectExec.execute(RedirectExec.java:110)
        	at org.apache.http.impl.client.InternalHttpClient.doExecute(InternalHttpClient.java:185)
        	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:83)
        	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:108)
        	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:56)
        	at feign.httpclient.ApacheHttpClient.execute(ApacheHttpClient.java:83)
        	at org.springframework.cloud.openfeign.loadbalancer.LoadBalancerUtils.executeWithLoadBalancerLifecycleProcessing(LoadBalancerUtils.java:57)
        	at org.springframework.cloud.openfeign.loadbalancer.RetryableFeignBlockingLoadBalancerClient.lambda$execute$2(RetryableFeignBlockingLoadBalancerClient.java:167)
        	at org.springframework.retry.support.RetryTemplate.doExecute(RetryTemplate.java:329)
        	at org.springframework.retry.support.RetryTemplate.execute(RetryTemplate.java:225)
        	at org.springframework.cloud.openfeign.loadbalancer.RetryableFeignBlockingLoadBalancerClient.execute(RetryableFeignBlockingLoadBalancerClient.java:113)
        	at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:119)
        	at feign.SynchronousMethodHandler.invoke(SynchronousMethodHandler.java:89)
        	at org.springframework.cloud.openfeign.FeignCircuitBreakerInvocationHandler.lambda$asSupplier$1(FeignCircuitBreakerInvocationHandler.java:128)
        	at java.util.concurrent.FutureTask.run$$$capture(FutureTask.java:266)
        	at java.util.concurrent.FutureTask.run(FutureTask.java)
        	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        	at java.lang.Thread.run(Thread.java:748)
        Caused by: java.net.ConnectException: Connection refused: connect
        	at java.net.DualStackPlainSocketImpl.waitForConnect(Native Method)
        	at java.net.DualStackPlainSocketImpl.socketConnect(DualStackPlainSocketImpl.java:85)
        	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
        	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
        	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
        	at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:172)
        	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
        	at java.net.Socket.connect(Socket.java:589)
        	at org.apache.http.conn.socket.PlainConnectionSocketFactory.connectSocket(PlainConnectionSocketFactory.java:75)
        	at org.apache.http.impl.conn.DefaultHttpClientConnectionOperator.connect(DefaultHttpClientConnectionOperator.java:142)
        	... 24 more
        
        14:09:35.370 7-thread-5 DEBUG f.s.Slf4jLogger         : 72 log             [KakaoWriteClient#write] <--- END ERROR
        ```
        
    - 두 번째로 다시 호출하면 UnknownHostException이 발생합니다.
        
        ```prolog
        14:11:16.528 7-thread-6 DEBUG ockingLoadBalancerClient:130 ambda$execute$2 Service instance retrieved from LoadBalancedRetryContext: was null. Reattempting service instance selection
        14:11:16.528 7-thread-6 WARN  c.RoundRobinLoadBalancer: 98 nstanceResponse No servers available for service: crema-gateway
        14:11:16.528 7-thread-6 DEBUG ockingLoadBalancerClient:138 ambda$execute$2 Selected service instance: null
        14:11:16.528 7-thread-6 WARN  ockingLoadBalancerClient:145 ambda$execute$2 Service instance was not resolved, executing the original request
        14:11:18.806 7-thread-6 DEBUG f.s.Slf4jLogger         : 72 log             [KakaoWriteClient#write] <--- ERROR UnknownHostException: crema-gateway (2279ms)
        14:11:18.807 7-thread-6 DEBUG f.s.Slf4jLogger         : 72 log             [KakaoWriteClient#write] java.net.UnknownHostException: crema-gateway
        	at java.net.Inet6AddressImpl.lookupAllHostAddr(Native Method)
        	at java.net.InetAddress$2.lookupAllHostAddr(InetAddress.java:929)
        	at java.net.InetAddress.getAddressesFromNameService(InetAddress.java:1324)
        	at java.net.InetAddress.getAllByName0(InetAddress.java:1277)
        	at java.net.InetAddress.getAllByName(InetAddress.java:1193)
        	at java.net.InetAddress.getAllByName(InetAddress.java:1127)
        	at org.apache.http.impl.conn.SystemDefaultDnsResolver.resolve(SystemDefaultDnsResolver.java:45)
        	at org.apache.http.impl.conn.DefaultHttpClientConnectionOperator.connect(DefaultHttpClientConnectionOperator.java:112)
        	at org.apache.http.impl.conn.PoolingHttpClientConnectionManager.connect(PoolingHttpClientConnectionManager.java:376)
        	at org.apache.http.impl.execchain.MainClientExec.establishRoute(MainClientExec.java:393)
        	at org.apache.http.impl.execchain.MainClientExec.execute(MainClientExec.java:236)
        	at org.apache.http.impl.execchain.ProtocolExec.execute(ProtocolExec.java:186)
        	at org.apache.http.impl.execchain.RetryExec.execute(RetryExec.java:89)
        	at org.apache.http.impl.execchain.RedirectExec.execute(RedirectExec.java:110)
        	at org.apache.http.impl.client.InternalHttpClient.doExecute(InternalHttpClient.java:185)
        	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:83)
        	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:108)
        	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:56)
        	at feign.httpclient.ApacheHttpClient.execute(ApacheHttpClient.java:83)
        	at org.springframework.cloud.openfeign.loadbalancer.LoadBalancerUtils.executeWithLoadBalancerLifecycleProcessing(LoadBalancerUtils.java:57)
        	at org.springframework.cloud.openfeign.loadbalancer.RetryableFeignBlockingLoadBalancerClient.lambda$execute$2(RetryableFeignBlockingLoadBalancerClient.java:167)
        	at org.springframework.retry.support.RetryTemplate.doExecute(RetryTemplate.java:329)
        	at org.springframework.retry.support.RetryTemplate.execute(RetryTemplate.java:225)
        	at org.springframework.cloud.openfeign.loadbalancer.RetryableFeignBlockingLoadBalancerClient.execute(RetryableFeignBlockingLoadBalancerClient.java:113)
        ```
        
    - exception에 대한 retry가 호출되어야합니다. 이건 다른 post로 정리하겠습니다.

## feignClient

1. 기존 FeignClient로 구현한 인터페이스는 변경사항이 없습니다.
2. 구현한 파일은 아래 정보로 되어있습니다
    1. ribbon으로 구현했던 포스트에 설명했듯이, 사이트별로 서버 및 방화벽 상황에 맞게 중간에 dmz를 통해 제어를 해야한다고 하면 name을 통해서 gateway서비스를 하게 하고 
    2. 그게 아니면, 외부API의 url에 direct로 통신하게 할 수 있고
    3. L4가 있으면 그또한 url을 통해서 통신하게 할 수도 있습니다
    4. 아래처럼 name과 url이 모두 설정된 경우, url이 빈값이면 name을 통해서 서비스하게 됩니다. 그래서 yml설정내용에 따라 유동적으로 사용할 수 있습니다.
    
    ```yaml
    @FeignClient(
        contextId = "KakaoWriteClient",
        url = "${crema.thirdparty.kakao.url.chat}",
        name = "${crema.gateway.name}",
        configuration = {FeignConfiguration.class, FeignRetryConfiguration.class},
        primary = false
    )
    public interface KakaoWriteClient {
        @PostMapping("test")
        KakaoWriteRdo send();
    }
    ```
    

## Modify yml file

1. 기존에 ribbon설정으로 되어있던 부분을 제거하고 loadbalancer설정으로 대체합니다.
2. [https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#instance-health-check-for-loadbalancer](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#instance-health-check-for-loadbalancer)
3. 설정한 serviceId의 health-check를 추가합니다. (없으면 actuator/healthcheck로 호출됨)
4. instance정보가 변경될 때 refetch할 수 있도록 설정할 수 있습니다.(refetch로 시작하는 설정)
5. repeat-health-check는 refetch가 일어날 때마다 health-check를 한다는 것입니다.
    1. refetch를 false로 해도 instance가 절체되었을 때 재시도하므로 false로 해도 됩니다

```yaml
spring:
  cloud:
    loadbalancer:
      enabled: true
      retry:
        enabled: true
        maxRetriesOnSameServiceInstance: 0
        maxRetriesOnNextServiceInstance: 2
        retryableStatusCodes: 502, 503
      health-check:
        path:
          crema-gateway: /ping
        crema-gateway:
          refetch-instances: false
          # refetch-instances-interval: 60
          # repeat-health-check: false
```

# About ping

- ribbon의 경우는 PingUrl을 상속받아 커스텀 구현이 가능했습니다.
    
    ```java
    import com.netflix.loadbalancer.PingUrl;
    import com.netflix.loadbalancer.Server;
    import org.springframework.beans.factory.annotation.Value;
    
    public class FeignRibbonPing extends PingUrl {
        @Value("${crema.thirdparty.thirdparty-gateway.health-check-location:}")
        private String healthCheckLocation;
    
        @Value("${crema.thirdparty.thirdparty-gateway.isSecure:false}")
        private boolean isHealthCheckSecure;
    
        @Override
        public String getPingAppendString() {
            return healthCheckLocation;
        }
    
        @Override
        public boolean isAlive(Server server) {
            super.setSecure(isHealthCheckSecure);
            return super.isAlive(server);
        }
    }
    ```
    
- scl의 경우는 찾아봤지만, 없는 것으로 보아 위에 HealthCheckServiceInstanceListSupplier으로 대체되는 것 같습니다
- 꼭 필요하다고 하면 instance의 serviceId를 얻어와서 HealthCheckServiceInstanceListSupplier에 등록한 `healthCheckFunction`을 통해 구현하는 방법 뿐인 것 같습니다.

# 기타

- loadbalancer의 log를 보고 싶다면, 추가
    
    ```yaml
    logging:
      level:
        org.springframework.cloud.openfeign.loadbalancer: debug
    ```
    
- loadbalancer policy에 대해
    - 찾아보니, scl의 경우 round-robin 또는 random 방식만 지원한다고 합니다.
    - 기존에 ribbon의 경우는 `com.netflix.loadbalancer.AvailabilityFilteringRule`을 사용했었는데요.  위에서 작업한 heathcheck가 비슷하다고 할 수 있겠습니다.
    - custom한 rule을 적용하고 싶다면 아래 내용을 참고해서 작업하면 됩니다.
        - [https://blog.cnscud.com/springcloud/2021/06/07/springcloud-betazonedemo-6.html](https://blog.cnscud.com/springcloud/2021/06/07/springcloud-betazonedemo-6.html)

# problems

- 일부러 가용하지 않은 서버를 instance로 설정하지 않겠지만, 
만약에 여러대의 instance 중 가용하지 않은 서버가 있으면, 
처음에 서비스 올리고 최초로 인스턴스를 호출하면, 2~3초정도 걸립니다.
그 이후로는 가용한 서버 정보가 메모리에 저장되어 있기 때문에 밀리세컨드안에 실행되지만
최초로 전송할 땐, 실패할 경우에 대한 healthcheck를 yml에 설정한 값으로 수행합니다.
그래서 그 실패에 대한 delay가 발생합니다.
    
    ```java
    2022-09-29 14:53:21.632 DEBUG 25112 --- [pool-1-thread-1] RetryableFeignBlockingLoadBalancerClient : Service instance retrieved from LoadBalancedRetryContext: was null. Reattempting service instance selection
    2022-09-29 14:53:21.661 ERROR 25112 --- [nio-9097-exec-2] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() for servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is org.springframework.cloud.client.circuitbreaker.NoFallbackAvailableException: No fallback available.] with root cause
    
    java.util.concurrent.TimeoutException: TimeLimiter 'WorldClient#hello(String)' recorded a timeout exception.
    	at java.util.concurrent.FutureTask.get(FutureTask.java:205) ~[na:1.8.0_202]
    	at io.github.resilience4j.timelimiter.internal.TimeLimiterImpl.lambda$decorateFutureSupplier$0(TimeLimiterImpl.java:46) ~[resilience4j-timelimiter-1.7.0.jar:1.7.0]
    	at io.github.resilience4j.circuitbreaker.CircuitBreaker.lambda$decorateCallable$3(CircuitBreaker.java:171) ~[resilience4j-circuitbreaker-1.7.0.jar:1.7.0]
    	at io.vavr.control.Try.of(Try.java:75) ~[vavr-0.10.2.jar:na]
    	at org.springframework.cloud.circuitbreaker.resilience4j.Resilience4JCircuitBreaker.run(Resilience4JCircuitBreaker.java:123) ~[spring-cloud-circuitbreaker-resilience4j-2.1.3.jar:2.1.3]
    	at org.springframework.cloud.client.circuitbreaker.CircuitBreaker.run(CircuitBreaker.java:30) ~[spring-cloud-commons-3.1.3.jar:3.1.3]
    	at org.springframework.cloud.openfeign.FeignCircuitBreakerInvocationHandler.invoke(FeignCircuitBreakerInvocationHandler.java:107) ~[spring-cloud-openfeign-core-3.1.3.jar:3.1.3]
    
    2022-09-29 14:53:25.163 DEBUG 25112 --- [pool-1-thread-1] RetryableFeignBlockingLoadBalancerClient : Selected service instance: DefaultServiceInstance{instanceId='localhost-9093', serviceId='world-no-eureka', host='localhost', port=9093, secure=false, metadata={}}
    2022-09-29 14:53:25.163 DEBUG 25112 --- [pool-1-thread-1] RetryableFeignBlockingLoadBalancerClient : Using service instance from LoadBalancedRetryContext: DefaultServiceInstance{instanceId='localhost-9093', serviceId='world-no-eureka', host='localhost', port=9093, secure=false, metadata={}}
    ```
    
- 

# reference

[[SC14] Spring Cloud LoadBalancer 란 ?](https://happycloud-lee.tistory.com/221)