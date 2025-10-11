

# 1.  **Grafana + Prometheus**

Great! Here's a step-by-step guide to set up **Grafana + Prometheus** to monitor your **Spring Boot application** using the `/actuator/prometheus` endpoint.

---

## 🧰 Step 1: Add Dependencies to Your Spring Boot App

In `build.gradle`:

```groovy
implementation 'org.springframework.boot:spring-boot-starter-actuator'
implementation 'io.micrometer:micrometer-registry-prometheus'
```

In `application.yml`:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

This exposes metrics at:

```
http://localhost:8080/actuator/prometheus
```

---

## 📦 Step 2: Install Prometheus

1. Download Prometheus from https://prometheus.io/download
2. Create a `prometheus.yml` config file:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080']  # or 'localhost:8080' if not using Docker
```

3. Run Prometheus:

```bash
./prometheus --config.file=prometheus.yml
```

Access Prometheus UI at:

```
http://localhost:9090
```

---

## 📊 Step 3: Install Grafana

1. Download from https://grafana.com/grafana/download
2. Start Grafana and go to:

```
http://localhost:3000
```

(Default login: `admin` / `admin`)

3. Add Prometheus as a data source:
    - Go to **Settings → Data Sources → Add data source**
    - Choose **Prometheus**
    - Set URL to `http://localhost:9090`

---

## 📈 Step 4: Create Dashboards

- Use built-in dashboards or import community dashboards from Grafana Dashboards
- Example search: “Spring Boot Micrometer”

---

Would you like a Docker Compose setup to run all of this with one command?