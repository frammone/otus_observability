# GAP-2

1) В качестве хранилища метрик решил использовать VictoriaMetrics.

```bash
docker run -d --name victoria-metrics -it --rm -v /data/victoria-metrics-data:/victoria-metrics-data -p 172.17.0.1:8428:8428 victoriametrics/victoria-metrics
```

2) Конфигурация Prometheus

```yaml
root@test-observability:~# cat /etc/prometheus/prometheus.yml
global:
 scrape_interval: 10s
 external_labels:
   server_name: prometheus01
remote_write:
  - url: http://172.17.0.1:8428/api/v1/write
    queue_config:
      max_samples_per_send: 10000
      capacity: 20000
      max_shards: 30
    write_relabel_configs:
      - source_labels: []
        target_label: 'site'
        replacement: 'prod'
        action: replace
scrape_configs:
  - job_name: 'prometheus_master'
    scrape_interval: 5s
    static_configs:
    - targets: ['localhost:9090']
  - job_name: 'nginx'
    scrape_interval: 5s
    static_configs:
    - targets: ['172.17.0.1:9113']
  - job_name: 'php-fpm'
    scrape_interval: 5s
    static_configs:
    - targets: ['172.17.0.1:9253']
  - job_name: 'mariadb'
    scrape_interval: 5s
    static_configs:
    - targets: ['172.17.0.1:9104']
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - http://65.108.93.46:9090
        - http://172.17.0.1:8080
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 172.17.0.1:9115
  - job_name: 'blackbox_exporter'
    static_configs:
      - targets: ['172.17.0.1:9115']
  - job_name: 'victoria_metrics_instance'
    static_configs:
      - targets: ['172.17.0.1:8428']
```

3) Настройка времени хранилища в tsdb:
```
root@test-observability:~# cat /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--storage.tsdb.retention.time=2w \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries
[Install]
WantedBy=multi-user.target
```
