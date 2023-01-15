---
layout: post
title: spring boot application running too slow
date: 2022-12-18 15:12:00
last_modified_at : 2022-12-18 15:12:00
parent: Spring
grand_parent: Msa
nav_exclude: true
---

# problem

- build, install, load도 느리지만,
- spring boot start가 제일 느리다.
    - 최고 800초를 넘는다.
    - 평균 500~600초

# Cause

## Try1.

```yaml
spring.jpa.properties.hibernate.temp.use_jdbc_metadata_defaults=false 
```
Started MochaApplication in 469.81 seconds (JVM running for 475.449)

## Try2.
chage db : mysql to h2

started MochaApplication in 466.076 seconds (JVM running for 471.271)

## Try3.

1. host에 127을 추가하라는 글 (x)→ 이미 있음
2. plugin이 문제일 수 있다고 해서 안쓰는 플러그인 disable처리(x)
3. hibernate validator를 사용안함으로 설정하라는 글 (x)
    1. application.yml에 아래 설정 추가
    
    ```yaml
    spring:
      jpa:
        properties:
          javax:
            persistence:
              validation:
                mode: none
    ```
    
4. start할때 before build옵션 제거
5. run java를 java8에서 java17로 변경 - 아주 약간 빨라짐

## Try4. add java option

- disable launch optimization 추가
- disable jmx agent 옵션 추가

![spring_boot_1](../img/spring_boot_1.png)

- result
    - as-is : 평균 550초
    - to-be: 평균 400초

## Try5. change server (tomcat → undertow)

- pom.xml 변경
    
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-tomcat</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-undertow</artifactId>
    </dependency>
    ```
    
- application.yml 변경
    
    ```yaml
    server:
      undertow:
        threads:
          io: 8
          worker: 200
    ```
    
- result
    - 454 ~ 471초
    - 경량이라 기대했는데, 빨라지지 않음

## Try6.

- spring.cloud.bus.autoconfigure.exclude 설정하고 lazyInit 적용
- bootstrap.yml
    
    ```yaml
    spring:
      cloud:
        bus:
          autoconfigure:
            exclude:
              - com.ulisesbocchio.jasyptspringbootstarter.JasyptSpringBootAutoConfiguration
              - io.github.resilience4j.bulkhead.autoconfigure.BulkheadAutoConfiguration
              - io.github.resilience4j.bulkhead.autoconfigure.BulkheadMetricsAutoConfiguration
              - io.github.resilience4j.bulkhead.autoconfigure.ThreadPoolBulkheadMetricsAutoConfiguration
              - io.github.resilience4j.circuitbreaker.autoconfigure.CircuitBreakerAutoConfiguration
              - io.github.resilience4j.circuitbreaker.autoconfigure.CircuitBreakerMetricsAutoConfiguration
              - io.github.resilience4j.circuitbreaker.autoconfigure.CircuitBreakerStreamEventsAutoConfiguration
              - io.github.resilience4j.circuitbreaker.autoconfigure.CircuitBreakersHealthIndicatorAutoConfiguration
              - io.github.resilience4j.ratelimiter.autoconfigure.RateLimiterAutoConfiguration
              - io.github.resilience4j.ratelimiter.autoconfigure.RateLimiterMetricsAutoConfiguration
              - io.github.resilience4j.ratelimiter.autoconfigure.RateLimitersHealthIndicatorAutoConfiguration
              - io.github.resilience4j.retry.autoconfigure.RetryMetricsAutoConfiguration
              - io.github.resilience4j.scheduled.threadpool.autoconfigure.ContextAwareScheduledThreadPoolAutoConfiguration
              - io.github.resilience4j.timelimiter.autoconfigure.TimeLimiterAutoConfiguration
              - io.github.resilience4j.timelimiter.autoconfigure.TimeLimiterMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.amqp.RabbitHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.audit.AuditAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.audit.AuditEventsEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.availability.AvailabilityHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.availability.AvailabilityProbesAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.beans.BeansEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.cache.CachesEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.cassandra.CassandraHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.cassandra.CassandraReactiveHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.cloudfoundry.reactive.ReactiveCloudFoundryActuatorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.cloudfoundry.servlet.CloudFoundryActuatorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.condition.ConditionsReportEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.context.ShutdownEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.context.properties.ConfigurationPropertiesReportEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.couchbase.CouchbaseHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.couchbase.CouchbaseReactiveHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.elasticsearch.ElasticSearchReactiveHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.elasticsearch.ElasticSearchRestHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.endpoint.EndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.endpoint.jmx.JmxEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.env.EnvironmentEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.flyway.FlywayEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.hazelcast.HazelcastHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.health.HealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.health.HealthEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.influx.InfluxDbHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.info.InfoContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.info.InfoEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.integration.IntegrationGraphEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.jdbc.DataSourceHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.jms.JmsHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.jolokia.JolokiaEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.ldap.LdapHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.liquibase.LiquibaseEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.logging.LogFileWebEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.logging.LoggersEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.mail.MailHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.management.HeapDumpWebEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.management.ThreadDumpEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.JvmMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.KafkaMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.Log4J2MetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.LogbackMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.MetricsEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.SystemMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.amqp.RabbitMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.cache.CacheMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.data.RepositoryMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.appoptics.AppOpticsMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.atlas.AtlasMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.datadog.DatadogMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.dynatrace.DynatraceMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.elastic.ElasticMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.ganglia.GangliaMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.graphite.GraphiteMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.humio.HumioMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.influx.InfluxMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.jmx.JmxMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.kairos.KairosMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.newrelic.NewRelicMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.prometheus.PrometheusMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.signalfx.SignalFxMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.simple.SimpleMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.stackdriver.StackdriverMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.statsd.StatsdMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.export.wavefront.WavefrontMetricsExportAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.graphql.GraphQlMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.jdbc.DataSourcePoolMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.jersey.JerseyServerMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.mongo.MongoMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.orm.jpa.HibernateMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.r2dbc.ConnectionPoolMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.redis.LettuceMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.startup.StartupTimeMetricsListenerAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.task.TaskExecutorMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.web.client.HttpClientMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.web.jetty.JettyMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.web.reactive.WebFluxMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.web.servlet.WebMvcMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.metrics.web.tomcat.TomcatMetricsAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.mongo.MongoHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.mongo.MongoReactiveHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.neo4j.Neo4jHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.quartz.QuartzEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.r2dbc.ConnectionFactoryHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.redis.RedisHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.redis.RedisReactiveHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.scheduling.ScheduledTasksEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.security.reactive.ReactiveManagementWebSecurityAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.security.servlet.ManagementWebSecurityAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.solr.SolrHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.startup.StartupEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.system.DiskSpaceHealthContributorAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.trace.http.HttpTraceAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.trace.http.HttpTraceEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.web.mappings.MappingsEndpointAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.web.reactive.ReactiveManagementContextAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.web.server.ManagementContextAutoConfiguration
              - org.springframework.boot.actuate.autoconfigure.web.servlet.ServletManagementContextAutoConfiguration
              - org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration
              - org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration
              - org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration
              - org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration
              - org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration
              - org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration
              - org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration
              - org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration
              - org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration
              - org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration
              - org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration
              - org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRestClientAutoConfiguration
              - org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration
              - org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration
              - org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration
              - org.springframework.boot.autoconfigure.data.neo4j.Neo4jReactiveDataAutoConfiguration
              - org.springframework.boot.autoconfigure.data.neo4j.Neo4jReactiveRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.r2dbc.R2dbcDataAutoConfiguration
              - org.springframework.boot.autoconfigure.data.r2dbc.R2dbcRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration
              - org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration
              - org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration
              - org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration
              - org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration
              - org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration
              - org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration
              - org.springframework.boot.autoconfigure.graphql.GraphQlAutoConfiguration
              - org.springframework.boot.autoconfigure.graphql.data.GraphQlQueryByExampleAutoConfiguration
              - org.springframework.boot.autoconfigure.graphql.data.GraphQlQuerydslAutoConfiguration
              - org.springframework.boot.autoconfigure.graphql.data.GraphQlReactiveQueryByExampleAutoConfiguration
              - org.springframework.boot.autoconfigure.graphql.data.GraphQlReactiveQuerydslAutoConfiguration
              - org.springframework.boot.autoconfigure.graphql.reactive.GraphQlWebFluxAutoConfiguration
              - org.springframework.boot.autoconfigure.graphql.rsocket.GraphQlRSocketAutoConfiguration
              - org.springframework.boot.autoconfigure.graphql.rsocket.RSocketGraphQlClientAutoConfiguration
              - org.springframework.boot.autoconfigure.graphql.security.GraphQlWebFluxSecurityAutoConfiguration
              - org.springframework.boot.autoconfigure.graphql.security.GraphQlWebMvcSecurityAutoConfiguration
              - org.springframework.boot.autoconfigure.graphql.servlet.GraphQlWebMvcAutoConfiguration
              - org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration
              - org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration
              - org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration
              - org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration
              - org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration
              - org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration
              - org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration
              - org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration
              - org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration
              - org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration
              - org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration
              - org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration
              - org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration
              - org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration
              - org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration
              - org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration
              - org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration
              - org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration
              - org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration
              - org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration
              - org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration
              - org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration
              - org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration
              - org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration
              - org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration
              - org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration
              - org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration
              - org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration
              - org.springframework.boot.autoconfigure.neo4j.Neo4jAutoConfiguration
              - org.springframework.boot.autoconfigure.netty.NettyAutoConfiguration
              - org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration
              - org.springframework.boot.autoconfigure.r2dbc.R2dbcAutoConfiguration
              - org.springframework.boot.autoconfigure.r2dbc.R2dbcTransactionManagerAutoConfiguration
              - org.springframework.boot.autoconfigure.rsocket.RSocketMessagingAutoConfiguration
              - org.springframework.boot.autoconfigure.rsocket.RSocketRequesterAutoConfiguration
              - org.springframework.boot.autoconfigure.rsocket.RSocketServerAutoConfiguration
              - org.springframework.boot.autoconfigure.rsocket.RSocketStrategiesAutoConfiguration
              - org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration
              - org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration
              - org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration
              - org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration
              - org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration
              - org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration
              - org.springframework.boot.autoconfigure.security.rsocket.RSocketSecurityAutoConfiguration
              - org.springframework.boot.autoconfigure.security.saml2.Saml2RelyingPartyAutoConfiguration
              - org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration
              - org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration
              - org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration
              - org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration
              - org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration
              - org.springframework.boot.autoconfigure.sql.init.SqlInitializationAutoConfiguration
              - org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration
              - org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration
              - org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration
              - org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration
              - org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration
              - org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration
              - org.springframework.boot.autoconfigure.web.reactive.ReactiveMultipartAutoConfiguration
              - org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration
              - org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration
              - org.springframework.boot.autoconfigure.web.reactive.WebSessionIdResolverAutoConfiguration
              - org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration
              - org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration
              - org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration
              - org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration
              - org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
              - org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration
              - org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration
              - org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration
              - org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration
              - org.springframework.cloud.autoconfigure.RefreshAutoConfiguration
              - org.springframework.cloud.autoconfigure.RefreshEndpointAutoConfiguration
              - org.springframework.cloud.autoconfigure.WritableEnvironmentEndpointAutoConfiguration
              - org.springframework.cloud.bus.BusAutoConfiguration
              - org.springframework.cloud.bus.BusRefreshAutoConfiguration
              - org.springframework.cloud.bus.BusStreamAutoConfiguration
              - org.springframework.cloud.bus.PathServiceMatcherAutoConfiguration
              - org.springframework.cloud.bus.jackson.BusJacksonAutoConfiguration
              - org.springframework.cloud.circuitbreaker.resilience4j.ReactiveResilience4JAutoConfiguration
              - org.springframework.cloud.circuitbreaker.resilience4j.Resilience4JAutoConfiguration
              - org.springframework.cloud.client.CommonsClientAutoConfiguration
              - org.springframework.cloud.client.ReactiveCommonsClientAutoConfiguration
              - org.springframework.cloud.client.discovery.composite.CompositeDiscoveryClientAutoConfiguration
              - org.springframework.cloud.client.discovery.composite.reactive.ReactiveCompositeDiscoveryClientAutoConfiguration
              - org.springframework.cloud.client.discovery.simple.SimpleDiscoveryClientAutoConfiguration
              - org.springframework.cloud.client.discovery.simple.reactive.SimpleReactiveDiscoveryClientAutoConfiguration
              - org.springframework.cloud.client.hypermedia.CloudHypermediaAutoConfiguration
              - org.springframework.cloud.client.loadbalancer.AsyncLoadBalancerAutoConfiguration
              - org.springframework.cloud.client.loadbalancer.LoadBalancerDefaultMappingsProviderAutoConfiguration
              - org.springframework.cloud.client.loadbalancer.reactive.LoadBalancerBeanPostProcessorAutoConfiguration
              - org.springframework.cloud.client.loadbalancer.reactive.ReactorLoadBalancerClientAutoConfiguration
              - org.springframework.cloud.client.serviceregistry.AutoServiceRegistrationAutoConfiguration
              - org.springframework.cloud.client.serviceregistry.ServiceRegistryAutoConfiguration
              - org.springframework.cloud.commons.config.CommonsConfigAutoConfiguration
              - org.springframework.cloud.commons.security.ResourceServerTokenRelayAutoConfiguration
              - org.springframework.cloud.config.client.ConfigClientAutoConfiguration
              - org.springframework.cloud.configuration.CompatibilityVerifierAutoConfiguration
              - org.springframework.cloud.function.context.config.ContextFunctionCatalogAutoConfiguration
              - org.springframework.cloud.function.context.config.FunctionsEndpointAutoConfiguration
              - org.springframework.cloud.function.context.config.KotlinLambdaToFunctionAutoConfiguration
              - org.springframework.cloud.loadbalancer.config.LoadBalancerCacheAutoConfiguration
              - org.springframework.cloud.loadbalancer.config.LoadBalancerStatsAutoConfiguration
              - org.springframework.cloud.loadbalancer.security.OAuth2LoadBalancerClientAutoConfiguration
              - org.springframework.cloud.netflix.eureka.reactive.EurekaReactiveDiscoveryClientConfiguration
              - org.springframework.cloud.openfeign.hateoas.FeignHalAutoConfiguration
              - org.springframework.cloud.stream.binder.kafka.config.ExtendedBindingHandlerMappingsProviderConfiguration
              - org.springframework.cloud.stream.config.BindersHealthIndicatorAutoConfiguration
              - org.springframework.cloud.stream.config.BindingServiceConfiguration
              - org.springframework.cloud.stream.config.BindingsEndpointAutoConfiguration
              - org.springframework.cloud.stream.config.ChannelsEndpointAutoConfiguration
              - org.springframework.cloud.stream.function.FunctionConfiguration
    ```
    
- `LazyBeanConfig`
    - 일부 bean은 lazy로딩이 되게 설정
    
    ```java
    import java.util.Arrays;
    
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.annotation.Profile;
    import org.springframework.kafka.annotation.KafkaListener;
    
    @Configuration
    @Slf4j
    @Profile("lazyInit")
    public class LazyBeanConfig {
    
        @Bean
        public static BeanFactoryPostProcessor beanFactoryPostProcessor() {
            return beanFactory ->
                Arrays.stream(beanFactory.getBeanDefinitionNames())
                    .filter(LazyBeanConfig::filterLazyInit)
                    .forEach(beanName -> beanFactory.getBeanDefinition(beanName).setLazyInit(true));
        }
    
        private static boolean filterLazyInit(String beanName) {
            if (beanName.endsWith("Subscriber") || beanName.endsWith("Client")) {
                return false;
            } else {
                return true;
            }
        }
    
        private static boolean hasKafkaListenerAnnotation(String beanName) {
            try {
                KafkaListener annotation = Class.forName(beanName).getAnnotation(KafkaListener.class);
                return annotation != null;
            } catch (ClassNotFoundException e) {}
    
            return false;
        }
    }
    ```
    
- active profile로 `lazyInit` 추가
- result
    
    Started MochaApplication in 372.174 seconds (JVM running for 377.947)

# Resolved
결국 장비를 맥북으로 변경하고 91초...

## Reference

[https://stackoverflow.com/a/59654120/14257397](https://stackoverflow.com/a/59654120/14257397)