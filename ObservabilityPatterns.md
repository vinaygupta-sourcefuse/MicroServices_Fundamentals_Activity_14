# **📊 Observability Patterns in Microservices (LoopBack 4)**  
**Observability** in a microservices architecture ensures **visibility into system behavior**, **detects failures**, and **troubleshoots performance issues**. It consists of **Logging, Monitoring, and Tracing**.

---

## **1️⃣ Logging 📜**
Logging captures system **events and errors**. In a microservices system, logs need to be **structured, centralized, and correlated**.

### **🔹 Best Practices for Logging in LB4**
✅ Use **structured JSON logs** for easy processing  
✅ Include **correlation IDs** for tracking requests  
✅ Store logs in a **centralized system** (e.g., Elasticsearch, Logstash, Kibana – ELK stack)  
✅ Use **log levels** (`info`, `warn`, `error`, `debug`)  

---

### **🔹 Implementing Logging in LoopBack 4**
1️⃣ **Install Winston for logging**  
```sh
npm install winston
```

2️⃣ **Create a Logger Provider (`src/providers/logger.provider.ts`)**
```ts
import {Provider} from '@loopback/core';
import winston from 'winston';

export class LoggerProvider implements Provider<winston.Logger> {
  value(): winston.Logger {
    return winston.createLogger({
      level: 'info',
      format: winston.format.json(),
      transports: [
        new winston.transports.Console(),
        new winston.transports.File({ filename: 'logs/app.log' }),
      ],
    });
  }
}
```

3️⃣ **Use the Logger in Controllers**
```ts
import {inject} from '@loopback/core';
import {Logger} from 'winston';

export class OrderController {
  constructor(@inject('services.Logger') private logger: Logger) {}

  async createOrder(order: any) {
    this.logger.info(`Order Created: ${JSON.stringify(order)}`);
    return order;
  }
}
```
✅ **Logs events and errors in JSON format**  
✅ **Can be stored in ELK, CloudWatch, or Loki**  

---

## **2️⃣ Monitoring 📊**
Monitoring tracks **system health, performance, and uptime**. It uses **metrics and alerts** to notify failures.

### **🔹 Key Metrics to Monitor**
📌 **API response time**  
📌 **Database query performance**  
📌 **CPU & Memory usage**  
📌 **Error rates**  

---

### **🔹 Implementing Monitoring in LB4**
1️⃣ **Install Prometheus Client**
```sh
npm install prom-client
```

2️⃣ **Expose Metrics API (`src/controllers/metrics.controller.ts`)**
```ts
import {get} from '@loopback/rest';
import client from 'prom-client';

export class MetricsController {
  private requestCounter = new client.Counter({
    name: 'http_requests_total',
    help: 'Total HTTP Requests',
  });

  @get('/metrics')
  async getMetrics() {
    this.requestCounter.inc();
    return client.register.metrics();
  }
}
```
3️⃣ **Configure Prometheus to Scrape Metrics**
```yaml
scrape_configs:
  - job_name: 'loopback-app'
    static_configs:
      - targets: ['localhost:3000']
```
✅ **Metrics are collected by Prometheus**  
✅ **Alerts can be set up in Grafana**  

---

## **3️⃣ Tracing 🕵️‍♂️**
Tracing provides **end-to-end visibility** into requests **across microservices**.

### **🔹 Why Tracing?**
📌 Detect slow services  
📌 Identify request bottlenecks  
📌 Debug errors in distributed systems  

---

### **🔹 Implementing Tracing in LB4 with OpenTelemetry**
1️⃣ **Install OpenTelemetry Dependencies**
```sh
npm install @opentelemetry/api @opentelemetry/sdk-trace-node @opentelemetry/auto-instrumentations-node
```

2️⃣ **Configure Tracing (`src/tracing.ts`)**
```ts
import {NodeTracerProvider} from '@opentelemetry/sdk-trace-node';
import {SimpleSpanProcessor} from '@opentelemetry/sdk-trace-base';
import {ConsoleSpanExporter} from '@opentelemetry/sdk-trace-base';

const provider = new NodeTracerProvider();
provider.addSpanProcessor(new SimpleSpanProcessor(new ConsoleSpanExporter()));
provider.register();
console.log('Tracing initialized');
```
3️⃣ **Use Tracing in Services**
```ts
import {trace} from '@opentelemetry/api';

const tracer = trace.getTracer('order-service');

export async function placeOrder() {
  const span = tracer.startSpan('placeOrder');
  try {
    console.log('Processing order...');
  } finally {
    span.end();
  }
}
```
✅ **Traces requests across services**  
✅ **Visualize traces in Jaeger or Zipkin**  

---

# **📌 Final Architecture**
| Component  | Tool Used  |
|------------|-----------|
| **Logging**  | Winston + ELK Stack |
| **Monitoring** | Prometheus + Grafana |
| **Tracing** | OpenTelemetry + Jaeger |

