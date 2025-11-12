# The Issue
Currently the router is using a hybrid model of both the llm and rule based routing.  
The issue is that the llm is being called before the rule-based routing which eliminates some of the benefits of using rule based routing.

## Token Issue
### In the current logic the LLM is always called to decide the route even in rule-based could easily decide it.
This adds latency issues and increases token usage.  
Rule Based routing is basically instant so the reduction in query time will be significant
Flipping the order will allow the rule-based routing to minimize latency and usage of tokens by routing the more obvious queries.  
Queries that include the keywords graphs, chart, pie chart can all very safely be assumed to take the chart route without having to trouble the llm.

### Currently the logic is based on this prompt
```
const prompt = `You are a mining operations query router. Analyze this question and decide whether it should be answered via SQL database query or RAG (document retrieval).
```
This creates a back and forth between the llm which is highly unnecessary.
 

### This also has benefits in confidentality as LLM does not know what information might be sensitive and is much more prone to being tricked then a simple set of guidelines to prevent exploitation.
