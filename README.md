# ai-driven-load-balancing
A cloud computing load balancer powered by **Q-Learning** — an adaptive reinforcement learning algorithm that learns optimal server routing policies over time.

---

## Architecture

```
Incoming Requests
       │
       ▼
┌─────────────────────────────┐
│    AI Load Balancer         │
│    Q-Learning Agent         │  ← learns which server to prefer
│    ε-greedy policy          │
└──────┬────────┬────────┬────┘
       │        │        │
  Server-A  Server-B  Server-C  Server-D
       │        │        │        │
       └────────┴────────┴────────┘
                      │
            Metrics Collector
            (feeds reward back)
```

## Components

| Class | Responsibility |
|---|---|
| `Server` | Simulates a cloud server with dynamic CPU/memory/latency |
| `ServerPool` | Manages the collection of servers |
| `QLearningAgent` | Q-table based adaptive routing brain |
| `LoadBalancer` | Orchestrates routing using the agent |
| `MetricsCollector` | Aggregates telemetry, computes rewards |
| `Simulation` | Main simulation loop + JSON output |
| `dashboard/index.html` | Live visualization dashboard |

## Q-Learning Details

**State**: Discretized health scores of all servers (5 buckets × 4 servers = 625 states).

**Action**: Which server to route the next request to (0–3).

**Reward**: `health_score × 0.7 − latency_penalty × 0.3` (negative if server is unhealthy).

**Policy**: ε-greedy — starts fully exploratory (ε=1.0), decays to near-greedy (ε=0.05).

**Bellman Update**:
```
Q(s,a) ← Q(s,a) + α × [r + γ × max_a'(Q(s',a')) − Q(s,a)]
```
- α (learning rate) = 0.15
- γ (discount factor) = 0.9
- ε decay = 0.9995 per step

## Getting Started

### Prerequisites
- Java 17+
- Maven 3.8+

### Build & Run

```bash
cd ai-load-balancer
mvn clean package -q
java -jar target/ai-load-balancer-1.0.0.jar
```

### View the Dashboard

After running the simulation, open `dashboard/index.html` in a browser.

The dashboard also runs a **live in-browser simulation** (the JS engine mirrors the Java logic exactly), so you can click **▶ RUN SIM** without needing to run the Java jar first.

### Run Tests

```bash
mvn test
```

## Simulation Details

- **200 ticks** total
- **10 requests/tick** normally, **30 requests/tick** during the spike (ticks 90–110)
- Server-C has lower capacity (10 vs 20) — represents a legacy server
- Random failures simulate cloud chaos (0.1% chance/tick, auto-recovery)
- The agent learns to avoid Server-C and unhealthy servers over time

## Project Structure

```
ai-load-balancer/
├── pom.xml
├── README.md
├── dashboard/
│   ├── index.html                  ← live visualization
│   ├── simulation_results.json     ← generated after Java run
│   └── routing_stats.json          ← generated after Java run
└── src/
    ├── main/java/com/loadbalancer/
    │   ├── Server.java
    │   ├── ServerPool.java
    │   ├── QLearningAgent.java
    │   ├── LoadBalancer.java
    │   ├── MetricsCollector.java
    │   └── Simulation.java
    └── test/java/com/loadbalancer/
        └── LoadBalancerTest.java
```

