---
layout: post
title: Cannot instantiate @InjectMocks field named
date: 2024-05-01 23:15:00
last_modified_at : 2024-05-01 23:15:00
parent: TestCase
grand_parent: Quality Practice
nav_exclude: true
---

# 현상

- jenkins로 junit test실행할 때 에러 발생
- intellij로 maven install했을 때는 발생하지 않는다.

```bash
Cannot instantiate @InjectMocks field named '' of type 'class '. 
You haven't provided the instance at field declaration so I tried to construct the instance. 
However the constructor or the initialization block threw an exception : null
```

# 코드

```java
@Component
public class FileResourceRepository {
    private final Path fileStorageLocation;

    public FileResourceRepository(
        final String uploadDir
    ) {
        this.fileStorageLocation = Paths.get(uploadDir, "test").toAbsolutePath().normalize();
    }

    public void saveStorage(String filePath) {
        try {
            Files.createDirectories(fileStorageLocation.resolve(filePath));
        } catch (IOException e) {
            throw new FileSystemWriteException("Create test repository fails: " + filePath + " when storage created");
        }
    }
}
```

# 테스트 코드

```java
@ExtendWith(MockitoExtension.class)
class FileResourceRepositoryTest {
	@InjectMocks
    private FileResourceRepository fileResourceRepository;

    @Mock
    private Path fileStorageLocation;

    private final String filePath = "/tmp/uploads";

    @BeforeEach
    void setup() {
        fileResourceRepository = new FileResourceRepository(filePath);
    }

    @Test
    void saveFileSystemWriteException() throws IOException {
        Resource resource = mock(Resource.class);
        when(resource.getInputStream()).thenReturn(mock(InputStream.class));

        Executable executable = () -> fileResourceRepository.save(FileMetaKey.sample(), resource);
        assertThrows(FileSystemWriteException.class, executable);
    }
```

# 원인

- 에러 메시지만 보면, @InjectMocks로 선언한 fileResourceRepository를 초기화할 때 npe가 발생했다는 것
- 테스트하려는 클래스에 정의한 생성자 주입과정 없이 초기화를 해서 발생한 것으로
- [https://4whomtbts.tistory.com/129](https://4whomtbts.tistory.com/129)

# 해결

- @InjectMocks를 제거
- initMocks(this); 추가해서 초기화하고
- 생성자에 Mock객체를 주입

```java
@ExtendWith(MockitoExtension.class)
class FileResourceRepositoryTest {
    private FileResourceRepository fileResourceRepository;

    @Mock
    private Path fileStorageLocation;

    private final String filePath = "/tmp/uploads";

    @BeforeEach
    void setup() {
        initMocks(this);
        fileResourceRepository = new FileResourceRepository(filePath);
    }

```