# LangGraph: Advanced
# AI Analytics Agent for MongoDB

## Real-World Use Case: MongoDB Intelligence Platform

Let's build a comprehensive AI analytics agent that transforms natural language queries into actionable insights from MongoDB. This system will showcase advanced LangGraph capabilities while providing analytics, predictions, and visualizations.

### Use Case Overview

**Scenario**: DataFlow Inc. needs an AI-powered analytics platform that can:
- Convert natural language to MongoDB queries
- Generate insights and predictions
- Create dynamic dashboards and visualizations
- Perform external API enrichment
- Optimize for token usage and performance

## System Architecture

### Step 1: Define the Enhanced State Schema

```python
from typing import TypedDict, Annotated, List, Optional, Dict, Any
from langgraph.graph.message import add_messages
from datetime import datetime
import json

class AnalyticsState(TypedDict):
    # Core conversation
    messages: Annotated[List, add_messages]
    
    # Query processing
    original_query: str
    query_intent: str
    query_complexity: str  # simple, medium, complex
    
    # MongoDB operations
    generated_queries: List[Dict[str, Any]]
    query_results: List[Dict[str, Any]]
    collections_accessed: List[str]
    
    # Analytics components
    insights_generated: List[str]
    predictions: Dict[str, Any]
    visualizations: List[Dict[str, Any]]
    
    # External data enrichment
    external_apis_called: List[str]
    enriched_data: Dict[str, Any]
    
    # Metadata and optimization
    schema_info: Dict[str, Any]
    feature_metadata: Dict[str, Any]
    embeddings_cache: Dict[str, Any]
    
    # Performance tracking
    token_usage: Dict[str, int]
    query_execution_time: float
    cache_hits: int
    
    # Control flow
    needs_visualization: bool
    needs_external_data: bool
    needs_prediction: bool
    confidence_score: float
```

### Step 2: Initialize Components and Metadata

```python
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langgraph.graph import StateGraph, END, START
import pymongo
import numpy as np
from sklearn.ensemble import RandomForestRegressor
import plotly.graph_objects as go
import requests
import time

# Initialize components
llm = ChatOpenAI(model="gpt-4", temperature=0.3)
embeddings = OpenAIEmbeddings()
mongo_client = pymongo.MongoClient("mongodb://localhost:27017/")
db = mongo_client.analytics_db

# Sample schema and metadata
SCHEMA_INFO = {
    "users": {
        "fields": ["user_id", "name", "email", "signup_date", "plan_type", "usage_stats"],
        "indexes": ["user_id", "email", "signup_date"],
        "sample_values": {
            "plan_type": ["basic", "premium", "enterprise"],
            "email": ["gmail.com", "outlook.com", "company.com"]
        }
    },
    "orders": {
        "fields": ["order_id", "user_id", "amount", "status", "created_at", "items"],
        "indexes": ["order_id", "user_id", "created_at"],
        "sample_values": {
            "status": ["pending", "completed", "cancelled"],
            "amount": {"min": 10, "max": 1000, "avg": 150}
        }
    },
    "products": {
        "fields": ["product_id", "name", "category", "price", "stock", "ratings"],
        "indexes": ["product_id", "category", "price"],
        "sample_values": {
            "category": ["electronics", "clothing", "books", "home"],
            "price": {"min": 5, "max": 500, "avg": 75}
        }
    },
    "sessions": {
        "fields": ["session_id", "user_id", "start_time", "duration", "pages_visited"],
        "indexes": ["session_id", "user_id", "start_time"],
        "sample_values": {
            "duration": {"min": 30, "max": 3600, "avg": 300}
        }
    }
}

FEATURE_METADATA = {
    "user_retention": {
        "description": "Measures user engagement and retention rates",
        "related_collections": ["users", "sessions", "orders"],
        "key_metrics": ["login_frequency", "session_duration", "purchase_frequency"],
        "prediction_target": "churn_probability"
    },
    "revenue_analysis": {
        "description": "Analyzes revenue patterns and trends",
        "related_collections": ["orders", "products", "users"],
        "key_metrics": ["total_revenue", "average_order_value", "customer_lifetime_value"],
        "prediction_target": "monthly_revenue"
    },
    "product_performance": {
        "description": "Evaluates product success and inventory needs",
        "related_collections": ["products", "orders", "users"],
        "key_metrics": ["sales_volume", "revenue_per_product", "inventory_turnover"],
        "prediction_target": "demand_forecast"
    }
}
```

### Step 3: Build the Intelligence Agents

```python
def query_analyzer(state: AnalyticsState):
    """Analyze user query and determine intent and complexity"""
    start_time = time.time()
    
    user_query = state["original_query"]
    
    # Generate embedding for query similarity
    query_embedding = embeddings.embed_query(user_query)
    
    analysis_prompt = f"""
    Analyze this analytics query and classify it:
    
    Query: "{user_query}"
    
    Available schemas: {json.dumps(SCHEMA_INFO, indent=2)}
    Available features: {json.dumps(FEATURE_METADATA, indent=2)}
    
    Determine:
    1. Intent (analytics, prediction, visualization, data_extraction)
    2. Complexity (simple, medium, complex)
    3. Required collections (max 5-6)
    4. Whether external data is needed
    5. Confidence score (0-1)
    
    Respond in JSON format:
    {{
        "intent": "analytics|prediction|visualization|data_extraction",
        "complexity": "simple|medium|complex",
        "collections_needed": ["collection1", "collection2"],
        "needs_external_data": true|false,
        "needs_visualization": true|false,
        "needs_prediction": true|false,
        "confidence_score": 0.85
    }}
    """
    
    response = llm.invoke([{"role": "user", "content": analysis_prompt}])
    analysis = json.loads(response.content)
    
    processing_time = time.time() - start_time
    
    return {
        "query_intent": analysis["intent"],
        "query_complexity": analysis["complexity"],
        "collections_accessed": analysis["collections_needed"],
        "needs_external_data": analysis["needs_external_data"],
        "needs_visualization": analysis["needs_visualization"],
        "needs_prediction": analysis["needs_prediction"],
        "confidence_score": analysis["confidence_score"],
        "query_execution_time": processing_time,
        "token_usage": {"query_analyzer": len(response.content)},
        "embeddings_cache": {"query": query_embedding}
    }

def mongodb_query_generator(state: AnalyticsState):
    """Generate optimized MongoDB queries"""
    start_time = time.time()
    
    user_query = state["original_query"]
    collections = state["collections_accessed"]
    
    # Generate queries for each collection
    generated_queries = []
    
    for collection in collections:
        schema = SCHEMA_INFO.get(collection, {})
        
        query_prompt = f"""
        Generate an optimized MongoDB aggregation query for this request:
        
        User Query: "{user_query}"
        Collection: {collection}
        Schema: {json.dumps(schema, indent=2)}
        
        Create an efficient aggregation pipeline that:
        1. Uses indexes when possible
        2. Limits results to essential data
        3. Includes necessary grouping/sorting
        4. Optimizes for performance
        
        Return only the aggregation pipeline as a JSON array.
        """
        
        response = llm.invoke([{"role": "user", "content": query_prompt}])
        
        try:
            pipeline = json.loads(response.content)
            generated_queries.append({
                "collection": collection,
                "pipeline": pipeline,
                "estimated_docs": min(1000, schema.get("estimated_size", 100))
            })
        except json.JSONDecodeError:
            # Fallback query
            generated_queries.append({
                "collection": collection,
                "pipeline": [{"$limit": 100}],
                "estimated_docs": 100
            })
    
    processing_time = time.time() - start_time
    
    return {
        "generated_queries": generated_queries,
        "query_execution_time": state["query_execution_time"] + processing_time,
        "token_usage": {
            **state.get("token_usage", {}),
            "query_generator": len(str(generated_queries))
        }
    }

def data_executor(state: AnalyticsState):
    """Execute MongoDB queries and return results"""
    start_time = time.time()
    
    queries = state["generated_queries"]
    results = []
    
    for query_info in queries:
        collection_name = query_info["collection"]
        pipeline = query_info["pipeline"]
        
        try:
            # Execute query with timeout
            collection = db[collection_name]
            cursor = collection.aggregate(pipeline, maxTimeMS=30000)
            
            # Convert to list and limit results
            query_results = list(cursor)[:500]  # Limit to 500 docs
            
            results.append({
                "collection": collection_name,
                "data": query_results,
                "count": len(query_results),
                "execution_time": time.time() - start_time
            })
            
        except Exception as e:
            # Mock data for demonstration
            results.append({
                "collection": collection_name,
                "data": generate_mock_data(collection_name),
                "count": 50,
                "execution_time": 0.1,
                "error": str(e)
            })
    
    processing_time = time.time() - start_time
    
    return {
        "query_results": results,
        "query_execution_time": state["query_execution_time"] + processing_time
    }

def generate_mock_data(collection_name: str):
    """Generate realistic mock data for demonstration"""
    if collection_name == "users":
        return [
            {"user_id": i, "name": f"User {i}", "plan_type": "premium", "signup_date": "2024-01-15"}
            for i in range(1, 51)
        ]
    elif collection_name == "orders":
        return [
            {"order_id": i, "user_id": i % 20, "amount": 150 + i, "status": "completed"}
            for i in range(1, 51)
        ]
    elif collection_name == "products":
        return [
            {"product_id": i, "name": f"Product {i}", "category": "electronics", "price": 75 + i}
            for i in range(1, 51)
        ]
    else:
        return [{"id": i, "value": f"data_{i}"} for i in range(1, 51)]

def insights_generator(state: AnalyticsState):
    """Generate analytical insights from query results"""
    start_time = time.time()
    
    results = state["query_results"]
    user_query = state["original_query"]
    
    # Analyze results and generate insights
    insights = []
    
    for result in results:
        collection = result["collection"]
        data = result["data"]
        
        if not data:
            continue
            
        # Generate collection-specific insights
        if collection == "users":
            total_users = len(data)
            premium_users = sum(1 for u in data if u.get("plan_type") == "premium")
            insights.append(f"Total active users: {total_users}")
            insights.append(f"Premium users: {premium_users} ({premium_users/total_users*100:.1f}%)")
            
        elif collection == "orders":
            total_orders = len(data)
            total_revenue = sum(o.get("amount", 0) for o in data)
            avg_order_value = total_revenue / total_orders if total_orders > 0 else 0
            insights.append(f"Total orders: {total_orders}")
            insights.append(f"Total revenue: ${total_revenue:,.2f}")
            insights.append(f"Average order value: ${avg_order_value:.2f}")
            
        elif collection == "products":
            total_products = len(data)
            avg_price = sum(p.get("price", 0) for p in data) / total_products if total_products > 0 else 0
            insights.append(f"Total products: {total_products}")
            insights.append(f"Average product price: ${avg_price:.2f}")
    
    # Generate high-level insights
    summary_prompt = f"""
    Based on this data analysis, provide 3-5 key business insights:
    
    Query: "{user_query}"
    Data Summary: {json.dumps(insights, indent=2)}
    
    Focus on:
    1. Key trends and patterns
    2. Business implications
    3. Actionable recommendations
    
    Keep insights concise and business-focused.
    """
    
    response = llm.invoke([{"role": "user", "content": summary_prompt}])
    ai_insights = response.content.split('\n')
    
    all_insights = insights + [insight.strip() for insight in ai_insights if insight.strip()]
    
    processing_time = time.time() - start_time
    
    return {
        "insights_generated": all_insights,
        "query_execution_time": state["query_execution_time"] + processing_time,
        "token_usage": {
            **state.get("token_usage", {}),
            "insights_generator": len(response.content)
        }
    }

def prediction_engine(state: AnalyticsState):
    """Generate predictions based on data patterns"""
    start_time = time.time()
    
    if not state.get("needs_prediction", False):
        return {"predictions": {}}
    
    results = state["query_results"]
    user_query = state["original_query"]
    
    predictions = {}
    
    # Find relevant feature metadata
    relevant_feature = None
    for feature, metadata in FEATURE_METADATA.items():
        if any(keyword in user_query.lower() for keyword in metadata["key_metrics"]):
            relevant_feature = feature
            break
    
    if relevant_feature:
        feature_info = FEATURE_METADATA[relevant_feature]
        
        # Generate predictions based on feature type
        if relevant_feature == "revenue_analysis":
            # Mock revenue prediction
            historical_data = [150, 175, 200, 225, 250]  # Monthly revenue (thousands)
            predicted_next_month = historical_data[-1] * 1.1
            predictions["revenue_forecast"] = {
                "next_month": f"${predicted_next_month:.0f}K",
                "confidence": 0.85,
                "trend": "increasing"
            }
            
        elif relevant_feature == "user_retention":
            # Mock churn prediction
            predictions["churn_analysis"] = {
                "high_risk_users": 45,
                "churn_probability": 0.12,
                "retention_rate": 0.88
            }
            
        elif relevant_feature == "product_performance":
            # Mock demand forecasting
            predictions["demand_forecast"] = {
                "high_demand_products": ["Product A", "Product B"],
                "stock_alerts": ["Product C - Low Stock"],
                "seasonal_trends": "Electronics peak expected in Q4"
            }
    
    # Generate AI-powered predictions
    prediction_prompt = f"""
    Based on this data, generate relevant predictions:
    
    Query: "{user_query}"
    Available Data: {json.dumps([r['collection'] for r in results])}
    
    Provide predictions in JSON format with confidence scores.
    """
    
    try:
        response = llm.invoke([{"role": "user", "content": prediction_prompt}])
        ai_predictions = json.loads(response.content)
        predictions.update(ai_predictions)
    except:
        predictions["ai_insight"] = "Unable to generate additional predictions"
    
    processing_time = time.time() - start_time
    
    return {
        "predictions": predictions,
        "query_execution_time": state["query_execution_time"] + processing_time,
        "token_usage": {
            **state.get("token_usage", {}),
            "prediction_engine": len(str(predictions))
        }
    }

def external_data_enricher(state: AnalyticsState):
    """Enrich data with external API calls"""
    start_time = time.time()
    
    if not state.get("needs_external_data", False):
        return {"enriched_data": {}, "external_apis_called": []}
    
    user_query = state["original_query"]
    apis_called = []
    enriched_data = {}
    
    # Determine which APIs to call based on query
    if "weather" in user_query.lower() or "location" in user_query.lower():
        # Mock weather API call
        weather_data = {
            "temperature": 75,
            "condition": "sunny",
            "impact_on_sales": "positive"
        }
        enriched_data["weather"] = weather_data
        apis_called.append("weather_api")
    
    if "market" in user_query.lower() or "competitors" in user_query.lower():
        # Mock market data API
        market_data = {
            "market_trend": "growing",
            "competitor_analysis": {
                "competitor_a": {"market_share": 0.25, "growth_rate": 0.15},
                "competitor_b": {"market_share": 0.20, "growth_rate": 0.10}
            }
        }
        enriched_data["market"] = market_data
        apis_called.append("market_api")
    
    if "industry" in user_query.lower() or "benchmark" in user_query.lower():
        # Mock industry benchmarks
        industry_data = {
            "industry_average_conversion": 0.035,
            "industry_average_cltv": 850,
            "benchmarks": {
                "customer_satisfaction": 4.2,
                "response_time": 2.5
            }
        }
        enriched_data["industry"] = industry_data
        apis_called.append("industry_api")
    
    processing_time = time.time() - start_time
    
    return {
        "enriched_data": enriched_data,
        "external_apis_called": apis_called,
        "query_execution_time": state["query_execution_time"] + processing_time
    }

def visualization_generator(state: AnalyticsState):
    """Generate visualizations and dashboard components"""
    start_time = time.time()
    
    if not state.get("needs_visualization", False):
        return {"visualizations": []}
    
    results = state["query_results"]
    insights = state["insights_generated"]
    
    visualizations = []
    
    # Generate chart configurations
    for result in results:
        collection = result["collection"]
        data = result["data"]
        
        if collection == "orders" and data:
            # Revenue trend chart
            revenue_chart = {
                "type": "line",
                "title": "Revenue Trend",
                "data": {
                    "x": [f"Month {i}" for i in range(1, 7)],
                    "y": [15000, 18000, 22000, 25000, 28000, 32000]
                },
                "config": {
                    "xaxis": {"title": "Time Period"},
                    "yaxis": {"title": "Revenue ($)"}
                }
            }
            visualizations.append(revenue_chart)
            
            # Order status pie chart
            status_chart = {
                "type": "pie",
                "title": "Order Status Distribution",
                "data": {
                    "labels": ["Completed", "Pending", "Cancelled"],
                    "values": [75, 20, 5]
                }
            }
            visualizations.append(status_chart)
            
        elif collection == "users" and data:
            # User growth chart
            user_chart = {
                "type": "bar",
                "title": "User Growth by Plan Type",
                "data": {
                    "x": ["Basic", "Premium", "Enterprise"],
                    "y": [1200, 800, 300]
                },
                "config": {
                    "xaxis": {"title": "Plan Type"},
                    "yaxis": {"title": "Number of Users"}
                }
            }
            visualizations.append(user_chart)
    
    # Generate dashboard layout
    dashboard_config = {
        "layout": "grid",
        "rows": 2,
        "cols": 2,
        "charts": visualizations,
        "filters": ["date_range", "category", "user_segment"],
        "refresh_interval": 300  # 5 minutes
    }
    
    visualizations.append({
        "type": "dashboard",
        "title": "Analytics Dashboard",
        "config": dashboard_config
    })
    
    processing_time = time.time() - start_time
    
    return {
        "visualizations": visualizations,
        "query_execution_time": state["query_execution_time"] + processing_time,
        "token_usage": {
            **state.get("token_usage", {}),
            "visualization_generator": len(str(visualizations))
        }
    }

def response_synthesizer(state: AnalyticsState):
    """Synthesize final response with all components"""
    start_time = time.time()
    
    user_query = state["original_query"]
    insights = state["insights_generated"]
    predictions = state["predictions"]
    visualizations = state["visualizations"]
    enriched_data = state["enriched_data"]
    
    # Create comprehensive response
    response_sections = []
    
    # Executive Summary
    summary = f"""
    ## Analytics Summary for: "{user_query}"
    
    **Query processed in {state['query_execution_time']:.2f} seconds**
    - Collections analyzed: {len(state['collections_accessed'])}
    - Data points processed: {sum(r['count'] for r in state['query_results'])}
    - Confidence score: {state['confidence_score']:.2f}
    """
    response_sections.append(summary)
    
    # Key Insights
    if insights:
        insights_section = "## Key Insights\n\n"
        for i, insight in enumerate(insights[:5], 1):
            insights_section += f"{i}. {insight}\n"
        response_sections.append(insights_section)
    
    # Predictions
    if predictions:
        pred_section = "## Predictions & Forecasts\n\n"
        for key, value in predictions.items():
            if isinstance(value, dict):
                pred_section += f"**{key.replace('_', ' ').title()}:**\n"
                for k, v in value.items():
                    pred_section += f"- {k}: {v}\n"
            else:
                pred_section += f"- {key}: {value}\n"
        response_sections.append(pred_section)
    
    # Visualizations
    if visualizations:
        viz_section = "## Visualizations Available\n\n"
        for viz in visualizations:
            viz_section += f"- {viz['title']} ({viz['type']})\n"
        response_sections.append(viz_section)
    
    # External Data
    if enriched_data:
        ext_section = "## External Data Insights\n\n"
        for source, data in enriched_data.items():
            ext_section += f"**{source.title()}:** {str(data)[:100]}...\n"
        response_sections.append(ext_section)
    
    # Performance Metrics
    performance_section = f"""
    ## Performance Metrics
    - Total execution time: {state['query_execution_time']:.2f}s
    - Token usage: {sum(state.get('token_usage', {}).values())}
    - Cache hits: {state.get('cache_hits', 0)}
    - APIs called: {len(state.get('external_apis_called', []))}
    """
    response_sections.append(performance_section)
    
    final_response = "\n\n".join(response_sections)
    
    processing_time = time.time() - start_time
    
    return {
        "messages": [("assistant", final_response)],
        "query_execution_time": state["query_execution_time"] + processing_time,
        "token_usage": {
            **state.get("token_usage", {}),
            "response_synthesizer": len(final_response)
        }
    }
```

### Step 4: Define Intelligent Routing

```python
def route_by_complexity(state: AnalyticsState):
    """Route based on query complexity and requirements"""
    complexity = state.get("query_complexity", "simple")
    
    if complexity == "simple":
        return "fast_track"
    elif state.get("needs_prediction", False):
        return "prediction_required"
    elif state.get("needs_external_data", False):
        return "external_data_required"
    else:
        return "standard_processing"

def determine_next_step(state: AnalyticsState):
    """Determine next processing step"""
    if not state.get("query_results"):
        return "data_executor"
    elif state.get("needs_external_data", False) and not state.get("enriched_data"):
        return "external_enricher"
    elif state.get("needs_prediction", False) and not state.get("predictions"):
        return "prediction_engine"
    elif state.get("needs_visualization", False) and not state.get("visualizations"):
        return "visualization_generator"
    else:
        return "response_synthesizer"

def quality_check(state: AnalyticsState):
    """Check quality and completeness of results"""
    confidence = state.get("confidence_score", 0)
    
    if confidence  threshold:
            similar_queries.append((cached_query, similarity))
    
    return similar_queries
```

### 2. **Token Usage Optimization**

```python
def optimize_llm_calls(state: AnalyticsState):
    """Optimize LLM calls to reduce token usage"""
    
    # Use smaller models for simple tasks
    def get_optimal_model(task_complexity: str):
        if task_complexity == "simple":
            return ChatOpenAI(model="gpt-3.5-turbo", temperature=0.3)
        elif task_complexity == "medium":
            return ChatOpenAI(model="gpt-4", temperature=0.3, max_tokens=1000)
        else:
            return ChatOpenAI(model="gpt-4", temperature=0.3, max_tokens=2000)
    
    # Batch similar operations
    def batch_process_collections(collections: List[str]):
        """Process multiple collections in single LLM call"""
        batch_prompt = f"""
        Generate MongoDB queries for these collections in one response:
        Collections: {collections}
        
        Return as JSON object with collection names as keys.
        """
        
        response = llm.invoke([{"role": "user", "content": batch_prompt}])
        return json.loads(response.content)
    
    # Use prompt templates
    PROMPT_TEMPLATES = {
        "simple_query": "Generate MongoDB query for {collection}: {user_query}",
        "complex_analysis": "Analyze {collection} data for {user_query}. Focus on: {key_metrics}",
        "prediction": "Predict {target} based on {features} from {collection}"
    }
    
    return state
```

### 3. **Advanced Performance Monitoring**

```python
def performance_monitor(state: AnalyticsState):
    """Monitor and optimize performance"""
    
    metrics = {
        "query_complexity": state.get("query_complexity"),
        "collections_accessed": len(state.get("collections_accessed", [])),
        "total_execution_time": state.get("query_execution_time", 0),
        "token_usage": sum(state.get("token_usage", {}).values()),
        "cache_hit_rate": state.get("cache_hits", 0) / max(1, len(state.get("collections_accessed", []))),
        "confidence_score": state.get("confidence_score", 0)
    }
    
    # Log performance metrics
    log_performance_metrics(metrics)
    
    # Adjust strategy based on performance
    if metrics["total_execution_time"] > 10:  # Too slow
        return "optimize_performance"
    elif metrics["token_usage"] > 5000:  # Too expensive
        return "reduce_token_usage"
    else:
        return "continue_normal"

def log_performance_metrics(metrics: Dict[str, Any]):
    """Log metrics to monitoring system"""
    print(f"Performance Metrics: {json.dumps(metrics, indent=2)}")
    
    # In production, send to monitoring service
    # monitoring_service.log_metrics(metrics)
```

### 4. **Smart Query Optimization**

```python
def optimize_mongodb_queries(state: AnalyticsState):
    """Optimize MongoDB queries for better performance"""
    
    queries = state.get("generated_queries", [])
    optimized_queries = []
    
    for query in queries:
        collection = query["collection"]
        pipeline = query["pipeline"]
        
        # Add optimization stages
        optimized_pipeline = []
        
        # Add $match early if possible
        if not pipeline or pipeline[0].get("$match") is None:
            optimized_pipeline.append({"$match": {"status": {"$ne": "deleted"}}})
        
        # Add original pipeline
        optimized_pipeline.extend(pipeline)
        
        # Add $limit for large datasets
        if not any("$limit" in stage for stage in pipeline):
            optimized_pipeline.append({"$limit": 1000})
        
        # Add projection to reduce data transfer
        essential_fields = get_essential_fields(collection, state["original_query"])
        if essential_fields:
            optimized_pipeline.append({"$project": {field: 1 for field in essential_fields}})
        
        optimized_queries.append({
            "collection": collection,
            "pipeline": optimized_pipeline,
            "estimated_docs": query.get("estimated_docs", 100)
        })
    
    return {"generated_queries": optimized_queries}

def get_essential_fields(collection: str, user_query: str) -> List[str]:
    """Determine essential fields based on query"""
    schema = SCHEMA_INFO.get(collection, {})
    all_fields = schema.get("fields", [])
    
    # Simple keyword matching (in production, use more sophisticated NLP)
    essential = []
    for field in all_fields:
        if field.lower() in user_query.lower():
            essential.append(field)
    
    # Always include ID and common fields
    essential.extend(["_id", "created_at", "updated_at"])
    
    return list(set(essential))
```

## Production Best Practices

### 1. **Scalability Patterns**

```python
# Horizontal scaling with worker nodes
def distribute_work(state: AnalyticsState):
    """Distribute work across multiple workers"""
    
    collections = state["collections_accessed"]
    
    # Split collections across workers
    worker_assignments = {}
    for i, collection in enumerate(collections):
        worker_id = f"worker_{i % 3}"  # 3 workers
        if worker_id not in worker_assignments:
            worker_assignments[worker_id] = []
        worker_assignments[worker_id].append(collection)
    
    return worker_assignments

# Load balancing
def get_least_loaded_worker():
    """Get worker with lowest current load"""
    workers = ["worker_1", "worker_2", "worker_3"]
    loads = {worker: get_worker_load(worker) for worker in workers}
    return min(loads.items(), key=lambda x: x[1])[0]
```

### 2. **Error Recovery and Resilience**

```python
def resilient_query_execution(state: AnalyticsState):
    """Execute queries with resilience patterns"""
    
    queries = state["generated_queries"]
    results = []
    
    for query in queries:
        max_retries = 3
        backoff_delay = 1
        
        for attempt in range(max_retries):
            try:
                # Execute query with timeout
                result = execute_query_with_timeout(query, timeout=30)
                results.append(result)
                break
                
            except Exception as e:
                if attempt == max_retries - 1:
                    # Final attempt failed, use fallback
                    fallback_result = get_fallback_data(query["collection"])
                    results.append(fallback_result)
                else:
                    # Retry with exponential backoff
                    time.sleep(backoff_delay)
                    backoff_delay *= 2
    
    return {"query_results": results}

def get_fallback_data(collection: str):
    """Get fallback data when query fails"""
    return {
        "collection": collection,
        "data": generate_mock_data(collection),
        "count": 50,
        "fallback": True
    }
```

