# 🎨 Visual Workflow Diagrams

## 1. System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                         USER LAYER                          │
│  ┌──────────────────┐              ┌──────────────────┐    │
│  │  Streamlit UI    │              │   REST Clients   │    │
│  │  (Port 8501)     │              │   (API Calls)    │    │
│  └────────┬─────────┘              └────────┬─────────┘    │
└───────────┼─────────────────────────────────┼──────────────┘
            │                                 │
            └─────────────┬───────────────────┘
                          │ HTTP POST
┌─────────────────────────▼─────────────────────────────────┐
│                    APPLICATION LAYER                       │
│  ┌─────────────────────────────────────────────────────┐  │
│  │          FastAPI Backend (Port 8000)                │  │
│  │  POST /query → async def query_travel_agent()       │  │
│  └──────────────────────┬──────────────────────────────┘  │
└─────────────────────────┼────────────────────────────────┘
                          │
┌─────────────────────────▼─────────────────────────────────┐
│                   ORCHESTRATION LAYER                      │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              GraphBuilder                           │  │
│  │  ┌──────────────┐  ┌──────────────┐                │  │
│  │  │ ModelLoader  │  │ System Prompt│                │  │
│  │  │ (Groq/OpenAI)│  │              │                │  │
│  │  └──────────────┘  └──────────────┘                │  │
│  │                                                     │  │
│  │          LangGraph State Machine                    │  │
│  │  ┌─────────┐      ┌──────────┐                     │  │
│  │  │ Agent   │◄────►│  Tools   │                     │  │
│  │  │  Node   │      │   Node   │                     │  │
│  │  └─────────┘      └──────────┘                     │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────┬────────────────────────────────┘
                          │
┌─────────────────────────▼─────────────────────────────────┐
│                      TOOL LAYER                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │   Weather    │  │    Place     │  │  Calculator  │    │
│  │    Tools     │  │  Search      │  │    Tools     │    │
│  │  - Current   │  │  - Attract.  │  │  - Hotel     │    │
│  │  - Forecast  │  │  - Restaur.  │  │  - Total     │    │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘    │
│  ┌──────────────┐         │                               │
│  │  Currency    │         │                               │
│  │ Conversion   │         │                               │
│  │    Tool      │         │                               │
│  └──────┬───────┘         │                               │
└─────────┼─────────────────┼───────────────────────────────┘
          │                 │
┌─────────▼─────────────────▼───────────────────────────────┐
│                    EXTERNAL APIs                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │OpenWeatherMap│  │Google Places │  │ExchangeRate  │    │
│  │     API      │  │     API      │  │     API      │    │
│  └──────────────┘  └──────┬───────┘  └──────────────┘    │
│                           │                               │
│                    ┌──────▼───────┐                       │
│                    │  Tavily API  │                       │
│                    │  (Fallback)  │                       │
│                    └──────────────┘                       │
└───────────────────────────────────────────────────────────┘
```

---

## 2. LangGraph Workflow Diagram

```
┌──────────┐
│  START   │
└────┬─────┘
     │
     ▼
┌────────────────────────────────────────┐
│         AGENT NODE                     │
│  ┌──────────────────────────────────┐ │
│  │  System Prompt + User Message    │ │
│  └──────────────┬───────────────────┘ │
│                 ▼                      │
│  ┌──────────────────────────────────┐ │
│  │  LLM (Groq/OpenAI) with Tools    │ │
│  └──────────────┬───────────────────┘ │
│                 ▼                      │
│  ┌──────────────────────────────────┐ │
│  │  Decision: Call Tool or Respond? │ │
│  └──────────────┬───────────────────┘ │
└─────────────────┼────────────────────┘
                  │
       ┌──────────┴──────────┐
       │                     │
       ▼                     ▼
  [Tool Call?]          [No Tool Call]
       │                     │
       ▼                     │
┌──────────────────┐         │
│   TOOLS NODE     │         │
│  ┌────────────┐  │         │
│  │ Execute    │  │         │
│  │ Selected   │  │         │
│  │ Tool       │  │         │
│  └─────┬──────┘  │         │
│        ▼         │         │
│  ┌────────────┐  │         │
│  │ Return     │  │         │
│  │ Result     │  │         │
│  └─────┬──────┘  │         │
└────────┼─────────┘         │
         │                   │
         └─────┬─────────────┘
               │
               ▼
        [More info needed?]
               │
       ┌───────┴────────┐
       │                │
      YES              NO
       │                │
       │                ▼
       │         ┌──────────┐
       │         │   END    │
       │         │  Return  │
       │         │ Response │
       │         └──────────┘
       │
       └─► Back to AGENT NODE
```

---

## 3. Request Processing Timeline

```
Time    User              FastAPI          GraphBuilder      Agent         Tools           External APIs
│       │                 │                │                 │             │               │
├─ 0s   │ Enter query     │                │                 │             │               │
│       │ "Plan trip..."  │                │                 │             │               │
│       └────────────────►│                │                 │             │               │
│                         │                │                 │             │               │
├─ 0.1s                   │ Initialize     │                 │             │               │
│                         │ GraphBuilder   │                 │             │               │
│                         └───────────────►│                 │             │               │
│                                          │                 │             │               │
├─ 0.2s                                    │ Load LLM        │             │               │
│                                          │ Bind Tools      │             │               │
│                                          │ Build Graph     │             │               │
│                                          └────────────────►│             │               │
│                                                            │             │               │
├─ 1s                                                        │ Analyze     │               │
│                                                            │ Query       │               │
│                                                            │ Decide:     │               │
│                                                            │ "Need       │               │
│                                                            │ weather"    │               │
│                                                            └────────────►│               │
│                                                                          │               │
├─ 2s                                                                      │ Call          │
│                                                                          │ get_current_  │
│                                                                          │ weather()     │
│                                                                          └──────────────►│
│                                                                                          │
├─ 3s                                                                                      │ OpenWeatherMap
│                                                                                          │ API Call
│                                                                          ◄───────────────┤
├─ 4s                                                        ◄─────────────┤ Return        │
│                                                            │             │ "15°C..."     │
│                                                            │             │               │
├─ 5s                                                        │ Decide:     │               │
│                                                            │ "Need       │               │
│                                                            │ attractions"│               │
│                                                            └────────────►│               │
│                                                                          │               │
├─ 6s                                                                      │ Call          │
│                                                                          │ search_       │
│                                                                          │ attractions() │
│                                                                          └──────────────►│
│                                                                                          │
├─ 8s                                                                                      │ Google Places
│                                                                                          │ API Call
│                                                                          ◄───────────────┤
├─ 9s                                                        ◄─────────────┤ Return        │
│                                                            │             │ [Places...]   │
│                                                            │             │               │
│                                                            │ ... (continue for other tools)
│                                                            │             │               │
├─ 12s                                                       │ All info    │               │
│                                                            │ gathered    │               │
│                                                            │ Generate    │               │
│                                                            │ response    │               │
│                                                            │             │               │
│                         ◄──────────────────────────────────┤             │               │
│                         │ Return final                     │             │               │
│                         │ travel plan                      │             │               │
│       ◄─────────────────┤                                  │             │               │
│       │ Display         │                                  │             │               │
├─ 13s │ Markdown        │                                  │             │               │
│       │ itinerary       │                                  │             │               │
```

---

## 4. Tool Selection Decision Tree

```
                         User Query
                              │
                              ▼
                  ┌───────────────────────┐
                  │  Agent Analyzes Query │
                  └───────────┬───────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
   [Location?]           [Budget?]            [Weather?]
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐     ┌───────────────┐    ┌───────────────┐
│ Place Search  │     │  Calculator   │    │    Weather    │
│ Tools         │     │  Tools        │    │    Tools      │
├───────────────┤     ├───────────────┤    ├───────────────┤
│• Attractions  │     │• Hotel Cost   │    │• Current      │
│• Restaurants  │     │• Total Exp.   │    │• Forecast     │
│• Activities   │     │• Daily Budget │    └───────────────┘
│• Transport    │     └───────────────┘
└───────────────┘             │
        │                     │
        ▼                     ▼
   [Foreign?]            [Sum costs]
        │                     │
       YES                    │
        │                     │
        ▼                     │
┌───────────────┐             │
│  Currency     │             │
│  Converter    │             │
│  Tool         │             │
└───────────────┘             │
        │                     │
        └─────────┬───────────┘
                  │
                  ▼
           [Generate Plan]
                  │
                  ▼
         Return Comprehensive
              Itinerary
```

---

## 5. Message State Evolution Flow

```
Step 1: Initial State
┌─────────────────────────────────────────┐
│ MessagesState                           │
│ {                                       │
│   "messages": [                         │
│     "Plan a trip to Tokyo for 3 days"   │
│   ]                                     │
│ }                                       │
└─────────────────────────────────────────┘
                   │
                   ▼
Step 2: After Agent Decides to Call Tool
┌─────────────────────────────────────────┐
│ MessagesState                           │
│ {                                       │
│   "messages": [                         │
│     "Plan a trip to Tokyo...",          │
│     AIMessage(                          │
│       content="",                       │
│       tool_calls=[{                     │
│         name: "get_current_weather",    │
│         args: {"city": "Tokyo"}         │
│       }]                                │
│     )                                   │
│   ]                                     │
│ }                                       │
└─────────────────────────────────────────┘
                   │
                   ▼
Step 3: After Tool Executes
┌─────────────────────────────────────────┐
│ MessagesState                           │
│ {                                       │
│   "messages": [                         │
│     "Plan a trip to Tokyo...",          │
│     AIMessage(tool_calls=[...]),        │
│     ToolMessage(                        │
│       content="Tokyo: 18°C, clear sky"  │
│     )                                   │
│   ]                                     │
│ }                                       │
└─────────────────────────────────────────┘
                   │
                   ▼
           ... (repeats for each tool call)
                   │
                   ▼
Step N: Final State
┌─────────────────────────────────────────┐
│ MessagesState                           │
│ {                                       │
│   "messages": [                         │
│     "Plan a trip to Tokyo...",          │
│     AIMessage(tool_calls=[...]),        │
│     ToolMessage(...),                   │
│     AIMessage(tool_calls=[...]),        │
│     ToolMessage(...),                   │
│     ...                                 │
│     AIMessage(                          │
│       content="# Tokyo Travel Plan\n... │
│     )                                   │
│   ]                                     │
│ }                                       │
└─────────────────────────────────────────┘
```

---

## 6. Fallback Mechanism Flow

```
User requests attractions
         │
         ▼
┌──────────────────────┐
│  Try: Google Places  │
│  API                 │
└──────┬───────────────┘
       │
       ├───► [Success?] ──YES──► Return Google results
       │                              └──► Continue
       │
       └───► [Error/Timeout]
                │
                ▼
         ┌──────────────────┐
         │  Log: "Google    │
         │  Places failed"  │
         └──────┬───────────┘
                │
                ▼
         ┌──────────────────┐
         │  Try: Tavily     │
         │  Search API      │
         └──────┬───────────┘
                │
                ├───► [Success?] ──YES──► Return Tavily results
                │                              └──► Continue
                │
                └───► [Error]
                       │
                       ▼
                ┌──────────────────┐
                │  Return:         │
                │  "Could not      │
                │  fetch data"     │
                └──────────────────┘
```

---

## 7. Component Interaction Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    GraphBuilder                         │
│                                                          │
│  __init__():                                            │
│  ┌────────────────────────────────────────────┐        │
│  │ 1. ModelLoader → load_llm()                │        │
│  │    └─► Groq or OpenAI LLM instance         │        │
│  │                                             │        │
│  │ 2. Initialize Tool Classes:                │        │
│  │    ├─► WeatherInfoTool()                   │        │
│  │    ├─► PlaceSearchTool()                   │        │
│  │    ├─► CalculatorTool()                    │        │
│  │    └─► CurrencyConverterTool()             │        │
│  │                                             │        │
│  │ 3. Aggregate all tools into list           │        │
│  │                                             │        │
│  │ 4. llm.bind_tools(tools)                   │        │
│  │    └─► LLM + Tools                         │        │
│  │                                             │        │
│  │ 5. Load SYSTEM_PROMPT                      │        │
│  └────────────────────────────────────────────┘        │
│                                                          │
│  build_graph():                                         │
│  ┌────────────────────────────────────────────┐        │
│  │ 1. StateGraph(MessagesState)               │        │
│  │                                             │        │
│  │ 2. Add Nodes:                              │        │
│  │    ├─► "agent" → agent_function()          │        │
│  │    └─► "tools" → ToolNode(tools)           │        │
│  │                                             │        │
│  │ 3. Add Edges:                              │        │
│  │    ├─► START → "agent"                     │        │
│  │    ├─► "agent" → tools_condition()         │        │
│  │    │    ├─► If tool call → "tools"         │        │
│  │    │    └─► If no tool → END               │        │
│  │    └─► "tools" → "agent" (loop back)       │        │
│  │                                             │        │
│  │ 4. compile()                               │        │
│  └────────────────────────────────────────────┘        │
│                                                          │
│  agent_function(state):                                 │
│  ┌────────────────────────────────────────────┐        │
│  │ 1. Get user messages from state            │        │
│  │ 2. Prepend SYSTEM_PROMPT                   │        │
│  │ 3. llm_with_tools.invoke(messages)         │        │
│  │ 4. Return {"messages": [response]}         │        │
│  └────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────┘
```

---

## 8. API Response Format Example

```
POST /query
{
  "question": "Plan a trip to Paris for 5 days"
}

                    ↓

Processing (5-15 seconds):
- GraphBuilder initialization
- Graph construction
- Agent loop (multiple tool calls)
- Response generation

                    ↓

Response 200 OK
{
  "answer": "
# 🌍 Paris Travel Plan

## Overview
Your 5-day adventure in the City of Lights!

## Weather Forecast
📅 Current: 15°C, partly cloudy
📅 Week forecast: 12-18°C, mix of sun and clouds

## Day-by-Day Itinerary

### Day 1: Arrival & Eiffel Tower
- **Morning**: Arrive, check into Hotel Le Marais ($120/night)
- **Afternoon**: Eiffel Tower visit ($25 entry)
- **Dinner**: Le Jules Verne restaurant ($80 per person)

### Day 2: Museums & Culture
...

## Estimated Costs
- Hotels (5 nights): $600
- Attractions: $150
- Meals: $400
- Transportation: $100
- **Total: $1,250**
- **Daily Budget: $250**

## Off-Beat Alternatives
...
"
}
```

---

## Quick Reference Symbols

- `►` Flow direction
- `└─►` Sub-flow or result
- `├─►` Branch flow
- `◄─` Return flow
- `▼` Continues down
- `│` Connection line
- `┌─┐` Box corners
- `[Decision]` Decision point
- `{State}` State object

