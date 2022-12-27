---
layout: post
title: change ByteArrayInputStream to InputStreamResource
date: 2022-12-27 17:07:00
last_modified_at : 2022-12-27 17:07:00
parent: Java
grand_parent: Msa
has_children: false
nav_exclude: true
---

# problem

- client로부터 수신된 첨부파일 url을 `MultipartFile`파일로 변환한 후 임시 저장하는 과정에  ByteArrayInputStream을 생성해서 하고 있었습니다.
- ByteArrayInputStream은 buffer가 아닌 byte[]에 데이터를 넣어두고 있는 것인데, 주로 물리적 파일로써 저장하기 전에 임시로 갖고 있다가 동적으로 변환하기 용이하기 때문에 사용했는데요
- 단점이 파일 사이즈만큼 메모리에 갖고 있어야 해서 문제가 될 소지가 많습니다.
    - 단, 힙메모리보다 커지게 갖고 있을 수는 없습니다
    - 이러한 단점은 ByteArrayOutputStream도 마찬가지

# 기존 코드

- feignClient를 통해서 파일을 바이트로 읽어온 후 ByteArrayInputStream로 변환해서 사용하고 있었습니다.

```java
// 1. ByteArrayMultipartFile 클래스를 유틸성으로 생성하고 (일부생략한 코드)
public class ByteArrayMultipartFile implements MultipartFile, JsonSerializable {
    private final byte[] content;

    private String name;

    private String originalFilename;

    @Override
    public byte[] getBytes() throws IOException {
        return content;
    }

    @Override
    public InputStream getInputStream() throws IOException {
        return new ByteArrayInputStream(content);
    }
}

// 2. client로부터 전달받은 파일url을 가져와서 MultipartFile로 변환
private MultipartFile getMultipartFile(FileCdo fileCdo) {
    FileByUrlCdo fileByUrlCdo = (FileByUrlCdo) fileCdo;
    String fileUrl = fileByUrlCdo.getUrl();
    ResponseEntity<byte[]> responseEntity = getResponseEntity(fileUrl);
    String fileName = fileUrl.substring(fileUrl.lastIndexOf('/')+ 1);
    return new ByteArrayMultipartFile(responseEntity.getBody(), "file", fileName);
}

// 3. feignclient를 통해서 첨부파일 url의 ResponseEntity를 가져온다.
private ResponseEntity<byte[]> getResponseEntity(String fileUrl) {
    return thirdPartyGatewayFileClient.download(fileUrl);
}
```

- APM - jvm모니터링
    - 첨부파일 메시지 인입했을 때, 힙메모리

![cbti](../img/cbti.png)


# 변경 작업

- 허니몬님의 블로그를 참조해서 적용했습니다. [https://gist.github.com/ihoneymon/836cd6ca162cc2b436e70a3cbd035760](https://gist.github.com/ihoneymon/836cd6ca162cc2b436e70a3cbd035760)
- 상세 구현

```java
// 1. InputStreamResource을 상속받은 클래스 생성
public class FileInputStreamResource extends InputStreamResource {
    private final String filename;
    private final long contentLength;

    public FileInputStreamResource(InputStream inputStream, long contentLength, String filename) {
        super(inputStream);
        this.filename = filename;
        this.contentLength = contentLength;
    }

    @Override
    public String getFilename() {
        return filename;
    }

    public long getContentLength() {
        return contentLength;
    }

    @Override
    public boolean exists() {
        return super.exists();
    }
}

//
private FileInputStreamResource getMultipartFile(FileCdo fileCdo) {
    FileByUrlCdo fileByUrlCdo = (FileByUrlCdo) fileCdo;
    String fileUrl = fileByUrlCdo.getUrl();
    ResponseEntity<Resource> responseEntity = getResponseEntity(fileUrl);
    String fileName = fileUrl.substring(fileUrl.lastIndexOf('/')+ 1);
    Resource resource = Objects.requireNonNull(responseEntity.getBody());

    try {
        return new FileInputStreamResource(resource.getInputStream(), resource.contentLength(), fileName);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}

```

- 리팩토링이 필요해보이긴 하지만, 서버에 선적용 후 테스트해봤습니다.
- 적용 후 APM 모니터링
    - 첨부파일 몇 개로 진행한 테스트라 개선점이 눈에 띄게 나타나는 것은 아니지만, 최악의 상황은 발생하지 않을 것 같다는 생각입니다.

![cbti1](../img/cbti1.png)