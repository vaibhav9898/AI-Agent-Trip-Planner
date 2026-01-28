# 🌍 AI Travel Planner - Agentic Workflow System

[![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-green.svg)](https://fastapi.tiangolo.com/)
[![Streamlit](https://img.shields.io/badge/Streamlit-1.30+-red.svg)](https://streamlit.io/)

An intelligent AI-powered travel planning application that leverages multi-agent workflows, real-time data integration, and advanced language models to create comprehensive travel itineraries with detailed cost breakdowns.

## 📚 Interview Preparation & Learning Resources

**New to this project or preparing for an interview?** Start here:

### 🎯 Quick Start
- **[QUICK_REFERENCE.md](QUICK_REFERENCE.md)** - Quick reference and overview (Start here!)
  - Documentation map
  - Key facts and concepts
  - Study checklist
  - Interview demo script

### 📖 Comprehensive Guides

1. **[INTERVIEW_PREP_GUIDE.md](INTERVIEW_PREP_GUIDE.md)** - Complete interview preparation guide
   - Prerequisites and topics to study
   - Recommended learning order
   - Common interview questions
   - Study timeline (4-week plan)

2. **[CONCEPTS_EXPLAINED.md](CONCEPTS_EXPLAINED.md)** - Deep dive into AI/ML concepts
   - Large Language Models (LLMs)
   - Agentic Workflows
   - Function Calling / Tool Use
   - LangChain & LangGraph frameworks

3. **[PROJECT_ARCHITECTURE.md](PROJECT_ARCHITECTURE.md)** - System design and architecture
   - High-level architecture overview
   - Component breakdown
   - Data flow diagrams
   - Design patterns used

4. **[STEP_BY_STEP_GUIDE.md](STEP_BY_STEP_GUIDE.md)** - Code execution walkthrough
   - Complete request flow trace
   - Line-by-line code explanation
   - Debugging guide
   - Common execution patterns

**Quick Start for Learners:** If you have basic AI/ML knowledge, start with the Interview Prep Guide, then read the other documents in order.

## 🚀 Features

### Core Capabilities
- **🤖 Multi-Agent Architecture**: Built with LangGraph for sophisticated agentic workflows
- **🌦️ Real-Time Weather Integration**: Live weather forecasts and current conditions
- **🗺️ Smart Place Discovery**: Google Places API + Tavily search for attractions, restaurants, and activities
- **💰 Dynamic Cost Calculations**: Automated expense planning and budget breakdowns
- **💱 Currency Conversion**: Real-time exchange rates for international travel
- **📱 Dual Interface**: Both REST API and Streamlit web interface

### AI-Powered Features
- **Comprehensive Itineraries**: Day-by-day travel plans with detailed recommendations
- **Off-Beat Locations**: Alternative suggestions beyond typical tourist spots
- **Smart Fallbacks**: Multiple data sources ensure reliable information retrieval
- **Personalized Budgeting**: Tailored cost estimates based on travel preferences

## 🏗️ Architecture

![Alt Text](arch.png)

System Components
GraphBuilder: Orchestrates the multi-agent workflow using LangGraph

Tool Suite: Modular tools for weather, places, calculations, and currency

Model Flexibility: Support for multiple LLM providers (Groq, OpenAI)

Robust Error Handling: Graceful fallbacks and comprehensive error management

🛠️ Installation
Prerequisites
Python 3.10+

API Keys for external services (see Environment Setup)

Quick Start with UV:->
# Install UV package manager
pip install uv

# Clone repository
git clone https://github.com/yourusername/ai-travel-planner.git
cd ai-travel-planner

# Create virtual environment
uv venv env --python cpython-3.10.18

# Activate environment (Windows)
env\Scripts\activate.bat
# Or Linux/Mac
source env/bin/activate

# Install dependencies
uv add pandas fastapi uvicorn streamlit langchain-groq langgraph python-dotenv requests

# Traditional Installation

git clone https://github.com/yourusername/ai-travel-planner.git
cd ai-travel-planner
pip install -r requirements.txt


Required API Keys
Groq/OpenAI: For LLM inference

OpenWeatherMap: Weather data

Google Places: Location and business data

ExchangeRate-API: Currency conversion

Tavily: Fallback search functionality
