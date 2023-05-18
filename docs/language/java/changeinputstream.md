---
layout: post
title: change ByteArrayInputStream to InputStreamResource
date: 2022-12-27 17:07:00
last_modified_at : 2023-01-07 17:07:00
parent: Java
grand_parent: Language
has_children: false
nav_exclude: true
---

# problem

- client로부터 수신된 첨부파일 url을 `MultipartFile`파일로 변환한 후 임시 저장하는 과정에  ByteArrayInputStream을 생성해서 하고 있었습니다.
- ByteArrayInputStream은 buffer가 아닌 byte[]에 데이터를 넣어두고 있는 것인데, 주로 물리적 파일로써 저장하기 전에 임시로 갖고 있다가 동적으로 변환하기 용이하기 때문에 사용했는데요
- 단점이 파일 사이즈만큼 메모리에 갖고 있어야 해서 문제가 될 소지가 많습니다.
    - 단, 힙메모리보다 커지게 갖고 있을 수는 없습니다
    - 이러한 단점은 ByteArrayOutputStream도 마찬가지

# try 1. send inputStream using feignClient

- 앞서 말한 것처럼 multipart로 첨부파일을 전송할 때 byteArray를 메모리에 적재해두는 문제 때문에 inputStream자체를 전송하는 것으로 변경하고 있는데
- 기존에는 feignClient를 통해서 multipart로 전송하고 있었습니다. 이를 inputStream전송으로 바꾸기 위해 inputStream을 map으로 담아서 전송하려고 했는데요.

```java
// FeignCLient
@FeignClient(
    contextId = "KakaoUploadClient",
    url = "${crema.thirdparty.kakao.url.upload}",
    name = "${crema.gateway.name}",
    configuration = {MultipartSupportConfig.class, FeignRetryConfiguration.class},
    fallbackFactory = KakaoUploadClient.KakaoUploadClientAFallback.class,
    primary = false
)
public interface KakaoUploadClient{
    @PostMapping(value = "attach", consumes = MULTIPART_FORM_DATA_VALUE)
    KakaoImageUploadRdo uploadImage(@RequestPart(value = "file") Map file);

    @Slf4j
    @Component
    class KakaoUploadClientAFallback implements FallbackFactory<KakaoUploadClient> {
        @Override
        public KakaoUploadClientcreate(Throwable throwable) {
            log.error(throwable.getMessage());
            return null;
        }
    }
}

// upload service - feignclient call
public class KakaoImageUploadFlowService  {
    private final KakaoUploadClient kakaoUploadClient;

    public KakaoImageUploadRdo uploadImage(@Valid KakaoWriteMessageCdo kakaoWriteMessageCdo, KakaoImageMessageCdo imageMessageCdo) {
        try {
            Resource resource = getFileResource(imageMessageCdo.getFileId());

            MultiValueMap<String, Object> bodyMap = new LinkedMultiValueMap<>();
            bodyMap.add("image", new FilenameAwareInputStreamResource(resource.getInputStream(), resource.getFilename(), resource.contentLength()));
            return kakaoUploadClient.uploadImage(kakaoWriteMessageCdo.getSender_key(), bodyMap);
        } catch (Exception e) {
            log.error("Failed to upload image.", e);
        }
        return KakaoImageUploadRdo.EMPTY;
    }
}
```

- 문제는 feignclient를 통해서 inputstream을 보낼 수 없다는 것. body에 map으로 전달해도 RequestTemplate에서 resolve할때 이미 body는 비어있었습니다.

![cbti1](../img/cbti1.png)

- fallbackfactory를 추가해봤습니다만, 200으로 응답이 오기 때문에 fallback에도 걸리지 않습니다.

![cbti2](../img/cbti2.png)

- nginx에도 request body크기가  61바이트로만 찍혀있네요.

![cbti3](../img/cbti3.png)

# try2. send inputStream using restTemplate

- restTemplate 클라이언트용 서비스를 생성했습니다.

```java
// RestTemplate client
public class KakaoUploadTemplateClient {
    private final RestTemplate restTemplate;

    private final LoadBalancerClient loadBalancerClient;

    public <T> T request(MultiValueMap<String, Object> requestMap, Class<T> clazz) {
        return restTemplate.postForEntity(
            uploadUrl(requestMap),
            new HttpEntity<>(requestMap, getHeaders()),
            clazz
        ).getBody();
    }

    private HttpHeaders getHeaders() {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.MULTIPART_FORM_DATA);
        return headers;
    }
}

// upload service - resttemplate call
public class KakaoImageUploadFlowService  {
    private final KakaoUploadTemplateClient kakaoUploadTemplateClient;

    public KakaoImageUploadRdo uploadImage(@Valid KakaoWriteMessageCdo kakaoWriteMessageCdo, KakaoImageMessageCdo imageMessageCdo) {
        try {
            Resource resource = getFileResource(imageMessageCdo.getFileId());

            MultiValueMap<String, Object> bodyMap = new LinkedMultiValueMap<>();
            bodyMap.add("image", new FilenameAwareInputStreamResource(resource.getInputStream(), resource.getFilename(), resource.contentLength()));
            return kakaoUploadTemplateClient.request(bodyMap, KakaoImageUploadRdo.class);
        } catch (Exception e) {
            log.error("Failed to upload image.", e);
        }
        return KakaoImageUploadRdo.EMPTY;
    }
}
```

# Try 3. send inputStream using Feign again..

- restTemplate을 통해 inputStream을 전송하는 것은 잘 되었지만, 곧 deprecated되는 restTemplate을 계속 사용할 수 없습니다.
- 아직 테스트해보지 않았지만, HttpClient를 커스텀으로 생성한 후 Feign에 주입하는 방식으로 작업해볼 생각입니다. httpmime의 `MultipartEntityBuilder`을 사용한다면 inputStream을 바이너리로 전송이 가능하니까 될것도 같습니다.

# APM 비교 - jvm모니터링

- AS-IS 
    - 첨부파일 메시지 인입했을 때, 힙메모리

![cbti](../img/cbti.png)



# reference

- [https://bortfolio.tistory.com/152#1.3. 전](https://bortfolio.tistory.com/152#1.3.%20%EC%A0%84)
- 허니몬님의 블로그를 참조해서 적용했습니다. [https://gist.github.com/ihoneymon/836cd6ca162cc2b436e70a3cbd035760](https://gist.github.com/ihoneymon/836cd6ca162cc2b436e70a3cbd035760)



