# 🔍 Step-by-Step Code Execution Guide

This document traces the complete execution flow of the AI Travel Planner, showing exactly what happens when a user makes a request.

---

## 📋 Table of Contents
1. [Example Scenario](#1-example-scenario)
2. [Execution Trace](#2-execution-trace)
3. [Code Walkthrough](#3-code-walkthrough)
4. [Decision Points](#4-decision-points)
5. [Common Execution Patterns](#5-common-execution-patterns)
6. [Debugging Guide](#6-debugging-guide)

---

## 1. Example Scenario

**User Input:** "Plan a 3-day trip to Tokyo in March with a budget of $2000"

**Expected Output:**
- 3-day itinerary
- Weather information for March
- Tourist attractions
- Restaurant recommendations
- Cost breakdown
- Currency conversion (USD to JPY)

Let's trace this execution step by step.

---

## 2. Execution Trace

### Timeline Overview

```
T=0ms:    User submits form in Streamlit
T=50ms:   HTTP POST request sent to FastAPI
T=100ms:  FastAPI receives request, initializes GraphBuilder
T=200ms:  LangGraph workflow starts
T=300ms:  LLM analyzes query, decides to call tools
T=500ms:  Tools execute (parallel API calls)
T=2000ms: Tools return results
T=2100ms: LLM receives results, may call more tools
T=3500ms: LLM generates final response
T=3600ms: Response sent back to Streamlit
T=3650ms: Streamlit displays formatted result
```

---

## 3. Code Walkthrough

### Step 1: User Interaction (Streamlit)

**File:** `streamlit_app.py`

**Code:**
```python
with st.form(key="query_form", clear_on_submit=True):
    user_input = st.text_input("User Input", 
                               placeholder="e.g. Plan a trip to Goa for 5 days")
    submit_button = st.form_submit_button("Send")
```

**What happens:**
1. User types: "Plan a 3-day trip to Tokyo in March with a budget of $2000"
2. User clicks "Send" button
3. Form data captured

**Execution Flow:**
```python
if submit_button and user_input.strip():
    # user_input = "Plan a 3-day trip to Tokyo in March with a budget of $2000"
    
    # Show loading spinner
    with st.spinner("Bot is thinking..."):
        # Prepare payload
        payload = {"question": user_input}
        # payload = {"question": "Plan a 3-day trip to Tokyo in March..."}
        
        # Make HTTP request
        response = requests.post(f"{BASE_URL}/query", json=payload)
        # BASE_URL = "http://localhost:8000"
        # Sends POST to: http://localhost:8000/query
```

**Network Request:**
```http
POST /query HTTP/1.1
Host: localhost:8000
Content-Type: application/json

{
  "question": "Plan a 3-day trip to Tokyo in March with a budget of $2000"
}
```

---

### Step 2: FastAPI Receives Request

**File:** `main.py`

**Code:**
```python
@app.post("/query")
async def query_travel_agent(query: QueryRequest):
```

**What happens:**
1. FastAPI router matches `/query` endpoint
2. Request body validated against `QueryRequest` model
3. Function executes

**Request Validation:**
```python
class QueryRequest(BaseModel):
    question: str

# Pydantic validates:
# - "question" field exists
# - "question" is a string
# - Creates QueryRequest object

# query.question = "Plan a 3-day trip to Tokyo in March with a budget of $2000"
```

---

### Step 3: GraphBuilder Initialization

**File:** `main.py` → `agent/agentic_workflow.py`

**Code in main.py:**
```python
try:
    print(query)  # Prints QueryRequest object
    
    # Initialize GraphBuilder with Groq as LLM provider
    graph = GraphBuilder(model_provider="groq")
```

**Code in agentic_workflow.py:**
```python
class GraphBuilder:
    def __init__(self, model_provider: str = "groq"):
        # Step 3.1: Load LLM
        self.model_loader = ModelLoader(model_provider=model_provider)
        self.llm = self.model_loader.load_llm()
```

**Step 3.1: Load LLM**

**File:** `utils/model_loader.py`

```python
class ModelLoader(BaseModel):
    model_provider: Literal["groq", "openai"] = "groq"
    
    def model_post_init(self, __context: Any) -> None:
        # Load configuration from YAML
        self.config = ConfigLoader()
        # ConfigLoader reads config/config.yaml
    
    def load_llm(self):
        print("LLM loading...")
        print(f"Loading model from provider: {self.model_provider}")
        
        if self.model_provider == "groq":
            print("Loading LLM from Groq..............")
            
            # Get API key from environment
            groq_api_key = os.getenv("GROQ_API_KEY")
            
            # Get model name from config
            model_name = self.config["llm"]["groq"]["model_name"]
            # model_name = "deepseek-r1-distill-llama-70b"
            
            # Initialize LLM
            llm = ChatGroq(model=model_name, api_key=groq_api_key)
            
        return llm
```

**Output:**
```
LLM loading...
Loading model from provider: groq
Loading LLM from Groq..............
```

**Step 3.2: Initialize Tools**

**Back in agentic_workflow.py:**
```python
def __init__(self, model_provider: str = "groq"):
    # ... LLM loaded ...
    
    # Step 3.2: Initialize empty tools list
    self.tools = []
    
    # Initialize weather tools
    self.weather_tools = WeatherInfoTool()
    
    # Initialize place search tools
    self.place_search_tools = PlaceSearchTool()
    
    # Initialize calculator tools
    self.calculator_tools = CalculatorTool()
    
    # Initialize currency converter tools
    self.currency_converter_tools = CurrencyConverterTool()
    
    # Extend tools list with all tools
    self.tools.extend([
        *self.weather_tools.weather_tool_list,
        *self.place_search_tools.place_search_tool_list,
        *self.calculator_tools.calculator_tool_list,
        *self.currency_converter_tools.currency_converter_tool_list
    ])
    
    # At this point, self.tools contains 8 tools:
    # 1. get_current_weather
    # 2. get_weather_forecast
    # 3. search_attractions
    # 4. search_restaurants
    # 5. search_activities
    # 6. search_transportation
    # 7. calculate_expense
    # 8. convert_currency
```

**Weather Tools Initialization:**

**File:** `tools/weather_info_tool.py`

```python
class WeatherInfoTool:
    def __init__(self):
        # Load environment variables
        load_dotenv()
        
        # Get API key
        self.api_key = os.environ.get("OPENWEATHERMAP_API_KEY")
        
        # Initialize weather service
        self.weather_service = WeatherForecastTool(self.api_key)
        
        # Setup tools
        self.weather_tool_list = self._setup_tools()
    
    def _setup_tools(self) -> List:
        """Setup all tools for weather forecast"""
        
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
            # ... implementation ...
        
        return [get_current_weather, get_weather_forecast]
```

**Note:** The `@tool` decorator converts regular Python functions into LangChain tools that:
- Extract docstring as description
- Infer parameters from type hints
- Can be called by LLMs

**Similar process for other tools:**
- `PlaceSearchTool` creates 4 tools
- `CalculatorTool` creates 1 tool
- `CurrencyConverterTool` creates 1 tool

**Step 3.3: Bind Tools to LLM**

**Back in agentic_workflow.py:**
```python
def __init__(self, model_provider: str = "groq"):
    # ... Tools initialized ...
    
    # Bind tools to LLM
    self.llm_with_tools = self.llm.bind_tools(tools=self.tools)
    
    # This tells the LLM:
    # "You have access to these 8 functions. You can call them when needed."
```

**What bind_tools does:**
```python
# LLM now knows about tools:
# Tool 1: get_current_weather(city: str) -> str
#         Description: "Get current weather for a city"
# Tool 2: get_weather_forecast(city: str) -> str
#         Description: "Get weather forecast for a city"
# ... and so on
```

**Step 3.4: Load System Prompt**

```python
def __init__(self, model_provider: str = "groq"):
    # ... Tools bound ...
    
    # Load system prompt
    self.system_prompt = SYSTEM_PROMPT
```

**File:** `prompt_library/prompt.py`

```python
from langchain_core.messages import SystemMessage

SYSTEM_PROMPT = SystemMessage(
    content="""You are a helpful AI Travel Agent and Expense Planner. 
    You help users plan trips to any place worldwide with real-time data from internet.
    
    Provide complete, comprehensive and a detailed travel plan. Always try to provide two
    plans, one for the generic tourist places, another for more off-beat locations...
    
    Use the available tools to gather information and make detailed cost breakdowns.
    Provide everything in one comprehensive response formatted in clean Markdown.
    """
)
```

**Initialization Complete!**

---

### Step 4: Build Graph

**File:** `main.py`

```python
# Build the graph
react_app = graph()  # Calls GraphBuilder.__call__()
```

**File:** `agent/agentic_workflow.py`

```python
def __call__(self):
    return self.build_graph()

def build_graph(self):
    """Build LangGraph workflow"""
    
    # Step 4.1: Create graph builder
    graph_builder = StateGraph(MessagesState)
    
    # Step 4.2: Add agent node
    graph_builder.add_node("agent", self.agent_function)
    
    # Step 4.3: Add tools node
    graph_builder.add_node("tools", ToolNode(tools=self.tools))
    
    # Step 4.4: Add edges
    graph_builder.add_edge(START, "agent")
    graph_builder.add_conditional_edges("agent", tools_condition)
    graph_builder.add_edge("tools", "agent")
    graph_builder.add_edge("agent", END)
    
    # Step 4.5: Compile graph
    self.graph = graph_builder.compile()
    
    return self.graph
```

**Graph Structure:**
```
START
  ↓
agent (self.agent_function)
  ↓
conditional: tools_condition
  ↓         ↓
 tools     END
  ↓
agent
  ↓
conditional: tools_condition
  (cycles back)
```

**Step 4.6: Generate Graph Visualization**

**File:** `main.py`

```python
# Generate graph PNG
png_graph = react_app.get_graph().draw_mermaid_png()

# Save to file
with open("my_graph.png", "wb") as f:
    f.write(png_graph)

print(f"Graph saved as 'my_graph.png' in {os.getcwd()}")
```

---

### Step 5: Execute Workflow

**File:** `main.py`

```python
# Prepare messages
messages = {"messages": [query.question]}
# messages = {
#     "messages": ["Plan a 3-day trip to Tokyo in March with a budget of $2000"]
# }

# Execute graph
output = react_app.invoke(messages)
```

**What happens inside invoke():**

The graph executes node by node, following the edges.

---

### Step 6: First Agent Node Execution

**File:** `agent/agentic_workflow.py`

```python
def agent_function(self, state: MessagesState):
    """Main agent function"""
    
    # Step 6.1: Get messages from state
    user_question = state["messages"]
    # user_question = ["Plan a 3-day trip to Tokyo in March with a budget of $2000"]
    
    # Step 6.2: Prepend system prompt
    input_question = [self.system_prompt] + user_question
    # input_question = [
    #     SystemMessage(content="You are a helpful AI Travel Agent..."),
    #     "Plan a 3-day trip to Tokyo in March with a budget of $2000"
    # ]
    
    # Step 6.3: Call LLM with tools
    response = self.llm_with_tools.invoke(input_question)
    
    # Step 6.4: Return updated state
    return {"messages": [response]}
```

**Inside self.llm_with_tools.invoke():**

**Request to Groq API:**
```json
{
  "model": "deepseek-r1-distill-llama-70b",
  "messages": [
    {
      "role": "system",
      "content": "You are a helpful AI Travel Agent and Expense Planner..."
    },
    {
      "role": "user",
      "content": "Plan a 3-day trip to Tokyo in March with a budget of $2000"
    }
  ],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_current_weather",
        "description": "Get current weather for a city",
        "parameters": {
          "type": "object",
          "properties": {
            "city": {"type": "string"}
          },
          "required": ["city"]
        }
      }
    },
    {
      "type": "function",
      "function": {
        "name": "get_weather_forecast",
        "description": "Get weather forecast for a city",
        "parameters": {
          "type": "object",
          "properties": {
            "city": {"type": "string"}
          },
          "required": ["city"]
        }
      }
    },
    // ... 6 more tools ...
  ]
}
```

**LLM Response:**
```json
{
  "role": "assistant",
  "content": "",
  "tool_calls": [
    {
      "id": "call_1",
      "type": "function",
      "function": {
        "name": "get_weather_forecast",
        "arguments": "{\"city\": \"Tokyo\"}"
      }
    },
    {
      "id": "call_2",
      "type": "function",
      "function": {
        "name": "search_attractions",
        "arguments": "{\"place\": \"Tokyo\"}"
      }
    },
    {
      "id": "call_3",
      "type": "function",
      "function": {
        "name": "search_restaurants",
        "arguments": "{\"place\": \"Tokyo\"}"
      }
    },
    {
      "id": "call_4",
      "type": "function",
      "function": {
        "name": "convert_currency",
        "arguments": "{\"amount\": 2000, \"from_currency\": \"USD\", \"to_currency\": \"JPY\"}"
      }
    }
  ]
}
```

**LLM Reasoning (what the model "thinks"):**
1. User wants to plan a Tokyo trip
2. Need weather for Tokyo in March
3. Need tourist attractions
4. Need restaurants
5. Need to convert $2000 to Japanese Yen
6. Will call 4 tools

**State after Agent Node:**
```python
{
  "messages": [
    HumanMessage(content="Plan a 3-day trip to Tokyo..."),
    AIMessage(
      content="",
      tool_calls=[
        {"name": "get_weather_forecast", "args": {"city": "Tokyo"}},
        {"name": "search_attractions", "args": {"place": "Tokyo"}},
        {"name": "search_restaurants", "args": {"place": "Tokyo"}},
        {"name": "convert_currency", "args": {"amount": 2000, "from_currency": "USD", "to_currency": "JPY"}}
      ]
    )
  ]
}
```

---

### Step 7: Conditional Edge - Route to Tools

**Code:** (built-in to LangGraph)

```python
graph_builder.add_conditional_edges("agent", tools_condition)
```

**tools_condition function:**
```python
def tools_condition(state):
    """Check if agent wants to use tools"""
    last_message = state["messages"][-1]
    
    if last_message.tool_calls:
        # Agent wants to use tools
        return "tools"
    else:
        # Agent is done
        return END
```

**In our case:**
- Last message has `tool_calls`
- Returns `"tools"`
- Graph routes to Tools Node

---

### Step 8: Tools Node Execution

**Code:** (built-in ToolNode from LangGraph)

```python
graph_builder.add_node("tools", ToolNode(tools=self.tools))
```

**What ToolNode does:**
1. Extracts tool calls from last message
2. For each tool call:
   - Finds the tool function by name
   - Calls it with the provided arguments
   - Collects the result
3. Returns all results as ToolMessages

**Execution:**

**Tool Call 1: get_weather_forecast("Tokyo")**

**File:** `tools/weather_info_tool.py`

```python
@tool
def get_weather_forecast(city: str) -> str:
    """Get weather forecast for a city"""
    # city = "Tokyo"
    
    # Call weather service
    forecast_data = self.weather_service.get_forecast_weather(city)
```

**File:** `utils/weather_info.py`

```python
def get_forecast_weather(self, city: str) -> dict:
    # city = "Tokyo"
    
    # Build URL
    url = f"{self.base_url}/forecast?q={city}&appid={self.api_key}&units=metric"
    # url = "https://api.openweathermap.org/data/2.5/forecast?q=Tokyo&appid=...&units=metric"
    
    # Make HTTP request
    response = requests.get(url)
    response.raise_for_status()
    
    # Return JSON
    return response.json()
```

**OpenWeatherMap API Response:**
```json
{
  "list": [
    {
      "dt_txt": "2026-03-15 12:00:00",
      "main": {"temp": 12.5},
      "weather": [{"description": "partly cloudy"}]
    },
    {
      "dt_txt": "2026-03-15 15:00:00",
      "main": {"temp": 14.2},
      "weather": [{"description": "clear sky"}]
    },
    // ... more forecast data ...
  ]
}
```

**Back in weather_info_tool.py:**
```python
if forecast_data and 'list' in forecast_data:
    forecast_summary = []
    
    for item in forecast_data['list']:
        date = item['dt_txt'].split(' ')[0]  # "2026-03-15"
        temp = item['main']['temp']  # 12.5
        desc = item['weather'][0]['description']  # "partly cloudy"
        
        forecast_summary.append(f"{date}: {temp} degree celcius, {desc}")
    
    result = f"Weather forecast for {city}:\n" + "\n".join(forecast_summary)
    return result
```

**Tool 1 Result:**
```
Weather forecast for Tokyo:
2026-03-15: 12.5 degree celcius, partly cloudy
2026-03-15: 14.2 degree celcius, clear sky
2026-03-16: 11.8 degree celcius, light rain
...
```

**Tool Call 2: search_attractions("Tokyo")**

**File:** `tools/place_search_tool.py`

```python
@tool
def search_attractions(place: str) -> str:
    """Search attractions of a place"""
    # place = "Tokyo"
    
    try:
        # Try Google Places first
        attraction_result = self.google_places_search.google_search_attractions(place)
        
        if attraction_result:
            return f"Following are the attractions of {place} as suggested by google: {attraction_result}"
    
    except Exception as e:
        # Fallback to Tavily
        tavily_result = self.tavily_search.tavily_search_attractions(place)
        return f"Google cannot find the details due to {e}. \nFollowing are the attractions of {place}: {tavily_result}"
```

**File:** `utils/place_info_search.py`

```python
class GooglePlaceSearchTool:
    def google_search_attractions(self, place: str) -> dict:
        # place = "Tokyo"
        
        # Use Google Places API
        return self.places_tool.run(f"top attractive places in and around {place}")
        # Query: "top attractive places in and around Tokyo"
```

**Google Places API Response (simplified):**
```json
{
  "results": [
    {
      "name": "Tokyo Skytree",
      "rating": 4.5,
      "vicinity": "Sumida, Tokyo"
    },
    {
      "name": "Senso-ji Temple",
      "rating": 4.6,
      "vicinity": "Asakusa, Tokyo"
    },
    {
      "name": "Meiji Shrine",
      "rating": 4.7,
      "vicinity": "Shibuya, Tokyo"
    },
    // ... more attractions ...
  ]
}
```

**Tool 2 Result:**
```
Following are the attractions of Tokyo as suggested by google:
1. Tokyo Skytree (Rating: 4.5) - Sumida, Tokyo
2. Senso-ji Temple (Rating: 4.6) - Asakusa, Tokyo
3. Meiji Shrine (Rating: 4.7) - Shibuya, Tokyo
...
```

**Tool Call 3: search_restaurants("Tokyo")**

Similar process, returns restaurant data.

**Tool Call 4: convert_currency(2000, "USD", "JPY")**

**File:** `tools/currency_conversion_tool.py`

```python
@tool
def convert_currency(amount: float, from_currency: str, to_currency: str) -> str:
    """Convert currency from one to another"""
    # amount = 2000, from_currency = "USD", to_currency = "JPY"
    
    converted_amount = self.currency_converter.convert_currency(
        amount, from_currency, to_currency
    )
    
    return f"{amount} {from_currency} = {converted_amount} {to_currency}"
```

**File:** `utils/currency_converter.py`

```python
def convert_currency(self, amount, from_currency, to_currency):
    # API call to ExchangeRate-API
    url = f"{self.base_url}/latest/{from_currency}"
    response = requests.get(url)
    data = response.json()
    
    rate = data['conversion_rates'][to_currency]
    # rate = 149.5 (1 USD = 149.5 JPY)
    
    converted_amount = amount * rate
    # converted_amount = 2000 * 149.5 = 299000
    
    return converted_amount
```

**Tool 4 Result:**
```
2000 USD = 299000 JPY
```

**State after Tools Node:**
```python
{
  "messages": [
    HumanMessage(content="Plan a 3-day trip to Tokyo..."),
    AIMessage(content="", tool_calls=[...]),
    ToolMessage(
      content="Weather forecast for Tokyo:\n2026-03-15: 12.5°C, partly cloudy\n...",
      name="get_weather_forecast"
    ),
    ToolMessage(
      content="Following are the attractions of Tokyo as suggested by google:\n1. Tokyo Skytree...",
      name="search_attractions"
    ),
    ToolMessage(
      content="Following are the restaurants of Tokyo as suggested by google:\n1. Sukiyabashi Jiro...",
      name="search_restaurants"
    ),
    ToolMessage(
      content="2000 USD = 299000 JPY",
      name="convert_currency"
    )
  ]
}
```

---

### Step 9: Back to Agent Node (2nd Time)

**Edge:** `tools → agent`

**Agent function executes again:**

```python
def agent_function(self, state: MessagesState):
    user_question = state["messages"]
    # Now includes:
    # - Original question
    # - Previous AI message with tool calls
    # - All tool results
    
    input_question = [self.system_prompt] + user_question
    
    # Call LLM with all context
    response = self.llm_with_tools.invoke(input_question)
    
    return {"messages": [response]}
```

**LLM Request (simplified):**
```json
{
  "messages": [
    {"role": "system", "content": "You are a helpful AI Travel Agent..."},
    {"role": "user", "content": "Plan a 3-day trip to Tokyo..."},
    {"role": "assistant", "content": "", "tool_calls": [...]},
    {"role": "tool", "name": "get_weather_forecast", "content": "Weather forecast..."},
    {"role": "tool", "name": "search_attractions", "content": "Attractions..."},
    {"role": "tool", "name": "search_restaurants", "content": "Restaurants..."},
    {"role": "tool", "name": "convert_currency", "content": "2000 USD = 299000 JPY"}
  ],
  "tools": [...]
}
```

**LLM Reasoning:**
1. I have weather data for Tokyo
2. I have attractions
3. I have restaurants
4. I have currency conversion
5. User wants 3-day itinerary with $2000 budget
6. I might need more info - let me call calculate_expense

**LLM Response (may call more tools or finish):**

**Option A: Call More Tools**
```json
{
  "tool_calls": [
    {
      "function": {
        "name": "calculate_expense",
        "arguments": "{\"expression\": \"299000 / 3\"}"
      }
    }
  ]
}
```

If Option A, graph cycles back to Tools Node.

**Option B: Generate Final Response**
```json
{
  "role": "assistant",
  "content": "# 🗾 Tokyo Travel Plan - 3 Days\n\n## Budget Overview\n- Total Budget: $2,000 USD (¥299,000 JPY)\n- Per Day: ~¥99,666\n\n## Weather in March\nTokyo in March typically experiences...\n\n## Day 1: Traditional Tokyo\n### Morning\n- Visit Senso-ji Temple...\n\n### Afternoon\n- Explore Asakusa district...\n\n### Evening\n- Dinner at Sukiyabashi Jiro...\n\n## Day 2: Modern Tokyo\n...\n\n## Day 3: Nature & Shopping\n...\n\n## Cost Breakdown\n- Accommodation: ¥90,000 (3 nights @ ¥30,000/night)\n- Food: ¥60,000 (~¥20,000/day)\n- Transportation: ¥30,000 (3-day metro pass + taxi)\n- Attractions: ¥40,000 (entry fees)\n- Shopping: ¥79,000\n**Total: ¥299,000**\n\n..."
}
```

If Option B, no tool calls, graph goes to END.

---

### Step 10: Conditional Edge - Check Again

**tools_condition executes:**

```python
last_message = state["messages"][-1]

if last_message.tool_calls:
    # More tools needed, cycle back
    return "tools"
else:
    # Done, go to END
    return END
```

**In our case (Option B):**
- No tool calls
- Returns `END`
- Graph execution completes

---

### Step 11: Extract Final Response

**File:** `main.py`

```python
# output = {
#     "messages": [
#         HumanMessage(...),
#         AIMessage(..., tool_calls=[...]),
#         ToolMessage(...),
#         ToolMessage(...),
#         ToolMessage(...),
#         ToolMessage(...),
#         AIMessage(content="# 🗾 Tokyo Travel Plan...")
#     ]
# }

if isinstance(output, dict) and "messages" in output:
    final_output = output["messages"][-1].content
    # final_output = "# 🗾 Tokyo Travel Plan - 3 Days\n\n..."
else:
    final_output = str(output)

return {"answer": final_output}
```

**FastAPI Response:**
```json
{
  "answer": "# 🗾 Tokyo Travel Plan - 3 Days\n\n## Budget Overview\n- Total Budget: $2,000 USD (¥299,000 JPY)..."
}
```

---

### Step 12: Streamlit Displays Result

**File:** `streamlit_app.py`

```python
if response.status_code == 200:
    answer = response.json().get("answer", "No answer returned.")
    # answer = "# 🗾 Tokyo Travel Plan..."
    
    markdown_content = f"""# 🌍 AI Travel Plan

    # **Generated:** {datetime.datetime.now().strftime('%Y-%m-%d at %H:%M')}  
    # **Created by:** Atriyo's Travel Agent

    ---

    {answer}

    ---

    *This travel plan was generated by AI. Please verify all information...*
    """
    
    st.markdown(markdown_content)
```

**User sees:**
- Formatted markdown
- Complete travel plan
- Weather info
- Attractions
- Restaurants
- Cost breakdown

---

## 4. Decision Points

### Decision Point 1: Which LLM Provider?

**Location:** `GraphBuilder.__init__()`

```python
graph = GraphBuilder(model_provider="groq")
# Could be "groq" or "openai"
```

**Impact:**
- Different models
- Different speeds
- Different costs

### Decision Point 2: Which Tools to Call?

**Location:** LLM reasoning in `agent_function()`

**LLM Decides Based On:**
- User query content
- Available tools
- Tool descriptions
- Previous tool results

**Example:**
- User mentions "weather" → Likely calls weather tools
- User mentions "budget" → Likely calls calculator/currency tools
- User mentions specific city → Calls place search tools for that city

### Decision Point 3: Call More Tools or Finish?

**Location:** LLM reasoning in `agent_function()` (2nd+ time)

**LLM Evaluates:**
- Do I have enough information?
- Is anything missing?
- Can I answer comprehensively?

**Options:**
- Call more tools (cycle back)
- Generate final response (go to END)

### Decision Point 4: Google or Tavily?

**Location:** Tool implementations (try-except blocks)

```python
try:
    result = google_api.search(query)
except:
    result = tavily_api.search(query)
```

**Factors:**
- Google API success/failure
- API quota remaining
- Network issues

---

## 5. Common Execution Patterns

### Pattern 1: Simple Weather Query

**Query:** "What's the weather in Paris?"

**Flow:**
```
1. Agent → Calls get_current_weather("Paris")
2. Tools → Returns weather data
3. Agent → Generates response with weather info
4. END
```

**Number of cycles:** 1 (agent → tools → agent → END)

### Pattern 2: Complex Travel Plan

**Query:** "Plan a 5-day trip to Bali with $3000 budget"

**Flow:**
```
1. Agent → Calls:
   - get_weather_forecast("Bali")
   - search_attractions("Bali")
   - search_restaurants("Bali")
   - search_activities("Bali")

2. Tools → Returns all data

3. Agent → Analyzes, calls:
   - convert_currency(3000, "USD", "IDR")
   - calculate_expense("expression")

4. Tools → Returns currency and calculation

5. Agent → Has everything, generates comprehensive plan

6. END
```

**Number of cycles:** 2 (agent → tools → agent → tools → agent → END)

### Pattern 3: Iterative Refinement

**Query:** "Plan budget-friendly trip to Tokyo"

**Flow:**
```
1. Agent → Calls search_attractions, search_restaurants

2. Tools → Returns data

3. Agent → Realizes needs budget info, calls:
   - search_transportation (for cost estimation)
   - calculate_expense (for budget calculation)

4. Tools → Returns more data

5. Agent → Generates budget-friendly recommendations

6. END
```

**Number of cycles:** 2+

---

## 6. Debugging Guide

### How to Debug Each Layer

#### 1. Streamlit Frontend

**Add print statements:**
```python
if submit_button and user_input.strip():
    print(f"User input: {user_input}")
    print(f"Payload: {payload}")
    
    response = requests.post(f"{BASE_URL}/query", json=payload)
    
    print(f"Response status: {response.status_code}")
    print(f"Response body: {response.text}")
```

#### 2. FastAPI Backend

**Add logging:**
```python
import logging

logging.basicConfig(level=logging.DEBUG)
logger = logging.getLogger(__name__)

@app.post("/query")
async def query_travel_agent(query: QueryRequest):
    logger.debug(f"Received query: {query.question}")
    
    graph = GraphBuilder(model_provider="groq")
    logger.debug("GraphBuilder initialized")
    
    output = react_app.invoke(messages)
    logger.debug(f"Output: {output}")
```

#### 3. Agent Workflow

**Add state logging:**
```python
def agent_function(self, state: MessagesState):
    print(f"Agent state: {len(state['messages'])} messages")
    print(f"Last message: {state['messages'][-1]}")
    
    response = self.llm_with_tools.invoke(input_question)
    
    print(f"LLM response: {response}")
    if hasattr(response, 'tool_calls'):
        print(f"Tool calls: {response.tool_calls}")
```

#### 4. Tool Execution

**Add tool logging:**
```python
@tool
def get_weather_forecast(city: str) -> str:
    print(f"get_weather_forecast called with city={city}")
    
    forecast_data = self.weather_service.get_forecast_weather(city)
    
    print(f"Forecast data received: {len(forecast_data.get('list', []))} items")
    
    return result
```

#### 5. External API Calls

**Add request logging:**
```python
def get_forecast_weather(self, city: str) -> dict:
    url = f"{self.base_url}/forecast?q={city}&appid={self.api_key}"
    
    print(f"Making request to: {url}")
    
    response = requests.get(url)
    
    print(f"Response status: {response.status_code}")
    print(f"Response size: {len(response.text)} bytes")
    
    return response.json()
```

### Common Issues and Solutions

#### Issue 1: LLM Not Calling Tools

**Symptom:** LLM generates response without using tools

**Debug:**
```python
# Check if tools are bound
print(f"Tools bound: {self.llm_with_tools._bound_tools}")

# Check system prompt
print(f"System prompt: {self.system_prompt.content}")
```

**Solution:**
- Ensure `bind_tools()` is called
- Check tool descriptions are clear
- Improve system prompt to emphasize tool usage

#### Issue 2: Tool Execution Fails

**Symptom:** Exception in tool execution

**Debug:**
```python
@tool
def search_attractions(place: str) -> str:
    try:
        result = self.google_places_search.google_search_attractions(place)
        return result
    except Exception as e:
        print(f"Error in search_attractions: {e}")
        import traceback
        traceback.print_exc()
        raise
```

**Solution:**
- Check API keys
- Verify API quotas
- Test external APIs independently

#### Issue 3: Infinite Loop

**Symptom:** Graph never reaches END

**Debug:**
```python
def agent_function(self, state: MessagesState):
    print(f"Agent iteration: {len([m for m in state['messages'] if isinstance(m, AIMessage)])}")
    
    if len(state['messages']) > 20:
        print("WARNING: Too many messages, possible infinite loop")
```

**Solution:**
- Add max iterations check
- Improve LLM decision-making
- Check tools_condition logic

---

## 🎯 Summary

**Complete Flow:**
1. User submits query in Streamlit
2. HTTP POST to FastAPI
3. GraphBuilder initializes (LLM + Tools)
4. Graph executes:
   - Agent decides → Tools execute → Agent processes → Repeat if needed
5. Final response extracted
6. Sent back to Streamlit
7. Displayed as formatted markdown

**Key Takeaways:**
- Execution is **cyclic** (can loop agent ↔ tools)
- LLM makes **decisions** at each step
- Tools provide **real-time data**
- State **accumulates** messages
- Graph **visualizable** for debugging

**For Interview:**
- Be able to trace a query end-to-end
- Explain decision points
- Know what each component does
- Understand state evolution

---

**Next:** Run the project yourself and add print statements to see this in action!
