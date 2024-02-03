---
layout: post
title: setting up grafana, prometheus, kafka exporter on window
date: 2024-02-03 22:56:00
last_modified_at : 2024-02-03 22:56:00
parent: Monitoring
grand_parent: Quality Practice
nav_exclude: true
description: setting up grafana, prometheus, kafka exporter on window
---

# Goal

- window 환경에서 모니터링 툴을 셋팅해서 kafka lag을 추적합니다.

# Install

## 1. grafana 설치

- [그라파나-설치-방법-Window](https://hstory0208.tistory.com/entry/Grafana-%EA%B7%B8%EB%9D%BC%ED%8C%8C%EB%82%98-%EC%84%A4%EC%B9%98-%EB%B0%A9%EB%B2%95-Window){:target="_blank"} 대로 그라파나 설치
- 기본 포트 변경 : 3000 → 9090으로 변경
- bin밑에서 grafana-server.exe 싫행
    
    ```java
    jhsim@jhsim MINGW64 /d/_WAS/grafana-v10.2.2/bin
    $ ./grafana-server.exe
    ```
    
- http://127.0.0.1:9090/ 로 접속 (admin/admin)

## 2. prometheus 설치

- [https://prometheus.io/download/](https://prometheus.io/download/){:target="_blank"} 에서 설치 파일 다운
- 압축풀고
- prometheus.yml 수정
    
    ```xml
    # my global config
    global:
      scrape_interval: 30s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
      evaluation_interval: 30s # Evaluate rules every 15 seconds. The default is every 1 minute.
      scrape_timeout: 30s # is set to the global default (10s).
    
    # Alertmanager configuration
    alerting:
      alertmanagers:
        - static_configs:
            - targets:
              # - alertmanager:9093
    
    # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
    rule_files:
      # - "first_rules.yml"
      # - "second_rules.yml"
    
    # A scrape configuration containing exactly one endpoint to scrape:
    # Here it's Prometheus itself.
    scrape_configs:
      # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
      - job_name: "prometheus"
    
        # metrics_path defaults to '/metrics'
        # scheme defaults to 'http'.
    
        static_configs:
          - targets: ["localhost:9089"]
    
      - job_name: 'eureka'
    
        metrics_path: /actuator/prometheus
    
        # Scrape Eureka itself to discover new services.
        eureka_sd_configs:
          - server: http://127.0.0.1:8888/eureka
    
        relabel_configs:
        # Path to rewrite metrics
          - source_labels: ["__meta_eureka_app_name"]
            action: replace
            target_label: app_name
          - source_labels: ["__meta_eureka_app_instance_ip_addr"]
            action: replace
            target_label: ip_addr
      
      - job_name: 'kafka'
    
          # metrics_path defaults to '/metrics'
          # scheme defaults to 'http'.
    
        static_configs:
          - targets:
             - '192.168.137.18:9308' # kafka_exporter
    ```
    

- 기본 포트 바꿔서 실행
    
    ```java
    jhsim@jhsim MINGW64 /d/_WAS/prometheus-2.48.1
    $ ./prometheus.exe --config.file=prometheus.yml --web.listen-address=:9089
    ```
    

## 3. grafana에서 datasource 셋팅

- Home → Connections → Data sources 선택
- 프로메테우스 정보 입력 후 save

![setting_up_kafka_exporter2.png](../img/setting_up_kafka_exporter2.png)


## 4. kafka-exporter설치

- [Kafka,-Prometheus,-Grafana.html](https://knowjea.github.io/2021/03/01/.Kafka,-Prometheus,-Grafana.html){:target="_blank"} 참고
- https://github.com/danielqsj/kafka_exporter/releases 에서 Download
- 압축풀고
- kafka 서버정보를 설정 후 실행
    
    ```java
    ./kafka_exporter --kafka.server=172.16.100.130:9092
    ```
    
- prometheus.yml에 정보 추가
    
    ```xml
      - job_name: 'kafka'
    
          # metrics_path defaults to '/metrics'
          # scheme defaults to 'http'.
    
        static_configs:
          - targets:
             - '192.168.137.18:9308' # kafka_exporter
    ```
    

# kafka lag 모니터링

- kafka lag을 모니터링하는 방법에는 몇 가지가 있지만, kafka exporter를 사용해보기로 했습니다.
- jmx 대비 좀더 디테일하게 모니터링할 수 있어서 좋은 것 같습니다.
- 토픽명을 필터가 가능하다는 것!
- 컨슈밍되는 카운트도 조회된다는 것!
- lag쌓이는 것 외에
    - Message in per second/minute 초/분당 발행되는 메시지 수
    - Message consume in minute 소비되는 메시지 수

[kafka exporter]

![setting_up_kafka_exporter1.png](../img/setting_up_kafka_exporter1.png)

![setting_up_kafka_exporter4.png](../img/setting_up_kafka_exporter4.png)

- 참고용으로 jmx로 테스트한 lag 결과입니다.

![setting_up_kafka_exporter3.png](../img/setting_up_kafka_exporter3.png)