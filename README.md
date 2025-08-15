# 🌾 Smart Farming IoT System — Simulated Setup (Podman + Minikube)

This project simulates a Smart Farming IoT system using MQTT and dummy sensor values.  
It follows a real-world architecture where sensor data is published via MQTT and processed by a FastAPI backend.  
All components are containerized using **Podman** and orchestrated with **Minikube**.

---

## 🧱 Architecture Overview

```text
+-------------------+         +------------------+         +----------------------+
| Java Dummy Sensor | ─────▶  |  MQTT Broker     | ─────▶  |  FastAPI Backend     |
| (Podman container)|         | (HiveMQ/Mosquitto)         | (Podman container)   |
+-------------------+         +------------------+         +----------------------+
                                                              |
                                                              ▼
                                                 +---------------------------+
                                                 |     Android App (Future)  |
                                                 |   Fetches stats via API   |
                                                 +---------------------------+
smart-farming/
├── backend-fastapi/               # FastAPI backend with MQTT subscriber
│   ├── main.py
│   ├── requirements.txt
│   ├── Dockerfile.podman
│
├── java-sensor-publisher/        # Java MQTT sensor simulator
│   ├── DummySensorPublisher.java
│   ├── pom.xml
│   ├── Dockerfile.podman
│
├── k8s/                          # Kubernetes manifests
│   ├── fastapi-deployment.yaml
│   ├── fastapi-service.yaml
│   ├── sensor-publisher.yaml
│   ├── mqtt-broker.yaml          # Optional: deploy Mosquitto inside cluster
│
├── README.md
└── .gitignore
