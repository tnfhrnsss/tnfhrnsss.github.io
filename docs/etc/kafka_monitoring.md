---
layout: post
title: kafka monitoring
date: 2022-03-28 22:56:00
last_modified_at : 2022-03-28 22:56:00
parent: Etc
has_children: false
nav_exclude: true
tags: [kafka, grafana, burrow, telegraf, elasticsearch]
---


### 구성

kafka + burrow + telegraf + es

### kafka와 es

- kafka와 es는 이미 설치되어있는 환경

### burrow 설치 및 실행

- burrow 설치
    - go언어 설치
    - burrow설치
        - git받은담에, go언어로 설치
    - burrow.toml 설정
        - 포트가 중복되어서 8200으로 변경
        - kafka랑 zookeeper정보 변경
    
    ```
    [general]
    pidfile="burrow.pid"
    stdout-logfile="burrow.out"
    access-control-allow-origin="mysite.example.com"
    
    [logging]
    filename="logs/burrow.log"
    level="info"
    maxsize=100
    maxbackups=30
    maxage=10
    use-localtime=false
    use-compression=true
    
    [zookeeper]
    servers=[ "localhost:2181" ]
    timeout=6
    root-path="/burrow"
    
    [client-profile.local]
    client-id="burrow-local"
    kafka-version="2.13.0"
    
    [cluster.local]
    class-name="kafka"
    servers=[ "localhost:9092" ]
    client-profile="local"
    topic-refresh=120
    offset-refresh=30
    groups-reaper-refresh=0
    
    [consumer.local]
    class-name="kafka"
    cluster="local"
    servers=[ "localhost:9092" ]
    client-profile="local"
    group-denylist="^(console-consumer-|python-kafka-consumer-|quick-).*$"
    group-allowlist=""
    
    [httpserver.default]
    address=":8200"
    
    [storage.default]
    class-name="inmemory"
    workers=20
    intervals=15
    expire-group=604800
    min-distance=1
    ```
    
    - burrow 서비스 시작
        
        ```
        nohup /root/go/bin/Burrow --config-dir=/usr/local/Burrow/config 1> /dev/null 2>&1 &
        ```
        
    - 참고
        - 
        
        [[Kafka] install Burrow to linux (centos)](https://jundol.me/m/147)
        
        - 
        
        [[Kafka] 카프카 Burrow 설치 (카프카 모니터링)](https://veneas.tistory.com/entry/Kafka-%EC%B9%B4%ED%94%84%EC%B9%B4-Burrow-%EC%84%A4%EC%B9%98-%EC%B9%B4%ED%94%84%EC%B9%B4-%EB%AA%A8%EB%8B%88%ED%84%B0%EB%A7%81)
        

### telegraf 설치 및 실행

- burrow로 수집한 내용을 es로 전달하기 위한 용도
- 설치
    
    ```
    sudo yum install telegraf
    ```
    
- 설정
    - /etc/telegraf/telegraf.conf 파일
    - input으로는 burrow로 하니까, burrow설정
    
    ```
    [[inputs.burrow]]
       ## Burrow API endpoints in format "schema://host:port".
       ## Default is "http://localhost:8000".
       servers = ["http://localhost:8200"]
       topics_exclude = [ "__consumer_offsets" ]
       groups_exclude = ["console-*"]
    ```
    
    - output은 es에 넣을꺼니까
    
    ```
    [[outputs.elasticsearch]]
       urls = [ "http://localhost:9200" ] # required.
       timeout = "5s"
       enable_sniffer = false
       health_check_interval = "10s"
       index_name = "burrow-%Y.%m.%d" # required.
       manage_template = true
       template_name = "telegraf"
       overwrite_template = false
    
    ```
    
    - influxdb는 사용안하므로 주석처리
- 서비스 시작
    
    ```
    systemctl enable --now telegraf
    systemctl status telegraf
    ```
    
- 에러
    - [agent] Failed to connect to [outputs.elasticsear...fined’ 라고 나서 , influxdb 주석하고 나니 사라짐
    - [agent] Failed to connect to [outputs.elasticsearch], retrying in 15s, error was 'elasticsearch template_name configuration not defined’
        - 이건 outputs.elasticsearch설정의 template_name이 주석되어있어서 주석해체하니 에러 해결
- 참조
    - 
    
    [Install Telegraf](https://docs.influxdata.com/telegraf/v1.21/introduction/installation/?t=RedHat+%26amp%3B+CentOS)
    
    - 
    
    [튜토리얼-Ubuntu Linux에서 Telegraf 설치 [단계별]](https://techexpert.tips/ko/influxdb-ko/ubuntu-linux%EC%97%90-telegraf-%EC%84%A4%EC%B9%98/)
    

### kibana

- es로 데이터가 전송되었으면, kibana에서 index를 burrow-*로 생성한다.

### grafana

- database를 elasticsearch 추가하고 index를 burrow꺼로 가져오도록 한다.

### reference

[아파치 카프카 Lag 모니터링 대시보드 만들기](https://blog.voidmainvoid.net/279)