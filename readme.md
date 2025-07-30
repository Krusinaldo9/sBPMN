# sBPMN Platform: Semantic BPMN Execution and Virtualization Architecture

This repository provides a modular Docker-based environment for exploring **semantic extensions of BPMN (sBPMN)**, dynamic process execution via **Camunda**, and **virtual SPARQL integration** using **Ontop** and **Comunica**. It demonstrates how BPMN process logs from Camunda can be made queryable in RDF, combined with ontology-driven reasoning, and federated with a background knowledge graph via SPARQL.

## 🚀 Architecture Overview

The architecture consists of five main services connected through shared volumes and Docker networking:

```
                    +--------------------+
                    |      Camunda       |
                    |   BPM Engine + UI  |
                    +---------+----------+
                              |
                              v
                    +---------+----------+
                    |         H2         |
                    |     SQL Backend    |
                    +---------+----------+
                              |
                              v
                    +---------+----------+
                    |        Ontop       |
                    |  Virtual RDF View  |
                    +---------+----------+
                              |
               +--------------+--------------+
               |                             |
               v                             v
         +-----------+               +---------------+
         |  Fuseki   |               |    Comunica    |
         |  KG store |<------------- |  Federation    |
         +-----------+               +---------------+
```

### Components

| Service   | Description |
|-----------|-------------|
| **[Camunda BPM Platform](https://camunda.com/platform/legacy/bpm/)** | BPMN execution engine with web frontend on port `9094`. Deployed with demo user: `demo` / `demo`. |
| **[H2 Database (v1.4.190)](https://www.h2database.com/html/main.html)** | Lightweight SQL database used to store Camunda history logs. |
| **[Ontop Endpoint](https://ontop-vkg.org/)** | Exposes virtual RDF graphs using the OBDA paradigm. It connects to H2 and uses `mapping.obda`, `ontology.owl`, and `database.properties`. |
| **[Apache Jena Fuseki](https://jena.apache.org/documentation/fuseki2/)** | Hosts an RDF knowledge graph on port `9096`, typically used for background domain knowledge (e.g. BPMN concepts). |
| **[Comunica Federated Query Engine](https://comunica.dev/)** | Exposes a unified SPARQL endpoint that federates Ontop and Fuseki data. Accessible at `http://localhost:9097/sparql`. |

---

## 🐳 Deployment

Ensure you have [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/) installed.

### 1. Clone the Repository

```bash
git clone https://github.com/Krusinaldo9/sBPMN.git
cd sBPMN
```

### 2. Launch the Architecture

```bash
docker-compose up --build
```

- Camunda UI: [http://localhost:9094](http://localhost:9094)  
  Login: `demo` / `demo`
- Ontop endpoint: [http://localhost:9095/sparql](http://localhost:9095/sparql)
- Fuseki endpoint: [http://localhost:9096/camunda/sparql](http://localhost:9096/camunda/sparql)
- Comunica endpoint: [http://localhost:9097/sparql](http://localhost:9097/sparql)

---

## 🧠 Ontop Semantic Layer

The **Ontop** service exposes a *virtual RDF knowledge graph* over the Camunda process logs stored in H2.

### Files of Interest

| File | Description |
|------|-------------|
| `ontop/ontology.owl` | OWL ontology used for reasoning. Currently minimal. You can place the sBPMN ontology or an extension here. |
| `ontop/mapping.obda` | OBDA mappings from H2 tables to OWL terms. |
| `ontop/database.properties` | JDBC connection details for the H2 database. |

> ⚠️ If the OWL file is updated (e.g., new BPMN process concepts), you must restart the Ontop service to reload reasoning.

---

## ⚠️ Known Issues and Best Practices

### 🧠 Comunica Caching Issue

If Comunica is used before the data sources (Ontop or Fuseki) return results, it may cache an **empty response**. This leads to seemingly persistent missing data.

**Best Practice**:
- Make sure **Camunda logs** and **Fuseki data** are present before querying Comunica.
- If an empty result was cached too early:

```bash
docker-compose restart comunica
```

### 🚨 Ontop Initial Metadata Error

The first SPARQL call from Comunica to Ontop triggers a metadata fetch, which may produce a **non-fatal error** in the logs. This can be ignored.

---

## 🧩 Extending the Ontology

You may replace `ontology.owl` with:

- The official [sBPMN ontology](https://github.com/bptlab/sbpmn-ontology)
- Any custom OWL file suitable for your domain

Then:
1. Update `mapping.obda` if needed.
2. Restart the `ontop` service:

```bash
docker-compose restart ontop
```

---

## 📚 References and Frameworks

- [Camunda BPM Platform](https://camunda.com/platform/legacy/bpm/)
- [H2 Database](https://www.h2database.com/)
- [Ontop OBDA Platform](https://ontop-vkg.org/)
- [Apache Jena Fuseki](https://jena.apache.org/documentation/fuseki2/)
- [Comunica SPARQL Engine](https://comunica.dev/)

---

## 💡 Use Case

This architecture is developed as part of the **Teaming.AI** project’s Use Case 2:  
*Human-AI teaming in the injection moulding industry.*

It enables semantic modeling and querying of human-executed business processes through:

- Dynamic logging via Camunda
- Ontological enrichment and reasoning via OWL + OBDA
- Unified access to hybrid data sources via federated SPARQL (Comunica)

---

## 📬 Contact

For academic inquiries, suggestions, or contributions, please open a GitHub issue or submit a pull request.
