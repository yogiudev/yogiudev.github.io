# LangGraph
# Complete Guide to Building Multi-Agent Workflows

## What is LangGraph?

LangGraph is a powerful library within the LangChain ecosystem that enables you to create stateful, multi-agent applications using large language models. Unlike traditional linear chains, LangGraph allows you to build **cyclical graphs** where agents can interact, make decisions, and coordinate their actions in sophisticated workflows.

Think of LangGraph as a framework for orchestrating multiple AI agents in a structured, graph-based architecture where each node represents a unit of work and edges define the flow of information and control.

## Core Concepts

### Graph Structure

At its heart, LangGraph uses a **directed graph** approach where:
- **Nodes** represent individual LLM agents or functions
- **Edges** serve as communication channels between agents
- **State** maintains context and information across interactions

### Key Components

**Nodes**: Units of work within your graph, typically Python functions that perform specific tasks such as:
- Interacting with an LLM
- Calling external tools or APIs
- Processing data
- Handling user input
- Executing business logic

**Edges**: Define the flow of information and execution order between nodes. LangGraph supports several types:
- **Normal Edges**: Direct, unconditional flow from one node to another
- **Conditional Edges**: Dynamic branching based on conditions
- **Entry Points**: Define which node executes first
- **Conditional Entry Points**: Use logic to determine the starting node

**State**: A central object that gets updated over time by nodes in the graph, maintaining:
- Conversation history
- Contextual data
- Internal variables and flags

## Getting Started

### Installation

```bash
pip install -U langgraph langchain-openai
```

### Basic Graph Setup

```python
from typing import Annotated
from typing_extensions import TypedDict
from langgraph.graph import StateGraph
from langgraph.graph.message import add_messages

# Define the state structure
class State(TypedDict):
    # Messages have the type "list"
    # The add_messages function appends messages to the list
    messages: Annotated[list, add_messages]

# Create the graph builder
graph_builder = StateGraph(State)
```

## Real-World Use Case: E-commerce Customer Service Agent

Let's build a comprehensive customer service system that handles multiple scenarios like order tracking, product recommendations, and billing inquiries. This system will showcase all LangGraph features in action.

### Use Case Overview

**Scenario**: TechMart, an online electronics retailer, needs an AI customer service system that can:
- Handle order inquiries and tracking
- Provide product recommendations
- Process returns and refunds
- Escalate complex issues to human agents
- Generate personalized follow-up emails

### Step 1: Define the State Schema

```python
from typing import TypedDict, Annotated, List, Optional, Dict
from langgraph.graph.message import add_messages
from datetime import datetime

class CustomerServiceState(TypedDict):
    # Core conversation
    messages: Annotated[List, add_messages]
    
    # Customer information
    customer_id: Optional[str]
    customer_name: Optional[str]
    customer_email: Optional[str]
    
    # Session tracking
    intent: Optional[str]
    session_id: str
    conversation_stage: str
    
    # Business data
    order_details: Optional[Dict]
    product_recommendations: List[Dict]
    support_ticket: Optional[Dict]
    
    # Control flow
    needs_human_escalation: bool
    retry_count: int
    satisfaction_score: Optional[int]
    
    # Performance tracking
    response_time: float
    tools_used: List[str]
```

### Step 2: Build the Agent Functions

```python
from langchain_openai import ChatOpenAI
from langgraph.graph import StateGraph, END, START
import json
import time
from datetime import datetime

# Initialize LLM
llm = ChatOpenAI(model="gpt-4", temperature=0.7)

def intent_classifier(state: CustomerServiceState):
    """First agent: Classify customer intent"""
    start_time = time.time()
    
    user_message = state["messages"][-1].content
    
    classification_prompt = f"""
    Analyze this customer message and classify the intent:
    
    Message: "{user_message}"
    
    Possible intents:
    - order_inquiry: Questions about existing orders
    - product_search: Looking for products or recommendations
    - billing_issue: Payment or billing problems
    - return_refund: Returns, exchanges, or refunds
    - technical_support: Product issues or technical help
    - general_inquiry: General questions or greetings
    
    Respond with just the intent name.
    """
    
    response = llm.invoke([{"role": "user", "content": classification_prompt}])
    intent = response.content.strip().lower()
    
    processing_time = time.time() - start_time
    
    return {
        "intent": intent,
        "conversation_stage": "intent_classified",
        "response_time": processing_time,
        "tools_used": state["tools_used"] + ["intent_classifier"]
    }

def order_tracking_agent(state: CustomerServiceState):
    """Handle order-related inquiries"""
    start_time = time.time()
    
    # Mock order lookup - in real app, this would query a database
    def lookup_order(customer_id: str, order_reference: str = None):
        # Simulated order data
        return {
            "order_id": "ORD-2024-001234",
            "status": "shipped",
            "tracking_number": "1Z999AA1234567890",
            "estimated_delivery": "2024-01-20",
            "items": [
                {"name": "iPhone 15 Pro", "quantity": 1, "price": 999.99},
                {"name": "MagSafe Charger", "quantity": 1, "price": 39.99}
            ],
            "total": 1039.98
        }
    
    # Extract order info from conversation
    user_message = state["messages"][-1].content
    order_details = lookup_order(state.get("customer_id", "CUST001"))
    
    response_prompt = f"""
    You're a helpful customer service agent. The customer asked: "{user_message}"
    
    Here's their order information:
    {json.dumps(order_details, indent=2)}
    
    Provide a helpful, friendly response about their order status.
    Include tracking information and estimated delivery date.
    """
    
    response = llm.invoke([{"role": "user", "content": response_prompt}])
    
    processing_time = time.time() - start_time
    
    return {
        "messages": [("assistant", response.content)],
        "order_details": order_details,
        "conversation_stage": "order_resolved",
        "response_time": processing_time,
        "tools_used": state["tools_used"] + ["order_tracking"]
    }

def product_recommendation_agent(state: CustomerServiceState):
    """Provide product recommendations"""
    start_time = time.time()
    
    # Mock product database
    products = [
        {"id": "P001", "name": "iPhone 15 Pro Max", "price": 1199.99, "category": "smartphones", "rating": 4.8},
        {"id": "P002", "name": "Samsung Galaxy S24", "price": 899.99, "category": "smartphones", "rating": 4.7},
        {"id": "P003", "name": "MacBook Pro 14", "price": 1999.99, "category": "laptops", "rating": 4.9},
        {"id": "P004", "name": "iPad Air", "price": 599.99, "category": "tablets", "rating": 4.6},
        {"id": "P005", "name": "AirPods Pro", "price": 249.99, "category": "audio", "rating": 4.5}
    ]
    
    user_message = state["messages"][-1].content
    
    # Simple recommendation logic (in real app, use ML/embeddings)
    recommendations = []
    if "phone" in user_message.lower() or "smartphone" in user_message.lower():
        recommendations = [p for p in products if p["category"] == "smartphones"]
    elif "laptop" in user_message.lower() or "computer" in user_message.lower():
        recommendations = [p for p in products if p["category"] == "laptops"]
    else:
        recommendations = products[:3]  # Top 3 products
    
    rec_text = "\n".join([
        f"• {p['name']} - ${p['price']} (Rating: {p['rating']}/5)"
        for p in recommendations
    ])
    
    response_prompt = f"""
    The customer asked: "{user_message}"
    
    Here are some great product recommendations:
    {rec_text}
    
    Provide a helpful response with these recommendations, highlighting key features and benefits.
    Ask if they'd like more details about any specific product.
    """
    
    response = llm.invoke([{"role": "user", "content": response_prompt}])
    
    processing_time = time.time() - start_time
    
    return {
        "messages": [("assistant", response.content)],
        "product_recommendations": recommendations,
        "conversation_stage": "recommendations_provided",
        "response_time": processing_time,
        "tools_used": state["tools_used"] + ["product_recommendations"]
    }

def billing_support_agent(state: CustomerServiceState):
    """Handle billing and payment issues"""
    start_time = time.time()
    
    # Mock billing data
    billing_info = {
        "account_balance": 0.00,
        "last_payment": "2024-01-15",
        "last_payment_amount": 1039.98,
        "payment_method": "Credit Card (**** 1234)",
        "billing_address": "123 Main St, Anytown, USA 12345"
    }
    
    user_message = state["messages"][-1].content
    
    # Check if this needs escalation
    escalation_keywords = ["fraud", "dispute", "unauthorized", "cancel account", "legal"]
    needs_escalation = any(keyword in user_message.lower() for keyword in escalation_keywords)
    
    if needs_escalation:
        response_content = """
        I understand this is a serious concern that requires immediate attention. 
        I'm escalating your case to our billing specialist who will contact you within 2 hours.
        
        Your case number is: BIL-2024-5678
        
        In the meantime, if this involves unauthorized charges, please contact your bank immediately.
        """
        
        support_ticket = {
            "ticket_id": "BIL-2024-5678",
            "type": "billing_escalation",
            "priority": "high",
            "created_at": datetime.now().isoformat(),
            "description": user_message
        }
        
        stage = "escalated_to_human"
        escalation_flag = True
    else:
        response_prompt = f"""
        You're a billing support agent. The customer asked: "{user_message}"
        
        Here's their billing information:
        {json.dumps(billing_info, indent=2)}
        
        Provide a helpful response about their billing inquiry.
        Be professional and address their concerns clearly.
        """
        
        response = llm.invoke([{"role": "user", "content": response_prompt}])
        response_content = response.content
        support_ticket = None
        stage = "billing_resolved"
        escalation_flag = False
    
    processing_time = time.time() - start_time
    
    return {
        "messages": [("assistant", response_content)],
        "support_ticket": support_ticket,
        "conversation_stage": stage,
        "needs_human_escalation": escalation_flag,
        "response_time": processing_time,
        "tools_used": state["tools_used"] + ["billing_support"]
    }

def satisfaction_survey_agent(state: CustomerServiceState):
    """Collect customer satisfaction feedback"""
    start_time = time.time()
    
    survey_message = """
    Thank you for contacting TechMart support! Before we end this conversation, 
    could you please rate your experience today on a scale of 1-5?
    
    1 = Very Dissatisfied
    2 = Dissatisfied  
    3 = Neutral
    4 = Satisfied
    5 = Very Satisfied
    
    Your feedback helps us improve our service!
    """
    
    processing_time = time.time() - start_time
    
    return {
        "messages": [("assistant", survey_message)],
        "conversation_stage": "satisfaction_survey",
        "response_time": processing_time,
        "tools_used": state["tools_used"] + ["satisfaction_survey"]
    }

def human_escalation_agent(state: CustomerServiceState):
    """Handle escalation to human agents"""
    start_time = time.time()
    
    escalation_message = f"""
    I've created a priority support ticket for you: {state['support_ticket']['ticket_id']}
    
    A human specialist will review your case and contact you within 2 hours at {state.get('customer_email', 'your registered email')}.
    
    Is there anything else I can help you with while you wait?
    """
    
    processing_time = time.time() - start_time
    
    return {
        "messages": [("assistant", escalation_message)],
        "conversation_stage": "escalated_complete",
        "response_time": processing_time,
        "tools_used": state["tools_used"] + ["human_escalation"]
    }
```

### Step 3: Define Routing Logic

```python
def route_by_intent(state: CustomerServiceState):
    """Route customer to appropriate agent based on intent"""
    intent = state.get("intent", "general_inquiry")
    
    routing_map = {
        "order_inquiry": "order_tracking",
        "product_search": "product_recommendations", 
        "billing_issue": "billing_support",
        "return_refund": "billing_support",  # Use billing agent for refunds
        "technical_support": "human_escalation",  # Tech issues go to humans
        "general_inquiry": "product_recommendations"  # Default to recommendations
    }
    
    return routing_map.get(intent, "product_recommendations")

def check_escalation_needed(state: CustomerServiceState):
    """Check if conversation needs human escalation"""
    if state.get("needs_human_escalation", False):
        return "human_escalation"
    elif state.get("conversation_stage") in ["order_resolved", "recommendations_provided", "billing_resolved"]:
        return "satisfaction_survey"
    else:
        return "continue_conversation"

def handle_satisfaction_response(state: CustomerServiceState):
    """Process satisfaction survey response"""
    user_message = state["messages"][-1].content
    
    # Extract rating (simple approach)
    rating = None
    for char in user_message:
        if char.isdigit():
            rating = int(char)
            break
    
    if rating:
        if rating >= 4:
            return "end_positive"
        elif rating <= 2:
            return "escalate_feedback"
        else:
            return "end_neutral"
    else:
        return "end_neutral"
```

### Step 4: Build the Complete Graph

```python
# Initialize the graph
workflow = StateGraph(CustomerServiceState)

# Add all agent nodes
workflow.add_node("intent_classifier", intent_classifier)
workflow.add_node("order_tracking", order_tracking_agent)
workflow.add_node("product_recommendations", product_recommendation_agent)
workflow.add_node("billing_support", billing_support_agent)
workflow.add_node("satisfaction_survey", satisfaction_survey_agent)
workflow.add_node("human_escalation", human_escalation_agent)

# Set entry point
workflow.set_entry_point("intent_classifier")

# Add routing from intent classifier
workflow.add_conditional_edges(
    "intent_classifier",
    route_by_intent,
    {
        "order_tracking": "order_tracking",
        "product_recommendations": "product_recommendations",
        "billing_support": "billing_support",
        "human_escalation": "human_escalation"
    }
)

# Add escalation checks for each agent
workflow.add_conditional_edges(
    "order_tracking",
    check_escalation_needed,
    {
        "satisfaction_survey": "satisfaction_survey",
        "human_escalation": "human_escalation"
    }
)

workflow.add_conditional_edges(
    "product_recommendations", 
    check_escalation_needed,
    {
        "satisfaction_survey": "satisfaction_survey",
        "human_escalation": "human_escalation"
    }
)

workflow.add_conditional_edges(
    "billing_support",
    check_escalation_needed,
    {
        "satisfaction_survey": "satisfaction_survey", 
        "human_escalation": "human_escalation"
    }
)

# Handle satisfaction survey responses
workflow.add_conditional_edges(
    "satisfaction_survey",
    handle_satisfaction_response,
    {
        "end_positive": END,
        "end_neutral": END,
        "escalate_feedback": "human_escalation"
    }
)

# Human escalation goes to end
workflow.add_edge("human_escalation", END)

# Compile the graph
app = workflow.compile()
```

### Step 5: Test the System with Real Examples

```python
# Test Case 1: Order Inquiry
def test_order_inquiry():
    """Test order tracking functionality"""
    initial_state = {
        "messages": [("user", "Hi, I'd like to check the status of my recent order")],
        "customer_id": "CUST001",
        "customer_name": "John Doe",
        "customer_email": "john@example.com",
        "session_id": "session_001",
        "conversation_stage": "initial",
        "needs_human_escalation": False,
        "retry_count": 0,
        "response_time": 0.0,
        "tools_used": []
    }
    
    result = app.invoke(initial_state)
    return result

# Test Case 2: Product Search
def test_product_search():
    """Test product recommendation functionality"""
    initial_state = {
        "messages": [("user", "I'm looking for a new smartphone under $1000")],
        "customer_id": "CUST002", 
        "customer_name": "Jane Smith",
        "customer_email": "jane@example.com",
        "session_id": "session_002",
        "conversation_stage": "initial",
        "needs_human_escalation": False,
        "retry_count": 0,
        "response_time": 0.0,
        "tools_used": []
    }
    
    result = app.invoke(initial_state)
    return result

# Test Case 3: Billing Issue (Escalation)
def test_billing_escalation():
    """Test billing issue that requires escalation"""
    initial_state = {
        "messages": [("user", "I see an unauthorized charge on my account for $500. This looks like fraud!")],
        "customer_id": "CUST003",
        "customer_name": "Bob Johnson", 
        "customer_email": "bob@example.com",
        "session_id": "session_003",
        "conversation_stage": "initial",
        "needs_human_escalation": False,
        "retry_count": 0,
        "response_time": 0.0,
        "tools_used": []
    }
    
    result = app.invoke(initial_state)
    return result

# Run tests and show results
print("=== Test 1: Order Inquiry ===")
order_result = test_order_inquiry()
print(f"Final stage: {order_result['conversation_stage']}")
print(f"Tools used: {order_result['tools_used']}")
print(f"Response: {order_result['messages'][-1].content}")

print("\n=== Test 2: Product Search ===")
product_result = test_product_search()
print(f"Final stage: {product_result['conversation_stage']}")
print(f"Recommendations: {len(product_result.get('product_recommendations', []))} products")
print(f"Response: {product_result['messages'][-1].content}")

print("\n=== Test 3: Billing Escalation ===")
billing_result = test_billing_escalation()
print(f"Final stage: {billing_result['conversation_stage']}")
print(f"Escalated: {billing_result['needs_human_escalation']}")
print(f"Ticket ID: {billing_result.get('support_ticket', {}).get('ticket_id')}")
```

### Expected Outputs

**Test 1 - Order Inquiry Expected Output:**
```
Final stage: satisfaction_survey
Tools used: ['intent_classifier', 'order_tracking', 'satisfaction_survey']
Response: Thank you for contacting TechMart support! Before we end this conversation, 
could you please rate your experience today on a scale of 1-5?

1 = Very Dissatisfied
2 = Dissatisfied  
3 = Neutral
4 = Satisfied
5 = Very Satisfied

Your feedback helps us improve our service!
```

**Test 2 - Product Search Expected Output:**
```
Final stage: satisfaction_survey
Recommendations: 2 products
Response: Thank you for contacting TechMart support! Before we end this conversation, 
could you please rate your experience today on a scale of 1-5?

Your feedback helps us improve our service!
```

**Test 3 - Billing Escalation Expected Output:**
```
Final stage: escalated_complete
Escalated: True
Ticket ID: BIL-2024-5678
Response: I've created a priority support ticket for you: BIL-2024-5678

A human specialist will review your case and contact you within 2 hours at bob@example.com.

Is there anything else I can help you with while you wait?
```

## Pro Tips and Tricks

### 1. **State Management Best Practices**

```python
# ✅ Good: Use typed state with defaults
class robustState(TypedDict):
    messages: Annotated[List, add_messages]
    retry_count: int = 0  # Default value
    error_log: List[str] = []
    
# ✅ Good: Validate state in nodes
def safe_node(state: CustomerServiceState):
    if not state.get("messages"):
        return {"messages": [("system", "No messages found")]}
    
    # Continue with processing...
```

### 2. **Performance Optimization Tricks**

```python
# ✅ Use caching for expensive operations
from functools import lru_cache

@lru_cache(maxsize=100)
def expensive_product_lookup(query: str):
    # Cached database lookup
    pass

# ✅ Implement timeouts
import asyncio
from asyncio import timeout

async def llm_with_timeout(prompt: str, timeout_seconds: int = 30):
    try:
        async with timeout(timeout_seconds):
            return await llm.ainvoke(prompt)
    except asyncio.TimeoutError:
        return "Request timed out"
```

### 3. **Error Handling Patterns**

```python
def resilient_agent(state: CustomerServiceState):
    """Agent with comprehensive error handling"""
    try:
        # Main logic
        result = process_request(state)
        return {"result": result, "error": None}
    
    except Exception as e:
        retry_count = state.get("retry_count", 0)
        
        if retry_count < 3:
            # Retry with exponential backoff
            time.sleep(2 ** retry_count)
            return {
                "retry_count": retry_count + 1,
                "error": f"Retry {retry_count + 1}: {str(e)}"
            }
        else:
            # Escalate after max retries
            return {
                "needs_human_escalation": True,
                "error": f"Max retries exceeded: {str(e)}"
            }
```

### 4. **Advanced Routing Strategies**

```python
# ✅ Multi-condition routing
def smart_router(state: CustomerServiceState):
    """Route based on multiple factors"""
    user_message = state["messages"][-1].content.lower()
    customer_tier = state.get("customer_tier", "standard")
    
    # VIP customers get priority routing
    if customer_tier == "vip":
        return "vip_agent"
    
    # Sentiment-based routing
    if any(word in user_message for word in ["angry", "frustrated", "disappointed"]):
        return "escalation_specialist"
    
    # Default routing
    return route_by_intent(state)

# ✅ Load balancing between agents
def load_balanced_routing(state: CustomerServiceState):
    """Distribute load across multiple agents"""
    agent_loads = {
        "agent_1": get_current_load("agent_1"),
        "agent_2": get_current_load("agent_2"),
        "agent_3": get_current_load("agent_3")
    }
    
    # Route to least loaded agent
    return min(agent_loads.items(), key=lambda x: x[1])[0]
```

### 5. **Testing and Debugging Strategies**

```python
# ✅ Unit test individual nodes
def test_intent_classifier():
    test_state = {
        "messages": [("user", "Where is my order?")],
        "tools_used": []
    }
    
    result = intent_classifier(test_state)
    assert result["intent"] == "order_inquiry"
    assert "intent_classifier" in result["tools_used"]

# ✅ Integration test full workflows
def test_complete_workflow():
    """Test end-to-end workflow"""
    test_cases = [
        ("Where is my order?", "order_inquiry"),
        ("I need a new phone", "product_search"),
        ("Billing issue", "billing_issue")
    ]
    
    for message, expected_intent in test_cases:
        result = app.invoke({
            "messages": [("user", message)],
            "session_id": f"test_{expected_intent}",
            "tools_used": []
        })
        
        assert expected_intent in result["tools_used"]

# ✅ Performance monitoring
def monitor_performance(state: CustomerServiceState):
    """Track performance metrics"""
    metrics = {
        "total_response_time": state.get("response_time", 0),
        "tools_used_count": len(state.get("tools_used", [])),
        "escalation_rate": 1 if state.get("needs_human_escalation") else 0,
        "conversation_length": len(state.get("messages", []))
    }
    
    # Log metrics to monitoring system
    log_metrics(metrics)
    
    return state
```

### 6. **Memory and Persistence Tricks**

```python
# ✅ Use checkpointing for conversation persistence
from langgraph.checkpoint.memory import MemorySaver

memory = MemorySaver()
persistent_app = workflow.compile(checkpointer=memory)

# ✅ Conversation continuity
def continue_conversation(session_id: str, new_message: str):
    """Continue existing conversation"""
    config = {"configurable": {"thread_id": session_id}}
    
    result = persistent_app.invoke(
        {"messages": [("user", new_message)]},
        config=config
    )
    
    return result
```

### 7. **Production Deployment Best Practices**

```python
# ✅ Environment-specific configurations
import os
from enum import Enum

class Environment(Enum):
    DEV = "development"
    STAGING = "staging"
    PROD = "production"

def get_llm_config(env: Environment):
    """Get environment-specific LLM configuration"""
    if env == Environment.PROD:
        return {
            "model": "gpt-4",
            "temperature": 0.3,
            "max_tokens": 1000,
            "timeout": 30
        }
    else:
        return {
            "model": "gpt-3.5-turbo",
            "temperature": 0.7,
            "max_tokens": 500,
            "timeout": 15
        }

# ✅ Health checks and monitoring
def health_check():
    """System health check"""
    try:
        # Test basic functionality
        test_result = app.invoke({
            "messages": [("user", "test")],
            "session_id": "health_check",
            "tools_used": []
        })
        
        return {
            "status": "healthy",
            "response_time": test_result.get("response_time", 0),
            "timestamp": datetime.now().isoformat()
        }
    except Exception as e:
        return {
            "status": "unhealthy",
            "error": str(e),
            "timestamp": datetime.now().isoformat()
        }
```

## Advanced Features in Action

### Streaming Results for Real-time Updates

```python
# ✅ Stream results for better user experience
async def stream_customer_service(user_message: str, session_id: str):
    """Stream responses in real-time"""
    config = {"configurable": {"thread_id": session_id}}
    
    async for chunk in app.astream(
        {"messages": [("user", user_message)]},
        config=config
    ):
        # Process each chunk as it arrives
        yield chunk

# Usage example:
async def handle_streaming_request():
    async for chunk in stream_customer_service("Where is my order?", "user123"):
        print(f"Received: {chunk}")
```

### Dynamic Graph Modification

```python
# ✅ Modify graph behavior at runtime
def add_seasonal_agent(workflow: StateGraph):
    """Add seasonal promotions agent during holidays"""
    def seasonal_promotions_agent(state: CustomerServiceState):
        # Holiday-specific promotions
        return {
            "messages": [("assistant", "Check out our holiday deals!")],
            "tools_used": state["tools_used"] + ["seasonal_promotions"]
        }
    
    workflow.add_node("seasonal_promotions", seasonal_promotions_agent)
    
    # Modify routing to include seasonal agent
    def holiday_router(state: CustomerServiceState):
        if is_holiday_season():
            return "seasonal_promotions"
        else:
            return route_by_intent(state)
    
    return workflow
```

