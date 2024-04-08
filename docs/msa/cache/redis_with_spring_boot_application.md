---
layout: post
title: redis with spring boot application
description: redis with spring boot application
date: 2024-04-08 11:08:00
last_modified_at : 2024-04-08 11:08:00
parent: Cache
grand_parent: Msa
nav_exclude: true
---

# 요구 사항

- github commit이력을 3분마다 조회해서, redis에 저장한다.
- redis에 저장된 정보를 기준으로 co-component-change, co-change 위반건을 검출

# 왜 Redis인가?

- 제품에 사용되는 기술로 포함되어 있지 않아서 한번 사용해보고 싶었다. 🤭
- 생각보다 설치도 간단했고, spring 통합도 복잡하지 않았다!!
- 설치할 서버에 이미 hazelcast가 사용중이여서 다른 캐시 구현체로 해보는 것도 운영할 때 장애전파가 적을 것이라고 판단했다.

# 준비

## 1. Redis 설치

- [https://velog.io/@juno0713/Windows-환경에서-Docker에-Redis-설치](https://velog.io/@juno0713/Windows-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-Docker%EC%97%90-Redis-%EC%84%A4%EC%B9%98){:target="_blank"}를 참고

```bash
$ docker version
$ docker pull redis
```

- 실행

```bash
$ docker run --name redis-latest -d -p 6379:6379 redis
```

## 2. redis-cli 접속 방법

```bash
$ docker exec -it 0eb966e6477f redis-cli
명령어
keys *
flushall
```

# 구현

### 1. 프로젝트 생성 및 redis정보 설정

- spring boot application을 생성하고 application.java에 **`@EnableCaching`**을 추가한다.
- application.yml에 redis정보 설정
    
    ```yaml
    spring:
      data:
        redis:
          host: localhost
          port: 6379
    ```
    

### 2. 샘플 pojo 클래스 생성

- redis에 json형태로 저장하고 파싱하기 위한 모델 클래스를 생성한다.

```java
@Getter
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class CoComponentChangeTargetSdo implements JsonSerializable {
    private String taskId;

    private String commitId;

    private String project;

    public static CoComponentChangeTargetSdo fromJson(String json) {
        return JsonParser.fromJson(json, CoComponentChangeTargetSdo.class);
    }

    @Override
    public String toString() {
        return toJson();
    }
}
```

### 3. redis configuration 클래스 생성

- key-value의 serializer를 설정한다.
    - StringRedisSerializer : 문자열 키로 생성
    - GenericJackson2JsonRedisSerializer : 위에서 생성한 데이터 모델 클래스 CoComponentChangeTargetSdo를 json형태로 해서 value를 직렬화
    
    ```java
    @Configuration
    public class RedisConfiguration {
        @Value("${spring.data.redis.host:localhost}")
        private String redisHost;
    
        @Value("${spring.data.redis.port:6379}")
        private int redisPort;
    
        @Bean
        public RedisConnectionFactory redisConnectionFactory() {
            return new LettuceConnectionFactory(redisHost, redisPort);
        }
    
        @Bean
        public RedisTemplate<String, Object> redisTemplate() {
            RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
            redisTemplate.setConnectionFactory(redisConnectionFactory());
            redisTemplate.setKeySerializer(new StringRedisSerializer());
            redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
    
            return redisTemplate;
        }
    
        @Bean
        public RedisCacheManager redisCacheManager(RedisConnectionFactory redisConnectionFactory) {
            RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
                .serializeKeysWith(RedisSerializationContext
                    .SerializationPair
                    .fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext
                    .SerializationPair
                    .fromSerializer(new GenericJackson2JsonRedisSerializer()));
    
            return RedisCacheManager
                    .RedisCacheManagerBuilder
                    .fromConnectionFactory(redisConnectionFactory)
                    .cacheDefaults(redisCacheConfiguration)
                    .build();
        }
    }
    ```
    

### 4. 데이터 저장

- github의 커밋로그에 있는 jira Id를 조합해서 key로 한다.

```java
@CachePut(value = "bteam-ccp-ccc-target-find", key = "#coComponentChangeTargetSdo.getTaskId().concat('_').concat(#coComponentChangeTargetSdo.getCommitId())")
public CoComponentChangeTargetSdo register(CoComponentChangeTargetSdo coComponentChangeTargetSdo) {
    return coComponentChangeTargetSdo;
}
```

### 5. redis 데이터 조회 서비스 작성

- 서비스 단

```java
@Cacheable(value = "bteam-ccp-ccc-target-find", key = "#coComponentChangeTargetSdo.getTaskId().concat('_').concat(#coComponentChangeTargetSdo.getCommitId())")
public CoComponentChangeTargetSdo find(CoComponentChangeTargetSdo coComponentChangeTargetSdo) {
    throw new NoSuchElementException("bteam-ccp-ccc-target-find not found. :" + coComponentChangeTargetSdo);
}
```

```java
public List<CoComponentChangeTargetSdo> findAll() {
    try {
      return cacheService.getAllCacheKeys("bteam-ccp-ccc-target-find")
          .stream()
          .map(value -> coComponentChangMapService.find(CoComponentChangeTargetSdo.fromJson(value.toString())))
                .collect(Collectors.toList());
        } catch (NoSuchElementException e) {
            return Collections.EMPTY_LIST;
        }
}
```

- 캐시 조회 서비스단

```java
public List<Object> getAllCacheKeys(String cacheName) {
        List<Object> keys = new ArrayList<>();

        Cache cache = cacheManager.getCache(cacheName);
        if (cache != null) {
            if (cache.getNativeCache() instanceof ConcurrentHashMap) {
                for (Object key : ((ConcurrentHashMap<?, ?>) cache.getNativeCache()).keySet()) {
                    keys.add(key);
                }
            } else {
                Set<String> redisKeys = redisTemplate.keys("*");
                redisKeys
                    .stream()
                    .filter(k -> k.contains(cacheName))
                    .forEach(k -> {
                        keys.add(redisTemplate.opsForValue().get(k));
                        log.debug("forEach key: {}-{}-{}-{}", cacheName, k, k.contains(cacheName), k.startsWith(cacheName));
                    });
            }
        }
        return keys;
    }
```

- key를 조회하는 이 부분에서 좀 많이 삽질했다..
- 전체 keys를 조회해서 contains로 String값을 필터하는 것으로 되어있는데, 애초에 key생성할 때부터 setKeySerializer에 String으로 하면 안되었을 것 같다. ;ㅁ;
    - contains는 안쓰고 싶었다….😭

# Redis 조회 snapshot

- 차곡차곡 저장되어있다.

![../img/redis_with_spring_boot_application.png](../img/redis_with_spring_boot_application.png)

# Reference

- [https://velog.io/@eeun95/Redis를-사용해보자-1-feat.-비관계형-데이터-베이스란](https://velog.io/@eeun95/Redis%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%B4%EB%B3%B4%EC%9E%90-1-feat.-%EB%B9%84%EA%B4%80%EA%B3%84%ED%98%95-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%B2%A0%EC%9D%B4%EC%8A%A4%EB%9E%80){:target="_blank"}
- [https://jane096.github.io/project/redis-caching-part2/](https://jane096.github.io/project/redis-caching-part2/){:target="_blank"}