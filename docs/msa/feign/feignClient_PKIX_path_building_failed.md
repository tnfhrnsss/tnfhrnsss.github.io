---
layout: post
title: Feign Client PKIX path building failed
date: 2024-03-02 10:50:00
last_modified_at : 2024-03-02 10:50:00
parent: Feign
grand_parent: Msa
nav_exclude: true
tags: [feign, ssl]
---

# Problem

- feign Client를 통해 https 주소의 API를 호출하면, “PKIX path building failed” 에러가 발생

# Error

```java
spring boot feignclient PKIX path building failed: 
sun.security.provider.certpath.SunCertPathBuilderException: 
unable to find valid certification path to requested 
target executing GET
```

# Cause

- client 서버에서 API 서버에 적용된 ssl을 신뢰하지 않기 때문에 발생한 것으로
    - ssl만료했거나, 사설 인증서이거나 등등의 이유로 발생할 수 있다
- 호출하려는 API 서버에 적용된 ssl은 Sectigo였는데, 로컬에서 zulu 17로 호출했을 때 인증실패한 것

# Solution

## 1안) JVM에 인증서 추가

- jvm의 cacerts파일에 API 서버의 ssl인증서를 추가한다.
- [https://lifeinprogram.tistory.com/37](https://lifeinprogram.tistory.com/37){:target="_blank"}에서 자세히 설명하고 있음

```java
keytool -import -trustcacerts -keystore $JAVA_HOME/jre/lib/security/cacerts -storepass changeit -noprompt -alias your-api-alias -file your-api-certificate.pem
```

- 하지만 이 방법은 java를 여러 서비스에서 사용하는 경우에는 위험할 수 있다.

## 2안) **어플리케이션 내에서 trustcacerts하도록 설정**

- feign client의 configuration으로 해당 인증서를 신뢰할 수 있도록 한다.
- [https://stackoverflow.com/a/67964403/14257397](https://stackoverflow.com/a/67964403/14257397){:target="_blank"}에서 자세히 설명하고 있다.
- 내가 갖고 있던 인증서는 crt파일이였기 때문에 일단 keystore로 변경했다
    
    ```java
    keytool -import -file _wildcard_spectra_co_kr.crt -keystore _wildcard_spectra_co_kr.jks -alias spectra
    ```
    
- yml에 trustStore, trustStorePassword를 설정한다.
    
    ```java
    monitoring:
      argus:
        api:
          url: https://~~~.com
          ssl:
            trustStore: "/Users/jhsim/Documents/aaa.jks"
            trustStorePassword: 
    ```
    
- feign client config java 생성
    - 코드는 stackoverflow내용과 같다.
    - 그 중 Resource → fileSystemResource로 변경했다.
    
    ```java
    public class FeignTrustCertConfig {
        @Bean
        public Client feignClient(
            @Value("${monitoring.argus.api.ssl.trustStore}") String trustStoreResource,
            @Value("${monitoring.argus.api.ssl.trustStorePassword}") String trustStorePassword) 
            throws Exception {
            try {
                Resource resource = new FileSystemResource(trustStoreResource);
    
                return new Client.Default(
                        sslSocketFactory(resource.getInputStream(), trustStorePassword),
                        null);
            } catch (Exception e) {
                throw new Exception("Error in initializing feign client", e);
            }
        }
    
        private static SSLSocketFactory sslSocketFactory
            InputStream trustStoreStream, String trustStorePassword)
                throws NoSuchAlgorithmException, KeyStoreException, CertificateException, IOException, KeyManagementException {
            SSLContext sslContext = SSLContext.getInstance("TLSv1.2");
            TrustManagerFactory tmf = createTrustManager(trustStoreStream, trustStorePassword);
            sslContext.init(new KeyManager[]{}, tmf.getTrustManagers(), null);
            return sslContext.getSocketFactory();
        }
    
        private static TrustManagerFactory createTrustManager(
            InputStream trustStoreStream, String trustStorePassword)
                throws KeyStoreException, IOException, NoSuchAlgorithmException, CertificateException {
            KeyStore trustStore = KeyStore.getInstance(KeyStore.getDefaultType());
            trustStore.load(trustStoreStream, trustStorePassword.toCharArray());
            TrustManagerFactory tmf = TrustManagerFactory
                    .getInstance(TrustManagerFactory.getDefaultAlgorithm());
            tmf.init(trustStore);
            return tmf;
        }
    }
    
    ```
    
- 위에서 만든 클래스를 feign client에 configuration으로 주입했다.
    
    ```java
    @FeignClient(
        contextId = "spectra.attic.tools.ccp.monitoring.argus.client.ArgusCoChangesClient",
        url = "${monitoring.argus.api.url}",
        name = "argus",
        configuration = {FeignTrustCertConfig.class, FeignLoggerLevelConfiguration.class},
        primary = false
    )
    public interface ArgusCoChangesClient {
    ```
