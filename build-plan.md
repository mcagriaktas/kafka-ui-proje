# Kafka Manager (UI) - Project Execution Plan

## Architecture Overview
* **Backend:** Python with FastAPI (high performance, async support).
* **Kafka Library:** `confluent-kafka` (fastest, most reliable AdminClient).
* **Config Management:** YAML parsing for multi-cluster support.
* **Frontend:** Simple HTML/JS with Bootstrap/Tailwind served directly by FastAPI (to keep it a single deployable unit initially).

---

## 🏗️ Phase 1: Initialization & Configuration
1. **Initialize Git & Environment:**
   - Initialize the local git repository if not already done.
   - Create a `requirements.txt` containing: `fastapi`, `uvicorn`, `confluent-kafka`, `pyyaml`, `jinja2`.
   - Create a `.gitignore` (ignore venv, __pycache__, etc.).
2. **Create `configs.yml`:**
   - Build a YAML file at the root to hold multiple cluster configurations. It must support cluster names, bootstrap servers, and optional security protocols (SASL/Kerberos placeholders).
3. **Build the Config Parser:**
   - Write a `config_loader.py` that reads `configs.yml` and makes the cluster data available to the app.

## 🔌 Phase 2: Kafka Admin Client Wrapper
1. **Create `kafka_service.py`:**
   - Implement a singleton or connection pool for the `confluent_kafka.admin.AdminClient`.
   - **Method:** `get_clusters()` - Return the list of clusters from the config.
   - **Method:** `get_brokers(cluster_name)` - Fetch live broker metadata (ID, Host, Port) using `list_topics().brokers`.
   - **Method:** `get_topics(cluster_name)` - Fetch all topics, filtering out internal ones (like `__consumer_offsets`) unless specifically requested.

## 📊 Phase 3: Deep Topic & Replica Inspection
1. **Expand `kafka_service.py`:**
   - **Method:** `get_topic_details(cluster_name, topic_name)`
   - Extract partition-level data:
     - Partition ID
     - Leader Broker ID
     - Replicas (List of broker IDs)
     - In-Sync Replicas (ISRs)
   - Calculate an "Under-Replicated" boolean flag if `len(ISRs) < len(Replicas)`.

## 🌐 Phase 4: REST API Implementation
1. **Create `main.py` (FastAPI app):**
   - `GET /api/clusters` -> Returns configured clusters.
   - `GET /api/clusters/{cluster_name}/brokers` -> Returns broker list.
   - `GET /api/clusters/{cluster_name}/topics` -> Returns topic list with partition counts.
   - `GET /api/clusters/{cluster_name}/topics/{topic_name}` -> Returns deep replica/ISR data.

## 🖥️ Phase 5: Minimal Frontend (UI)
1. **Create `templates/index.html`:**
   - Use Jinja2 templates via FastAPI.
   - Build a simple sidebar to select a cluster.
   - Build a dashboard showing: Active Brokers, Total Topics.
   - Build a Topic Table showing: Name, Partitions, Replicas, and a warning badge for Under-Replicated partitions.
