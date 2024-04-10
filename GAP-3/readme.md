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
