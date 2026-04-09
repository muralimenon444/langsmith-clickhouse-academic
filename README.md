# 🏥 LangGraph Clinical Data Agent

> **Production-ready clinical data analysis agent powered by LangGraph, Databricks Foundation Models, and Unity Catalog**

[![LangGraph](https://img.shields.io/badge/LangGraph-State%20Machine-blue)](https://github.com/langchain-ai/langgraph)
[![Databricks](https://img.shields.io/badge/Databricks-Production-orange)](https://databricks.com)
[![Python](https://img.shields.io/badge/Python-3.11+-green)](https://python.org)
[![LangSmith](https://img.shields.io/badge/LangSmith-Tracing-purple)](https://smith.langchain.com)

## 📋 Overview

A production-grade agentic system that translates natural language questions into validated SQL queries, executes them against clinical data tables, and generates professional healthcare insights. Built with **deterministic safety guardrails**, **distributed tracing**, and **graceful error handling** for enterprise deployment.

### Key Capabilities

- 🧠 **Natural Language to SQL**: Meta Llama 3.3 70B converts clinical questions to optimized Spark SQL
- 🛡️ **Safety-First Architecture**: Deterministic validation blocks all destructive SQL operations
- ⚡ **Production-Ready Execution**: Databricks SQL Warehouse integration with configurable timeout
- 📊 **Clinical Insights**: LLM-powered summarization with operational recommendations
- 🔍 **Full Observability**: LangSmith distributed tracing + ClickHouse logging (best-effort)
- 🔐 **Secure by Design**: Databricks Secrets for all credentials, no hardcoded keys

---

## 🏗️ Architecture

### 5-Node LangGraph State Machine

```
┌─────────────┐
│ User Query  │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│  Analyst Node   │  Generate SQL from natural language
│  (Meta Llama)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Validator Node  │  Deterministic safety checks
│ (Rule-based)    │
└────────┬────────┘
         │
         ▼
    ┌────────┐
    │ Router │  Safe SQL?
    └───┬────┘
        │
    ┌───┴───┐
    │       │
    ▼       ▼
 ┌─────┐  ┌────┐
 │ END │  │ OK │
 └─────┘  └──┬─┘
             │
             ▼
    ┌─────────────────┐
    │ Executor Node   │  Run SQL on warehouse
    │ (SQL API)       │
    └────────┬────────┘
             │
             ▼
    ┌─────────────────┐
    │ Summarizer Node │  Generate clinical insights
    │ (Meta Llama)    │
    └────────┬────────┘
             │
             ▼
      ┌──────────────┐
      │ Final Answer │
      └──────────────┘
```

### Node Responsibilities

| Node | Purpose | Technology | Output |
|------|---------|------------|--------|
| **Analyst** | Natural language → SQL | Meta Llama 3.3 70B | Spark SQL query |
| **Validator** | Safety checks | Deterministic rules | Boolean (safe/unsafe) |
| **Router** | Conditional branching | LangGraph | Route decision |
| **Executor** | SQL execution | Databricks SQL API | Query results |
| **Summarizer** | Insight generation | Meta Llama 3.3 70B | Clinical summary |

---

## 🚀 Quick Start

### Prerequisites

- **Databricks Workspace** (Serverless or All-Purpose Cluster)
- **Unity Catalog** with clinical data tables
- **LangSmith Account** (for tracing)
- **ClickHouse Cloud** (optional, for additional logging)

### Installation

1. **Clone the repository**
   ```bash
   git clone <your-repo-url>
   cd langchain-production-agent
   ```

2. **Install dependencies** (in notebook Cell 2)
   ```python
   %pip install --upgrade numpy<2.0 langgraph langsmith clickhouse-connect --quiet
   dbutils.library.restartPython()
   ```

3. **Configure Databricks Secrets**
   ```python
   from databricks.sdk import WorkspaceClient
   w = WorkspaceClient()
   
   # LangSmith API Key
   w.secrets.create_scope(scope="langsmith", initial_manage_principal="users")
   w.secrets.put_secret(scope="langsmith", key="api_key", string_value="lsv2_pt_...")
   
   # ClickHouse credentials (optional)
   w.secrets.create_scope(scope="clickhouse", initial_manage_principal="users")
   w.secrets.put_secret(scope="clickhouse", key="host", string_value="xyz.clickhouse.cloud")
   w.secrets.put_secret(scope="clickhouse", key="password", string_value="...")
   w.secrets.put_secret(scope="clickhouse", key="port", string_value="8443")
   w.secrets.put_secret(scope="clickhouse", key="user", string_value="default")
   ```

4. **Run the main notebook**  
   Open `agent_logic/01_langgraph_state_machine.ipynb` and run cells 3-12

---

## 💻 Usage

### Basic Query Execution

```python
# Initialize agent (cells 3-9)
app = workflow.compile()

# Execute a clinical query
result = app.invoke({
    "user_query": "How many unique patients had inpatient encounters in 2023?"
})

print(result["final_answer"])  # Clinical insights
```

### Example Queries

```python
# Patient demographics
"What is the average age of patients with diabetes?"

# Encounter analysis
"Compare inpatient admissions between Q3 and Q4 2024"

# Trend analysis
"Show monthly emergency room visits for the past 6 months"

# Complex joins
"What percentage of patients over 65 had wellness encounters?"
```

### Safety Validation in Action

```python
# ✅ SAFE - SELECT only
app.invoke({"user_query": "Show me all encounters from 2024"})
# Result: Query executes successfully

# ❌ UNSAFE - Blocked by validator
app.invoke({"user_query": "Delete all records from patient_registry"})
# Result: Workflow stops at validator, no execution
```

---

## 📁 Project Structure

```
langchain-production-agent/
├── agent_logic/
│   └── 01_langgraph_state_machine.ipynb   # Main agent workflow
├── observability/
│   └── 01_clickhouse_trace_logger.ipynb   # ClickHouse logging
├── utils/
│   └── 01_Clickhouse_Connection_Test.ipynb # Connection utilities
├── Add_Secret.ipynb                        # Secret management
└── README.md                               # This file
```

### Key Notebooks

#### `agent_logic/01_langgraph_state_machine.ipynb`
Core agent implementation with all 5 nodes:
- **Cell 3**: LangSmith configuration
- **Cell 4**: AgentState definition
- **Cell 5**: Analyst node (SQL generation)
- **Cell 6**: Validator node (safety checks)
- **Cell 7**: Router function
- **Cell 9**: Executor & Summarizer nodes
- **Cell 12**: Production test with dual observability

---

## 🔍 Observability

### Primary: LangSmith Distributed Tracing

**Status**: ✅ Fully Operational

LangSmith provides complete visibility into agent execution:

- **Node-level tracing**: See timing and outputs for each node
- **LLM call monitoring**: Track prompts, responses, and token usage
- **State transitions**: Visualize workflow routing decisions
- **Error debugging**: Inspect failures with full stack traces

**Access your traces**:  
👉 [https://smith.langchain.com](https://smith.langchain.com)

### Secondary: ClickHouse Logging (Best-Effort)

**Status**: ⚠️ Graceful Degradation on Serverless

ClickHouse logging captures:
- User queries
- Generated SQL
- Safety validation results
- Execution latency (ms)
- Final answers

**Known Limitation**:  
NumPy binary incompatibility on Databricks serverless clusters causes ClickHouse logging to fail gracefully. The agent continues working perfectly—LangSmith provides full observability.

---

## 🔐 Security

### Credential Management

**All secrets stored in Databricks Secrets**:
- ✅ LangSmith API key: `langsmith.api_key`
- ✅ ClickHouse host: `clickhouse.host`
- ✅ ClickHouse password: `clickhouse.password`
- ✅ ClickHouse port: `clickhouse.port`
- ✅ ClickHouse user: `clickhouse.user`

**Zero hardcoded credentials** in notebooks or code.

### SQL Safety Guardrails

**Deterministic validation blocks**:
- `DROP` - Prevents table deletion
- `DELETE` - Prevents data deletion
- `UPDATE` - Prevents data modification
- `INSERT` - Prevents data insertion
- `ALTER` - Prevents schema changes
- `TRUNCATE` - Prevents data removal
- `MERGE` - Prevents upsert operations

**Only `SELECT` queries are allowed**.

---

## 🎯 Production Deployment

### Readiness Checklist

- ✅ No hardcoded credentials
- ✅ Full LangSmith observability
- ✅ Deterministic safety validation
- ✅ Error handling in all nodes
- ✅ SQL execution via warehouse API
- ✅ Clinical insight generation
- ✅ Graceful degradation (ClickHouse)
- ✅ Safe for version control

### Performance Metrics (Typical)

| Metric | Value |
|--------|-------|
| **End-to-end latency** | 4-30 seconds |
| **SQL generation** | 1-2 seconds |
| **Validation** | < 100ms (deterministic) |
| **Execution** | 2-25 seconds (depends on query) |
| **Summarization** | 1-3 seconds |

---

## 🐛 Troubleshooting

### Common Issues

#### 1. "numpy.dtype size changed" error

**Symptom**: ClickHouse logging fails with binary incompatibility error  
**Impact**: Non-blocking, agent continues working  
**Solution**: Accept graceful degradation (LangSmith provides full observability)

```python
# This is expected behavior on serverless
⚠️  ClickHouse logging skipped (known serverless limitation)
```

#### 2. SQL execution timeout

**Symptom**: Query stays in `PENDING` state  
**Solution**: Increase timeout in executor node (Cell 9)

```python
statement = w.statement_execution.execute_statement(
    warehouse_id="your-warehouse-id",
    statement=sql,
    wait_timeout="60s"  # Increase from 30s
)
```

#### 3. "Secret not found" error

**Symptom**: `dbutils.secrets.get()` fails  
**Solution**: Verify secret scope and key exist

```python
# List all scopes
w.secrets.list_scopes()

# List keys in scope
w.secrets.list_secrets(scope="langsmith")
```

---

## 📊 Data Schema

### Expected Tables (Unity Catalog)

#### `clinical_dev.analytics_agent.clinical_encounters`
| Column | Type | Description |
|--------|------|-------------|
| `PATIENT` | STRING | Patient identifier |
| `ENCOUNTERCLASS` | STRING | Type of encounter (Inpatient, Outpatient, etc.) |
| `START` | TIMESTAMP | Encounter start time |
| `STOP` | TIMESTAMP | Encounter end time |

#### `clinical_dev.analytics_agent.patient_registry`
| Column | Type | Description |
|--------|------|-------------|
| `PATIENT` | STRING | Patient identifier |
| `BIRTHDATE` | DATE | Date of birth |
| `GENDER` | STRING | Patient gender |
| `RACE` | STRING | Patient race |
| `ETHNICITY` | STRING | Patient ethnicity |

---

## 📊 Data Engineering & Compliance

- **Source:** 100% Synthetic Clinical Dataset
- **Generation:** Custom Python pipeline using NumPy/Pandas to simulate realistic patient behavior
- **Scale:** 5,000 encounters across 1,000 unique patients
- **Compliance:** Built using a "Privacy-by-Design" approach to ensure zero Protected Health Information (PHI) was used during development

---

## 📚 Resources

- [LangGraph Documentation](https://langchain-ai.github.io/langgraph/)
- [Databricks Foundation Models](https://docs.databricks.com/machine-learning/foundation-models/index.html)
- [LangSmith Documentation](https://docs.smith.langchain.com/)
- [Unity Catalog](https://docs.databricks.com/data-governance/unity-catalog/index.html)

---

## 👤 Author

**Murali Menon**  
muralimenon444@gmail.com

**Built with**: Databricks, LangGraph, LangChain, Meta Llama 3.3 70B, ClickHouse

---

## 🙏 Acknowledgments

- **LangChain**: State machine framework (LangGraph)
- **Databricks**: Foundation models and Unity Catalog
- **Meta**: Llama 3.3 70B Instruct
- **ClickHouse**: Time-series logging infrastructure
