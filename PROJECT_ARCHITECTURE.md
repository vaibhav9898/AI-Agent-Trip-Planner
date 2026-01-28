# 🏛️ Project Architecture - System Design Deep Dive

This document explains the architecture and design decisions of the AI Travel Planner project.

---

## 📋 Table of Contents
1. [High-Level Architecture](#1-high-level-architecture)
2. [Directory Structure](#2-directory-structure)
3. [Component Breakdown](#3-component-breakdown)
4. [Data Flow](#4-data-flow)
5. [Design Patterns](#5-design-patterns)
6. [Scalability Considerations](#6-scalability-considerations)

---

## 1. High-Level Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                         USER LAYER                           │
│  ┌──────────────────┐              ┌──────────────────┐    │
│  │  Streamlit UI    │              │   REST Client    │    │
│  │  (Web Interface) │              │   (Postman/curl) │    │
│  └────────┬─────────┘              └────────┬─────────┘    │
└───────────┼─────────────────────────────────┼──────────────┘
            │                                 │
            │         HTTP POST               │
            └─────────────┬───────────────────┘
                          │
┌─────────────────────────▼─────────────────────────────────┐
│                   APPLICATION LAYER                        │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              FastAPI Backend                        │  │
│  │  ┌──────────────────────────────────────────────┐   │  │
│  │  │  /query endpoint                             │   │  │
│  │  │  - Receives user query                       │   │  │
│  │  │  - Initializes GraphBuilder                  │   │  │
│  │  │  - Returns travel plan                       │   │  │
│  │  └──────────────────────────────────────────────┘   │  │
│  └────────────────────┬────────────────────────────────┘  │
└───────────────────────┼───────────────────────────────────┘
                        │
┌───────────────────────▼───────────────────────────────────┐
│                    AGENT LAYER                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │           LangGraph Workflow (GraphBuilder)        │  │
│  │                                                     │  │
│  │   ┌─────────┐    ┌──────────┐    ┌──────────┐    │  │
│  │   │  START  │───→│  Agent   │───→│   END    │    │  │
│  │   └─────────┘    │  (LLM)   │    └──────────┘    │  │
│  │                  └────┬─────┘                     │  │
│  │                       │                           │  │
│  │                       ├→ Should use tools?        │  │
│  │                       │                           │  │
│  │                  ┌────▼─────┐                     │  │
│  │                  │  Tools   │                     │  │
│  │                  │  Node    │                     │  │
│  │                  └────┬─────┘                     │  │
│  │                       │                           │  │
│  │                       └→ Back to Agent            │  │
│  └─────────────────────────────────────────────────────┘  │
└───────────────────────┬───────────────────────────────────┘
                        │
┌───────────────────────▼───────────────────────────────────┐
│                     TOOL LAYER                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │   Weather    │  │    Places    │  │  Calculator  │   │
│  │    Tools     │  │    Tools     │  │     Tool     │   │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘   │
│         │                 │                  │            │
│  ┌──────────────┐  ┌──────────────┐                      │
│  │   Currency   │  │  Arithmetic  │                      │
│  │     Tool     │  │     Tool     │                      │
│  └──────┬───────┘  └──────┬───────┘                      │
└─────────┼──────────────────┼─────────────────┼───────────┘
          │                  │                 │
┌─────────▼──────────────────▼─────────────────▼───────────┐
│                  INTEGRATION LAYER                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │ OpenWeather  │  │ Google Places│  │ ExchangeRate │   │
│  │     API      │  │     API      │  │     API      │   │
│  └──────────────┘  └──────────────┘  └──────────────┘   │
│  ┌──────────────┐  ┌──────────────┐                      │
│  │    Tavily    │  │  Groq/OpenAI │                      │
│  │     API      │  │     LLM      │                      │
│  └──────────────┘  └──────────────┘                      │
└───────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

**1. User Layer:**
- User interface (Streamlit web app)
- API clients (Postman, curl, etc.)
- Input validation and display

**2. Application Layer:**
- FastAPI REST endpoint
- Request/response handling
- CORS configuration
- Error handling

**3. Agent Layer:**
- LangGraph workflow orchestration
- LLM reasoning and decision-making
- Tool selection and coordination
- State management

**4. Tool Layer:**
- Individual tools for specific tasks
- Data formatting and validation
- Fallback mechanisms

**5. Integration Layer:**
- External API clients
- Third-party service integration
- Network communication

---

## 2. Directory Structure

```
AI-Agent-Trip-Planner/
│
├── main.py                      # FastAPI application entry point
├── streamlit_app.py             # Streamlit web interface
├── requirements.txt             # Python dependencies
├── setup.py                     # Package setup
├── pyproject.toml              # Modern Python project config
├── README.md                    # Project documentation
│
├── agent/                       # Agent layer
│   ├── __init__.py
│   └── agentic_workflow.py     # LangGraph workflow definition
│
├── tools/                       # Tool layer
│   ├── __init__.py
│   ├── weather_info_tool.py    # Weather tools
│   ├── place_search_tool.py    # Place search tools
│   ├── expense_calculator_tool.py  # Calculator tool
│   ├── currency_conversion_tool.py # Currency tool
│   └── arthamatic_op_tool.py   # Arithmetic operations
│
├── utils/                       # Integration layer
│   ├── __init__.py
│   ├── model_loader.py         # LLM provider management
│   ├── config_loader.py        # Configuration loading
│   ├── weather_info.py         # Weather API client
│   ├── place_info_search.py    # Place API clients
│   ├── expense_calculator.py   # Expense calculation logic
│   ├── currency_converter.py   # Currency API client
│   └── save_to_document.py     # Document export utility
│
├── config/                      # Configuration
│   ├── __init__.py
│   └── config.yaml             # LLM provider configs
│
├── prompt_library/              # Prompts
│   ├── __init__.py
│   └── prompt.py               # System prompt definition
│
├── logger/                      # Logging
│   ├── __init__.py
│   └── logging.py              # Logging configuration
│
└── exception/                   # Error handling
    ├── __init__.py
    └── exceptiohandling.py     # Custom exceptions
```

### Design Principles

**1. Separation of Concerns:**
- Each directory has a specific responsibility
- No mixing of tool logic and API calls
- Clear boundaries between layers

**2. Modularity:**
- Each tool is independent
- Easy to add/remove tools
- Reusable components

**3. Configuration Over Code:**
- LLM settings in YAML
- API keys in `.env`
- Easy to change without code modification

---

## 3. Component Breakdown

### 3.1 Entry Points

#### FastAPI Backend (`main.py`)

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from agent.agentic_workflow import GraphBuilder

app = FastAPI()

# Enable CORS for frontend access
app.add_middleware(CORSMiddleware, allow_origins=["*"])

@app.post("/query")
async def query_travel_agent(query: QueryRequest):
    # 1. Create agent
    graph = GraphBuilder(model_provider="groq")
    react_app = graph()
    
    # 2. Generate and save workflow visualization
    png_graph = react_app.get_graph().draw_mermaid_png()
    with open("my_graph.png", "wb") as f:
        f.write(png_graph)
    
    # 3. Execute workflow
    messages = {"messages": [query.question]}
    output = react_app.invoke(messages)
    
    # 4. Extract final response
    final_output = output["messages"][-1].content
    
    return {"answer": final_output}
```

**Key Features:**
- Single POST endpoint
- Workflow visualization
- Async support
- Error handling
- CORS enabled

#### Streamlit Frontend (`streamlit_app.py`)

```python
import streamlit as st
import requests

BASE_URL = "http://localhost:8000"

st.title("🌍 Travel Planner Agentic Application")

# Chat interface
with st.form(key="query_form", clear_on_submit=True):
    user_input = st.text_input("User Input", 
                               placeholder="e.g. Plan a trip to Goa for 5 days")
    submit_button = st.form_submit_button("Send")

if submit_button and user_input.strip():
    with st.spinner("Bot is thinking..."):
        payload = {"question": user_input}
        response = requests.post(f"{BASE_URL}/query", json=payload)
    
    if response.status_code == 200:
        answer = response.json().get("answer")
        st.markdown(answer)
```

**Key Features:**
- Simple chat interface
- Loading spinner
- Markdown rendering
- Error display

### 3.2 Agent Orchestration

#### GraphBuilder (`agent/agentic_workflow.py`)

```python
from langgraph.graph import StateGraph, MessagesState, END, START
from langgraph.prebuilt import ToolNode, tools_condition

class GraphBuilder:
    def __init__(self, model_provider: str = "groq"):
        # 1. Load LLM
        self.llm = ModelLoader(model_provider=model_provider).load_llm()
        
        # 2. Initialize tools
        self.tools = []
        self.tools.extend(WeatherInfoTool().weather_tool_list)
        self.tools.extend(PlaceSearchTool().place_search_tool_list)
        self.tools.extend(CalculatorTool().calculator_tool_list)
        self.tools.extend(CurrencyConverterTool().currency_converter_tool_list)
        
        # 3. Bind tools to LLM
        self.llm_with_tools = self.llm.bind_tools(tools=self.tools)
        
        # 4. Load system prompt
        self.system_prompt = SYSTEM_PROMPT
    
    def agent_function(self, state: MessagesState):
        """Agent node: LLM reasoning"""
        user_question = state["messages"]
        input_question = [self.system_prompt] + user_question
        response = self.llm_with_tools.invoke(input_question)
        return {"messages": [response]}
    
    def build_graph(self):
        """Build LangGraph workflow"""
        graph_builder = StateGraph(MessagesState)
        
        # Add nodes
        graph_builder.add_node("agent", self.agent_function)
        graph_builder.add_node("tools", ToolNode(tools=self.tools))
        
        # Add edges
        graph_builder.add_edge(START, "agent")
        graph_builder.add_conditional_edges("agent", tools_condition)
        graph_builder.add_edge("tools", "agent")
        graph_builder.add_edge("agent", END)
        
        # Compile
        self.graph = graph_builder.compile()
        return self.graph
```

**Architecture Decisions:**

1. **Why GraphBuilder class?**
   - Encapsulation of workflow logic
   - Reusable across different endpoints
   - Easy to test and modify

2. **Why bind_tools?**
   - LLM-native function calling
   - Automatic parameter extraction
   - No manual parsing needed

3. **Why MessagesState?**
   - Built-in message management
   - Automatic state updates
   - Compatible with LangChain

### 3.3 Tool Architecture

#### Tool Pattern

All tools follow this pattern:

```python
class ToolWrapper:
    def __init__(self):
        # Initialize API clients
        self.api_client = SomeAPIClient()
        
        # Setup tools
        self.tool_list = self._setup_tools()
    
    def _setup_tools(self) -> List:
        """Create LangChain tools"""
        
        @tool
        def tool_function(param: str) -> str:
            """Tool description for LLM"""
            # Call API
            result = self.api_client.call_api(param)
            
            # Format result
            return f"Formatted: {result}"
        
        return [tool_function, ...]
```

**Benefits:**
- Consistent structure
- Easy to add new tools
- Testable
- API client encapsulation

#### Example: Weather Tools

```python
class WeatherInfoTool:
    def __init__(self):
        load_dotenv()
        self.api_key = os.environ.get("OPENWEATHERMAP_API_KEY")
        self.weather_service = WeatherForecastTool(self.api_key)
        self.weather_tool_list = self._setup_tools()
    
    def _setup_tools(self) -> List:
        @tool
        def get_current_weather(city: str) -> str:
            """Get current weather for a city"""
            weather_data = self.weather_service.get_current_weather(city)
            if weather_data:
                temp = weather_data.get('main', {}).get('temp', 'N/A')
                desc = weather_data.get('weather', [{}])[0].get('description', 'N/A')
                return f"Current weather in {city}: {temp}°C, {desc}"
            return f"Could not fetch weather for {city}"
        
        @tool
        def get_weather_forecast(city: str) -> str:
            """Get weather forecast for a city"""
            forecast_data = self.weather_service.get_forecast_weather(city)
            if forecast_data and 'list' in forecast_data:
                forecast_summary = []
                for item in forecast_data['list']:
                    date = item['dt_txt'].split(' ')[0]
                    temp = item['main']['temp']
                    desc = item['weather'][0]['description']
                    forecast_summary.append(f"{date}: {temp}°C, {desc}")
                return f"Weather forecast for {city}:\n" + "\n".join(forecast_summary)
            return f"Could not fetch forecast for {city}"
        
        return [get_current_weather, get_weather_forecast]
```

**Design Decisions:**

1. **Closure over inheritance:**
   - Tools defined inside `_setup_tools()`
   - Access to `self.weather_service`
   - Clean LangChain integration

2. **Error handling:**
   - Graceful degradation
   - User-friendly error messages
   - No exceptions exposed to LLM

3. **Data formatting:**
   - Raw API data → Human-readable text
   - LLM can easily understand
   - Consistent format

### 3.4 Integration Layer

#### Model Loader (`utils/model_loader.py`)

```python
class ModelLoader(BaseModel):
    model_provider: Literal["groq", "openai"] = "groq"
    config: Optional[ConfigLoader] = Field(default=None, exclude=True)
    
    def model_post_init(self, __context: Any) -> None:
        self.config = ConfigLoader()
    
    def load_llm(self):
        """Load LLM based on provider"""
        if self.model_provider == "groq":
            groq_api_key = os.getenv("GROQ_API_KEY")
            model_name = self.config["llm"]["groq"]["model_name"]
            llm = ChatGroq(model=model_name, api_key=groq_api_key)
        
        elif self.model_provider == "openai":
            openai_api_key = os.getenv("OPENAI_API_KEY")
            model_name = self.config["llm"]["openai"]["model_name"]
            llm = ChatOpenAI(model_name=model_name, api_key=openai_api_key)
        
        return llm
```

**Design Decisions:**

1. **Pydantic BaseModel:**
   - Type validation
   - Configuration management
   - IDE autocomplete

2. **Literal type:**
   - Only "groq" or "openai" allowed
   - Compile-time checking
   - Clear API

3. **Config-driven:**
   - Model names in YAML
   - Easy to update
   - No code changes needed

#### API Clients

**Weather Client (`utils/weather_info.py`):**
```python
class WeatherForecastTool:
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = "https://api.openweathermap.org/data/2.5"
    
    def get_current_weather(self, city: str) -> dict:
        url = f"{self.base_url}/weather?q={city}&appid={self.api_key}&units=metric"
        response = requests.get(url)
        response.raise_for_status()
        return response.json()
    
    def get_forecast_weather(self, city: str) -> dict:
        url = f"{self.base_url}/forecast?q={city}&appid={self.api_key}&units=metric"
        response = requests.get(url)
        response.raise_for_status()
        return response.json()
```

**Place Search Client (`utils/place_info_search.py`):**
```python
class GooglePlaceSearchTool:
    def __init__(self, api_key: str):
        self.places_wrapper = GooglePlacesAPIWrapper(gplaces_api_key=api_key)
        self.places_tool = GooglePlacesTool(api_wrapper=self.places_wrapper)
    
    def google_search_attractions(self, place: str) -> dict:
        return self.places_tool.run(f"top attractive places in and around {place}")

class TavilyPlaceSearchTool:
    def tavily_search_attractions(self, place: str) -> dict:
        tavily_tool = TavilySearch(topic="general", include_answer="advanced")
        result = tavily_tool.invoke({"query": f"top attractive places in and around {place}"})
        return result.get("answer", result)
```

**Design Pattern:**
- Wrapper classes for API clients
- Consistent interface
- LangChain integration
- Error handling

---

## 4. Data Flow

### Complete Request Flow

```
1. User Input (Streamlit)
   ↓
   User types: "Plan a trip to Paris for 3 days"
   ↓

2. Frontend Processing (streamlit_app.py)
   ↓
   Creates JSON: {"question": "Plan a trip to Paris for 3 days"}
   ↓
   Sends POST request to: http://localhost:8000/query
   ↓

3. Backend Receives (main.py)
   ↓
   FastAPI endpoint: @app.post("/query")
   ↓
   Validates request with Pydantic
   ↓

4. Graph Initialization
   ↓
   GraphBuilder(model_provider="groq")
   ├── Loads Groq LLM
   ├── Loads system prompt
   ├── Initializes 8 tools:
   │   ├── get_current_weather
   │   ├── get_weather_forecast
   │   ├── search_attractions
   │   ├── search_restaurants
   │   ├── search_activities
   │   ├── search_transportation
   │   ├── calculate_expense
   │   └── convert_currency
   └── Binds tools to LLM
   ↓

5. Graph Execution
   ↓
   react_app.invoke({"messages": [query]})
   ↓
   
   Step 1: START → Agent
   ├── State: {"messages": ["Plan a trip to Paris for 3 days"]}
   ├── System prompt added
   ├── LLM analyzes: "Need weather, attractions, restaurants, costs"
   └── LLM decides to call tools
   ↓
   
   Step 2: Agent → Tools (conditional edge)
   ├── tools_condition detects tool calls
   └── Routes to Tools node
   ↓
   
   Step 3: Tools Node Execution
   ├── Tool 1: get_weather_forecast("Paris")
   │   ├── WeatherInfoTool called
   │   ├── OpenWeatherMap API request
   │   ├── Response: "Paris weather: 15°C, partly cloudy..."
   │   └── Result stored in state
   │
   ├── Tool 2: search_attractions("Paris")
   │   ├── PlaceSearchTool called
   │   ├── Try: Google Places API
   │   ├── Success: Returns attractions
   │   └── Result stored in state
   │
   ├── Tool 3: search_restaurants("Paris")
   │   └── Similar process...
   │
   └── All tool results added to messages
   ↓
   
   Step 4: Tools → Agent
   ├── Agent receives tool results
   ├── LLM analyzes results
   └── LLM decides if more tools needed
   ↓
   
   Step 5: Agent → Tools (possibly, cyclic)
   └── May call calculate_expense, convert_currency, etc.
   ↓
   
   Step 6: Tools → Agent (final iteration)
   ├── Agent has all data
   ├── LLM generates comprehensive travel plan
   └── No more tool calls
   ↓
   
   Step 7: Agent → END
   └── tools_condition sees no tool calls
   └── Routes to END
   ↓

6. Response Extraction (main.py)
   ↓
   output["messages"][-1].content
   ↓
   Extracts final AI message (the travel plan)
   ↓

7. API Response
   ↓
   Returns: {"answer": "# Paris Travel Plan\n\n## Day 1..."}
   ↓

8. Frontend Display (streamlit_app.py)
   ↓
   st.markdown(answer)
   ↓
   Renders formatted travel plan
   ↓

9. User Sees Result
   └── Complete travel plan with weather, attractions, costs, etc.
```

### State Evolution

**Initial State:**
```python
{
  "messages": [
    HumanMessage(content="Plan a trip to Paris for 3 days")
  ]
}
```

**After Agent Node (1st time):**
```python
{
  "messages": [
    HumanMessage(content="Plan a trip to Paris for 3 days"),
    AIMessage(
      content="",
      tool_calls=[
        {"name": "get_weather_forecast", "args": {"city": "Paris"}},
        {"name": "search_attractions", "args": {"place": "Paris"}}
      ]
    )
  ]
}
```

**After Tools Node:**
```python
{
  "messages": [
    HumanMessage(content="Plan a trip to Paris for 3 days"),
    AIMessage(content="", tool_calls=[...]),
    ToolMessage(content="Paris weather: 15°C, partly cloudy...", name="get_weather_forecast"),
    ToolMessage(content="Attractions: Eiffel Tower, Louvre...", name="search_attractions")
  ]
}
```

**After Agent Node (2nd time, final):**
```python
{
  "messages": [
    HumanMessage(content="Plan a trip to Paris for 3 days"),
    AIMessage(content="", tool_calls=[...]),
    ToolMessage(content="Weather data...", name="get_weather_forecast"),
    ToolMessage(content="Attractions...", name="search_attractions"),
    AIMessage(content="# Paris Travel Plan\n\n## Day 1...\n\nBased on the weather...")
  ]
}
```

---

## 5. Design Patterns

### 5.1 Factory Pattern (ModelLoader)

**Problem:** Need to create different LLM instances based on configuration

**Solution:** Factory method

```python
class ModelLoader:
    def load_llm(self):
        """Factory method"""
        if self.model_provider == "groq":
            return ChatGroq(...)
        elif self.model_provider == "openai":
            return ChatOpenAI(...)
```

**Benefits:**
- Centralized LLM creation
- Easy to add new providers
- Configuration-driven

### 5.2 Builder Pattern (GraphBuilder)

**Problem:** Complex graph construction with many components

**Solution:** Builder pattern

```python
class GraphBuilder:
    def __init__(self, model_provider):
        # Setup components
        self.llm = ...
        self.tools = ...
        self.system_prompt = ...
    
    def build_graph(self):
        # Build graph step by step
        graph_builder = StateGraph(MessagesState)
        graph_builder.add_node(...)
        graph_builder.add_edge(...)
        return graph_builder.compile()
```

**Benefits:**
- Step-by-step construction
- Readable and maintainable
- Reusable components

### 5.3 Wrapper Pattern (Tool Classes)

**Problem:** Need to adapt external APIs to LangChain tool format

**Solution:** Wrapper classes

```python
class WeatherInfoTool:
    def __init__(self):
        # Wrap external API
        self.weather_service = WeatherForecastTool(api_key)
    
    def _setup_tools(self):
        @tool
        def get_current_weather(city: str) -> str:
            # Adapter: External API → LangChain tool
            raw_data = self.weather_service.get_current_weather(city)
            return formatted_data
```

**Benefits:**
- Clean separation
- Testable
- Reusable API clients

### 5.4 Strategy Pattern (Fallback Mechanism)

**Problem:** Need to try multiple search providers

**Solution:** Strategy pattern with fallback

```python
@tool
def search_attractions(place: str) -> str:
    try:
        # Strategy 1: Google Places
        return google_search(place)
    except Exception:
        # Strategy 2: Tavily (fallback)
        return tavily_search(place)
```

**Benefits:**
- Resilience
- Flexibility
- Graceful degradation

---

## 6. Scalability Considerations

### Current Architecture Limitations

1. **Synchronous Processing:**
   - Each request blocks until complete
   - Can't handle multiple users efficiently

2. **No Caching:**
   - Same queries hit APIs every time
   - Costs money and time

3. **No Rate Limiting:**
   - Could exhaust API quotas
   - No protection against abuse

4. **In-Memory State:**
   - No persistence
   - Can't resume interrupted workflows

### Scaling Solutions

#### 1. Async Processing

**Current:**
```python
@app.post("/query")
async def query_travel_agent(query: QueryRequest):
    output = react_app.invoke(messages)
    return {"answer": output}
```

**Improved:**
```python
from fastapi import BackgroundTasks
import asyncio

@app.post("/query")
async def query_travel_agent(query: QueryRequest, background_tasks: BackgroundTasks):
    # Return job ID immediately
    job_id = create_job()
    
    # Process in background
    background_tasks.add_task(process_query, job_id, query)
    
    return {"job_id": job_id, "status": "processing"}

@app.get("/status/{job_id}")
async def get_status(job_id: str):
    # Check job status
    return {"status": "...", "result": "..."}
```

#### 2. Caching Layer

**Redis Integration:**
```python
import redis

redis_client = redis.Redis(host='localhost', port=6379)

@tool
def get_weather_forecast(city: str) -> str:
    # Check cache
    cache_key = f"weather:{city}"
    cached = redis_client.get(cache_key)
    
    if cached:
        return cached.decode('utf-8')
    
    # Fetch from API
    result = fetch_weather(city)
    
    # Cache for 1 hour
    redis_client.setex(cache_key, 3600, result)
    
    return result
```

#### 3. Rate Limiting

**Using slowapi:**
```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@app.post("/query")
@limiter.limit("10/minute")
async def query_travel_agent(request: Request, query: QueryRequest):
    # Limited to 10 requests per minute per IP
    ...
```

#### 4. Database for Persistence

**Using PostgreSQL:**
```python
from sqlalchemy import create_engine, Column, String, JSON
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class TravelPlan(Base):
    __tablename__ = 'travel_plans'
    
    id = Column(String, primary_key=True)
    query = Column(String)
    result = Column(JSON)
    state = Column(JSON)  # LangGraph state

# Save state at each step
def save_state(state):
    db.session.add(TravelPlan(state=state))
    db.session.commit()
```

#### 5. Load Balancing

**Using Multiple Workers:**
```bash
# Run with Gunicorn
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app
```

**Benefits:**
- Multiple processes
- CPU utilization
- Better throughput

#### 6. Monitoring and Logging

**Using Prometheus + Grafana:**
```python
from prometheus_client import Counter, Histogram
import time

request_counter = Counter('requests_total', 'Total requests')
response_time = Histogram('response_time_seconds', 'Response time')

@app.post("/query")
async def query_travel_agent(query: QueryRequest):
    request_counter.inc()
    
    start = time.time()
    result = process_query(query)
    response_time.observe(time.time() - start)
    
    return result
```

---

## 🎯 Interview Questions: Architecture

### Q1: "Why separate tools from utils?"
**Answer:**
- **tools/**: LangChain-specific tool wrappers
- **utils/**: Reusable API clients (could be used outside LangChain)
- **Benefit**: Can use API clients in other parts of the app

### Q2: "Why use LangGraph instead of simple function calls?"
**Answer:**
- **State Management**: Automatic message tracking
- **Conditional Routing**: Built-in tool calling logic
- **Visualization**: Can see workflow as graph
- **Extensibility**: Easy to add nodes/edges
- **Error Handling**: Framework handles edge cases

### Q3: "How would you add a new tool?"
**Answer:**
```python
# 1. Create API client in utils/
class NewAPIClient:
    def call_api(self, param):
        return result

# 2. Create tool wrapper in tools/
class NewTool:
    def _setup_tools(self):
        @tool
        def new_tool_function(param: str) -> str:
            """Tool description"""
            return self.api_client.call_api(param)
        return [new_tool_function]

# 3. Add to GraphBuilder
self.tools.extend(NewTool().tool_list)
```

### Q4: "What are potential bottlenecks?"
**Answer:**
1. **LLM API calls**: Slowest part (1-5 seconds)
2. **External API calls**: Weather, Places APIs (0.5-2 seconds each)
3. **Synchronous processing**: Blocks on each request
4. **No caching**: Repeated queries re-fetch data

**Solutions**: Async, caching, batching, streaming responses

---

**Next:** Read `STEP_BY_STEP_GUIDE.md` to trace actual code execution!
