# Unified Development Plan: Smart Irrigation Telemetry MVP

This document describes a phased development roadmap for the Smart Irrigation telemetry system, integrating backend, infrastructure, and (future) publisher client work. It’s intended to keep the work ordered, explicit, and easy to track.

## Legend
- `[x]` Task Completed
- `[ ]` Task Pending

---

## **PHASE 0: Ground Rules & Fixed Conventions**

This phase defines naming, schema, ports, and env vars to avoid churn later.

*   **Naming & Contracts**
    *   [ ] Task 0.1: Finalize MQTT topic names (e.g. `sensors/telemetry`).
    *   [ ] Task 0.2: Freeze message JSON schema:
        ```json
        { "device_id": "string", "moisture": 0.0, "temperature": 0.0, "humidity": 0.0, "ts": "ISO8601" }
        ```
    *   [ ] Task 0.3: Finalize environment variable keys (APP_PORT, MQTT_BROKER_HOST, MQTT_TOPIC, etc.).
*   **Persistence Decision**
    *   [ ] Task 0.4: Start with in-memory ring buffer.
    *   [ ] Task 0.5: Plan for SQLite or TimescaleDB later (PHASE 7).
*   **Ports & K8s Service Names**
    *   [ ] Task 0.6: Assign ports: FastAPI: `8000`, MQTT: `1883`.
    *   [ ] Task 0.7: Reserve service names for K8s: `fastapi`, `mosquitto`.

---

## **PHASE 1: Local Dev Loop**

Minimal FastAPI server and Mosquitto broker for end-to-end “hello world”.

*   **Backend Setup**
    *   [ ] Task 1.1: Scaffold FastAPI app with `/health` and `/telemetry/latest`.
    *   [ ] Task 1.2: Add background MQTT subscriber thread (paho-mqtt).
    *   [ ] Task 1.3: Store last received message in memory.
*   **Local Broker**
    *   [ ] Task 1.4: Run Mosquitto locally (container or host).
    *   [ ] Task 1.5: Connect backend subscriber to broker.
*   **Validation**
    *   [ ] Task 1.6: Publish test message with `mosquitto_pub`.
    *   [ ] Task 1.7: Verify `/health` works and `/telemetry/latest` shows last message.

---

## **PHASE 2: Message Contract & Ring Buffer**

Add validation and proper storage of recent telemetry.

*   **Validation**
    *   [ ] Task 2.1: Implement Pydantic model for Telemetry schema.
    *   [ ] Task 2.2: Drop invalid messages, log reason.
*   **Ring Buffer**
    *   [ ] Task 2.3: Add deque-based buffer with size limit (1000).
    *   [ ] Task 2.4: Update `/telemetry/latest` to return newest-first up to `limit` query param.

---

## **PHASE 3: Dockerize with Podman**

Create container images for backend and Java publisher.

*   **Backend Image**
    *   [ ] Task 3.1: Write `Dockerfile.podman` (Python slim, uvicorn).
    *   [ ] Task 3.2: Build and run with env vars for broker connection.
*   **Java Publisher Image**
    *   [ ] Task 3.3: Multi-stage Maven build for small runtime image.
    *   [ ] Task 3.4: Build and run publisher container locally.

---

## **PHASE 4: Java Sensor Publisher**

Simulated sensor client to send telemetry.

*   **Functionality**
    *   [ ] Task 4.1: Publish 1 msg/sec with realistic value ranges.
    *   [ ] Task 4.2: Read broker URL and topic from env vars.
    *   [ ] Task 4.3: Auto-reconnect on disconnect.
*   **Validation**
    *   [ ] Task 4.4: Confirm backend buffer count increases steadily.

---

## **PHASE 5: Kubernetes Basics (Minikube + Podman)**

Deploy backend and publisher in-cluster.

*   **Manifests**
    *   [ ] Task 5.1: Write Deployment + Service for FastAPI.
    *   [ ] Task 5.2: Write Deployment for publisher.
    *   [ ] Task 5.3: Optional: Mosquitto Deployment + Service.
*   **Configuration**
    *   [ ] Task 5.4: ConfigMap for MQTT_BROKER_HOST, MQTT_TOPIC.
*   **Validation**
    *   [ ] Task 5.5: `kubectl port-forward` FastAPI; verify telemetry flows from in-cluster publisher.

---

## **PHASE 6: Probes, Resources, Logs**

Make pods production-ish.

*   **Health Probes**
    *   [ ] Task 6.1: Add readiness and liveness probes to FastAPI Deployment.
*   **Resources**
    *   [ ] Task 6.2: Set CPU/memory requests and limits.
*   **Logging**
    *   [ ] Task 6.3: Switch to one-line JSON logs for consistency.

---

## **PHASE 7: Persistence Layer (Optional but Recommended)**

Swap buffer for DB storage; keep cache for `/latest`.

*   **Database**
    *   [ ] Task 7.1: Integrate SQLite or Postgres (SQLAlchemy).
    *   [ ] Task 7.2: Store incoming telemetry in DB.
    *   [ ] Task 7.3: Implement `/telemetry/query?from=&to=&device_id=` endpoint.
*   **Archival**
    *   [ ] Task 7.4: Optionally add K8s Job/CronJob to archive/rotate old data.

---

## **PHASE 8: Security & Config Management**

Lock it down without tripping over yourself.

*   **Auth**
    *   [ ] Task 8.1: If broker needs auth, add Secret for username/password.
*   **K8s Hygiene**
    *   [ ] Task 8.2: Namespace scoping and minimal RBAC for ServiceAccounts.
    *   [ ] Task 8.3: Disable anonymous access on Mosquitto in-cluster.

---

## **PHASE 9: Observability Lite**

Basic metrics without diving into full Grafana stack (yet).

*   **Metrics Endpoint**
    *   [ ] Task 9.1: Add `/metrics` using `prometheus_client`.
    *   [ ] Task 9.2: Expose counters for valid/invalid messages, subscriber lag.

---

## **PHASE 10: CI/CD & Polish**

Automate build and deploy.

*   **Build**
    *   [ ] Task 10.1: Lint, test, build images on push.
    *   [ ] Task 10.2: Tag images with git SHA; push to registry.
*   **Deploy**
    *   [ ] Task 10.3: Use Kustomize overlays for dev vs prod (broker host, replicas).
    *   [ ] Task 10.4: Smoke test job after deploy to ensure telemetry flows.

---

## **Known Gotchas**

- **Cluster DNS:** Use K8s service name for `MQTT_BROKER_HOST` (e.g. `mosquitto`), never `localhost`.
- **Startup race:** Backend must tolerate broker not being ready immediately.
- **Minikube + Podman:** Ensure same runtime context for build and deploy.
- **MQTT QoS:** Default to `QoS=1`; don’t retain unless intended.
