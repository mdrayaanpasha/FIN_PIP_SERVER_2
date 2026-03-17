## Anomaly Detection Service (`s_2`)

This service is a **Kafka Consumer**. It listens for triggers, pulls the latest market data from Redis, and runs an algorithm to detect significant price swings (anomalies) in stock tickers.

---

### 🧠 Logic & Flow
The service acts as an intelligent observer. Instead of just passing data through, it compares current market prices against the previous state.



1.  **Listen:** Waits for a message on the `anomaly` Kafka topic.
2.  **Fetch:** When triggered, it pulls the full OHLCV batch from Redis (`latest_ohlcv`).
3.  **Analyze:** It compares the current `close` price of each ticker (e.g., AAPL, MSFT) to the price stored in `ohlcv_baseline`.
4.  **Detect:** If the price has changed by **more than 5%**, it flags it as an anomaly.
5.  **Update:** It saves the new "Baseline" and the list of detected "Anomalies" back to Redis.

---

### 📋 Technical Requirements
* **Node.js 20+**: The runtime environment.
* **Redis**: Used as the primary data source and result store.
* **Kafka Cluster**: Provides the trigger signals for the analysis.
* **SSL Certs**: Decoded from Base64 at startup for secure Kafka communication.

---

### 🔑 Configuration (`.env`)
The service requires the same secure connection details as the producer to access the shared infrastructure:

| Variable | Description |
| :--- | :--- |
| `KAFKA_URL` | Kafka broker endpoint. |
| `REDIS_URL` | Redis connection URL. |
| `SERVICE_CERT` | Client SSL Certificate (Base64). |
| `SERVICE_KEY` | Client Private Key (Base64). |

---

### 🌐 API Endpoints
While the service runs automatically, you can inspect its findings via these HTTP endpoints:

* **GET `/anomalies`**: Returns a list of all tickers currently showing a >5% price swing.
* **GET `/baseline`**: Returns the previous market state used for comparison.

---

### 🐳 Deployment
Optimized for production using a lightweight Docker image.

**Build:**
```bash
docker build -t anomaly-detector .
```

**Run:**
```bash
docker run -p 3001:3001 --env-file .env anomaly-detector
```

