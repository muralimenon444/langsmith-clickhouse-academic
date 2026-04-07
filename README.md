# ClickHouse + LangChain + LangSmith Integration

An academic project demonstrating real-time AI observability by integrating **LangChain** with **ClickHouse** for high-performance logging and **LangSmith** for distributed tracing.

## 📋 Overview

This notebook showcases a production-ready pattern for:
- **AI Application Logging**: Store LangChain interactions in ClickHouse for fast analytics
- **Distributed Tracing**: Automatic trace capture with LangSmith
- **Performance Analytics**: Query and analyze AI response metrics
- **Academic Research**: Foundation for studying LLM application behavior at scale

## 🎯 Features

- ✅ **ClickHouse Integration**: High-performance columnar database for AI logs
- ✅ **LangSmith Tracing**: Automatic observability for LangChain applications
- ✅ **Modern LangChain**: Uses LCEL (LangChain Expression Language) syntax
- ✅ **Analytics Built-in**: Response size tracking, aggregations, and statistics
- ✅ **Production-Ready**: Environment variable management, error handling

## 🚀 Quick Start

### Prerequisites

1. **ClickHouse Cloud Account** (or self-hosted ClickHouse instance)
   - Sign up at [clickhouse.cloud](https://clickhouse.cloud)
   - Note your host URL and password

2. **LangSmith Account**
   - Sign up at [smith.langchain.com](https://smith.langchain.com)
   - Get your API key from Settings

3. **Databricks Workspace** (or local Python environment)
   - Python 3.12+
   - Access to install packages

### Installation

1. **Clone or import the notebook** to your Databricks workspace

2. **Create a `.env` file** in your workspace with the following variables:

```env
CH_HOST=your-clickhouse-host.clickhouse.cloud
CH_PASSWORD=your-clickhouse-password
LANGCHAIN_API_KEY=your-langsmith-api-key
```

3. **Run the notebook cells in order** (Cells 1-7)

## 📦 Dependencies

All dependencies are installed automatically in Cell 1:

```bash
pip install clickhouse-connect langchain langchain-community langchain-openai python-dotenv
```

## 🏗️ Architecture

```
┌─────────────────┐
│  LangChain App  │
│   (Cell 6)      │
└────────┬────────┘
         │
         ├─────────────────┐
         │                 │
         ▼                 ▼
┌─────────────────┐ ┌──────────────┐
│   ClickHouse    │ │  LangSmith   │
│  (Data Store)   │ │   (Tracing)  │
└─────────────────┘ └──────────────┘
         │
         ▼
┌─────────────────┐
│   Analytics     │
│    (Cell 7)     │
└─────────────────┘
```

## 📓 Notebook Structure

| Cell | Title | Purpose |
|------|-------|----------|
| 1 | Install Dependencies | Install required Python packages |
| 2 | Restart Python | Apply installed packages |
| 3 | Connect to ClickHouse | Establish database connection |
| 4 | Configure LangSmith Tracing | Enable observability |
| 5 | Create ClickHouse Table | Set up `ai_logs` schema |
| 6 | Run LangChain Chain | Execute AI chain & log to ClickHouse |
| 7 | Query and Display AI Logs | View results with analytics |

## 💾 Database Schema

The `ai_logs` table structure:

```sql
CREATE TABLE ai_logs (
    event_time DateTime DEFAULT now(),
    prompt String,
    response String,
    tokens_used UInt32
) ENGINE = MergeTree()
ORDER BY event_time
```

## 🔍 Usage Examples

### Basic Chain Execution (Cell 6)

```python
from langchain_community.llms.fake import FakeListLLM
from langchain_core.prompts import PromptTemplate

# Create chain
prompt = PromptTemplate(
    input_variables=["topic"],
    template="Tell me something about {topic}."
)
llm = FakeListLLM(responses=["Response 1", "Response 2"])
chain = prompt | llm

# Execute and log
result = chain.invoke({"topic": "Your Topic"})
```

### Query Analytics (Cell 7)

```python
query = """
SELECT 
    event_time,
    prompt, 
    length(response) AS response_size
FROM ai_logs 
ORDER BY event_time DESC
"""
results = client.query(query)
```

## 🛠️ Customization

### Switch to Real LLM

Replace the `FakeListLLM` in Cell 6 with a real model:

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4",
    temperature=0.7,
    openai_api_key=os.getenv('OPENAI_API_KEY')
)
```

### Add More Fields to Logs

Modify the table schema in Cell 5:

```sql
ALTER TABLE ai_logs ADD COLUMN model_name String;
ALTER TABLE ai_logs ADD COLUMN latency_ms UInt32;
```

### Change LangSmith Project

Update Cell 4:

```python
os.environ["LANGCHAIN_PROJECT"] = "Your-Project-Name"
```

## 📊 Analytics Queries

### Average Response Length by Hour

```sql
SELECT 
    toStartOfHour(event_time) AS hour,
    AVG(length(response)) AS avg_length
FROM ai_logs
GROUP BY hour
ORDER BY hour DESC
```

### Most Common Prompts

```sql
SELECT 
    prompt,
    COUNT(*) AS count
FROM ai_logs
GROUP BY prompt
ORDER BY count DESC
LIMIT 10
```

### Token Usage Statistics

```sql
SELECT 
    SUM(tokens_used) AS total_tokens,
    AVG(tokens_used) AS avg_tokens,
    MAX(tokens_used) AS max_tokens
FROM ai_logs
```

## 🔒 Security Notes

- **Never commit `.env` files** to version control
- Use Databricks Secrets for production deployments
- Restrict ClickHouse user permissions to minimum required
- Rotate API keys regularly

## 🐛 Troubleshooting

### Connection Failed

**Problem**: `❌ Connection failed. Did you create the .env file?`

**Solution**: 
- Verify `.env` file exists in the notebook directory
- Check `CH_HOST` and `CH_PASSWORD` are correct
- Ensure ClickHouse instance is running and accessible

### Module Not Found

**Problem**: `ModuleNotFoundError: No module named 'clickhouse_connect'`

**Solution**:
- Run Cell 1 to install dependencies
- Run Cell 2 to restart Python
- Re-run subsequent cells

### LangSmith Not Tracing

**Problem**: Traces not appearing in LangSmith dashboard

**Solution**:
- Verify `LANGCHAIN_API_KEY` is set correctly
- Check project name in Cell 4
- Wait 5-10 seconds for traces to appear
- Refresh the LangSmith dashboard

## 📚 Additional Resources

- [ClickHouse Documentation](https://clickhouse.com/docs)
- [LangChain Documentation](https://python.langchain.com/docs/get_started/introduction)
- [LangSmith Documentation](https://docs.smith.langchain.com/)
- [LangChain Expression Language (LCEL)](https://python.langchain.com/docs/expression_language/)

## 🤝 Contributing

This is an academic project. Contributions welcome:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📄 License

MIT License - feel free to use this for academic or commercial projects.

## 👤 Author

Murali Menon  
Academic Research Project  
[GitHub Repository](https://github.com/muralimenon444/langsmith-clickhouse-academic)

## 🙏 Acknowledgments

- ClickHouse team for the incredible database
- LangChain team for the AI framework
- LangSmith team for observability tools
- Databricks for the development platform

---

**Last Updated**: April 2026  
**Version**: 1.0.0  
**Status**: Production Ready ✅
