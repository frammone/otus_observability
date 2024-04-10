# GAP-3

/etc/alertmanager/alertmanager.yml

```yaml
templates:
  - '/etc/alertmanager/templates/telegram.tmpl'
route:
  group_by: ['alertname', 'severity']
  group_wait: 30s
  group_interval: 1m
  repeat_interval: 30m
  receiver: 'default'
  routes:
    - receiver: 'critical'
      match:
        severity: 'critical'
    - receiver: 'warning'
      match:
        severity: 'warning'
    - receiver: 'default'
      match:
        severity: 'info'
receivers:
  - name: 'default'
    telegram_configs:
      - api_url: https://api.telegram.org
        bot_token: ''
        chat_id: -4127765568
        parse_mode: 'HTML'
  - name: 'critical'
    telegram_configs:
      - api_url: https://api.telegram.org
        bot_token: ''
        chat_id: -4127765568
        parse_mode: 'HTML'
        message: '{{ template "telegram.default.message.text" . }}'
  - name: 'warning'
    telegram_configs:
      - api_url: https://api.telegram.org
        bot_token: ''
        chat_id: -4120680993
        parse_mode: 'HTML'
        message: '{{ template "telegram.default.message.text" . }}'
```

/etc/prometheus/alert.rules.yml

```yaml
groups:
- name: alert.rules
  rules:
  - alert: Instance Down!
    expr: up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      description: '{{ $labels.instance }} of job {{ $labels.job }} has been down
        for more than 1 minute.'
      summary: Instance {{ $labels.instance }} down
  - alert: Сервис nginx не работает!
    expr: nginx_up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      description: '{{ $labels.instance }} of job {{ $labels.job }} has been down
        for more than 1 minute.'
      summary: Instance {{ $labels.instance }} down
  - alert: Сервис MariaDB не работает!
    expr: mysql_up == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      description: '{{ $labels.instance }} of job {{ $labels.job }} has been down
        for more than 1 minute.'
      summary: Instance {{ $labels.instance }} down
  - alert: Nginx Exporter не работает!
    expr: up{job="nginx"} == 0
    for: 1m
    labels:
      severity: warning
    annotations:
      description: '{{ $labels.instance }} of job {{ $labels.job }} has been down
        for more than 1 minute.'
      summary: Instance {{ $labels.instance }} down
  - alert: MariaDB Exporter не работает!
    expr: up{job="mariadb"} == 0
    for: 1m
    labels:
      severity: warning
    annotations:
      description: '{{ $labels.instance }} of job {{ $labels.job }} has been down
        for more than 1 minute.'
      summary: Instance {{ $labels.instance }} down
```

/etc/prometheus/prometheus.yml

```yaml
...
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 127.0.0.1:9093
rule_files:
  - "alert.rules.yml"
...
```
