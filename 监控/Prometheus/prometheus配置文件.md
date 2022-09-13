## 配置文件

```yaml
# my global config
global:
  scrape_interval: 15s # 设置抓取间隔为15s, 默认是1分钟
  evaluation_interval: 15s # 监控数据规则评估频率, 指的是prometheus会多长时间进行一次监控规则的评估. 比如设置内存使用量 > 70%时进行报警, 那么prometheus会默认15s来执行一次这个规则, 检查内存的情况
  # scrape_timeout is set to the global default (10s).

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

# prometheus本身的配置
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    # 被监控节点
    static_configs:
      - targets: ["192.168.32.3:9100"]
```

## 启动参数

```bash
--web.read-timeout=5m # 请求链接的最大等待时间
--web.max-connections=512 # 最大的连接数
--storage.tsdb.retention.time=15d # 监控数据保留的时间
--storage.tsdb.path="data/" # 存储数据的目录

--query.timeout=2m # 用户执行一次prometheus查询的时候的超时时间
--query.max-concurrency=20 # 允许同时进行的查询
```

