---
layout: post
title: nginx logrotate without root
date: 2025-02-19 21:14:00
last_modified_at : 2025-02-19 21:14:00
parent: Nginx
grand_parent: Msa
has_children: false
nav_exclude: true
tags: [nginx log]
---

# 요구사항

- root 계정 접근이 안되는 서버에 nginx log파일 관리가 필요한 상황
    - logrotate 등록자체는 root권한아니여도 가능하다. 다만 /etc/logrotate.d/nginx 하위에는 생성할 수 없다.
- 로그파일이 일자전환되지 않고 누적되는 상황
- 오래된 로그파일은 주기적으로 자동 삭제가 필요하다.
- 서버 호출이 많기 때문에, 파일용량을 고려해서 압축되어 관리될 필요가 있다.
- 참고) root계정 사용이 가능하다면 logrotate설정만으로도 충분히 가능하다.

# 작업내용

1. root 권한 없이 일반 계정에서 설정하는 방법
    - `logrotate` 을 적용할 로그 정책 설정 파일을 작성한다.
        - 파일위치는 참조가능한 위치면 무관하다.
        - 주의할 점은 conf확장자가 아닌 일반 파일로 작성한다.
            - conf파일로 만들면 logrotate로 인식하기 때문에 발생
            - 에러
                
                ```prolog
                nginx: [emerg] unknown directive "/app/nginx/logs/*.log" in /app/nginx/conf.d/nginx_logrotate.conf:1
                ```
                
        - 예)
            - **error.log.2025-01-15.log.gz 로 로테이트되며 30일 후 삭제된다**
            
            ```prolog
            /app/nginx/logs/*.log {
              create 0640 nginx nginx
              daily
              missingok
              rotate 30
              compress
              delaycompress
              notifempty
              dateformat .%Y-%m-%d.log
              dateext
              sharedscripts
              postrotate
                /app/nginx/sbin/nginx -s reload
              endscript
            }
            ```
            
2. crontab을 이용한 자동 실행
    - logrotate를 사용할 수 없기 때문에, crontab으로 등록시켜서 스케쥴링되게 한다.
    - `crontab -e`로 스케쥴 정보 저장
        
        ```prolog
        0 0 * * * /usr/sbin/logrotate -s /app/nginx/logs/nginx-logrotate.status /app/nginx/conf.d/nginx_logrotate
        ```
        
        - 0시마다 일정 주기로 실행

3. 설정 체크
    - 제대로 설정했는지 실행시켜본다.
    
    ```prolog
    logrotate -f /app/nginx/conf.d/nginx_logrotate
    ```
    
    - 한번 실행하면, 날짜변환되어 생성되기때문에 다음날 access.log.2025-01-15.log.2025-02-08.log 이런식으로 전환된다. 그래서 테스트 후 삭제한다.

4. 정상 동작 체크
    - cron log를 통해 크론탭이 정상 실행되는지 확인하는 방법
5. 추가 보안 조치
    - 위에서 생성한 nginx_logrotate파일의 읽기,실행 권한을 제한 (예 644)