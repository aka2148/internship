# Query Inconsistency

The Request
```
Give me a complete summary of February 2025 production including total tonnage, trips, and equipment utilization
```
Generated two different response queries

1.)
```
SELECT qty_ton AS total_tonnage, total_trips, 0 AS equipment_utilization
FROM production_summary
WHERE EXTRACT(MONTH FROM date) = 2 AND EXTRACT(YEAR FROM date) = 2025
```

2.)
```
SELECT date, shift, SUM(qty_ton) AS total_tonnage, SUM(total_trips) AS total_trips, AVG(excavator + dumper + grader + dozer) AS equipment_utilization
FROM production_summary
WHERE EXTRACT(MONTH FROM date) = 2 AND EXTRACT(YEAR FROM date) = 2025
GROUP BY date, shift

```

This is a slight variation of the quick question **January Summary**  
That returns this query:
```
SELECT SUM(ps.qty_ton) AS total_tonnage, SUM(ps.total_trips) AS total_trips, 
       COUNT(e.id) AS equipment_utilization
FROM production_summary ps
LEFT JOIN equipment e ON ps.user_id = e.user_id
WHERE EXTRACT(MONTH FROM ps.date) = 1 AND EXTRACT(YEAR FROM ps.date) = 2025
```

### Current Execution logic
User → LLM → SQL → Execute
### Ideal Execution Logic
1. Rule-based / embedding-based intent classifier
    ↓
2. Intent Normalization → structured JSON
    ↓
3. Template-based SQL generator (deterministic)
      ↘ if template missing → LLM generation + validation + cache
    ↓
4. Schema validation + auto-fix
    ↓
5. Query execution + alias memory update


    

### Intent Normalization Example
```
User: "Give me a complete summary of February 2025 production including total tonnage, trips, and equipment utilization"
↓
Intent Extractor → 
{
  "intent": "summary_aggregated",
  "time": {"month": 2, "year": 2025},
  "metrics": ["qty_ton", "trip_count"],
  "include_equipment_utilization": true
}
```
### Deterministic Query Generator
**This would be mainly used for very specific commonly used templates such as:**  
Highest production in (month)  
Most productive Machine  
Complete Summary of (month)  

This allows for greater consistency and saves tokens.

Example Intent Objects:
```
 Intent 1: user said "show me total sales"
{
  intent: "get_total_sales",
  table: "sales"
}

 Intent 2: user said "show me sales by month"
{
  intent: "get_sales_by_month",
  table: "sales"
}
```


    
**Sql Generator**
```
interface Intent {
  intent: string;
  table: string;
}

// Simple SQL templates
const templates: Record<string, string> = {
  get_total_sales: `
    SELECT SUM(amount) AS total_sales
    FROM {{table}};
  `,

  get_sales_by_month: `
    SELECT EXTRACT(MONTH FROM date) AS month, SUM(amount) AS total_sales
    FROM {{table}}
    GROUP BY month
    ORDER BY month;
  `
};

// Function to fill the template
function generateSQL(intent: Intent): string {
  const template = templates[intent.intent];

  if (!template) {
    throw new Error(`Unknown intent: ${intent.intent}`);
  }

  return template.replace(/{{table}}/g, intent.table).trim();
}
```
**Example Output**
Input:
```
generateSQL({ intent: "get_total_sales", table: "sales" });
```
Output:
```
SELECT EXTRACT(MONTH FROM date) AS month, SUM(amount) AS total_sales
FROM sales
GROUP BY month
ORDER BY month;
```


    
### Query template example
```
TEMPLATES = {
  summary_aggregated: `
    SELECT date, shift,
           SUM(qty_ton) AS total_tonnage,
           SUM(trip_count) AS total_trips,
           0 AS equipment_utilization
    FROM production_summary
    WHERE EXTRACT(MONTH FROM date) = {month}
      AND EXTRACT(YEAR FROM date) = {year}
    GROUP BY date, shift;
  `,
  summary_detailed: `
    SELECT date, shift, qty_ton AS total_tonnage, trip_count AS total_trips
    FROM production_summary
    WHERE EXTRACT(MONTH FROM date) = {month}
      AND EXTRACT(YEAR FROM date) = {year};
  `
}
```

  
### Optional Memory Map
```
{
  "alias_map": {
    "total_tonnage": "qty_ton",
    "total_trips": "trip_count",
    "equipment_utilization": "computed_field"
  }
}
```



## Secondary Issue
After running the quick question query and attemmpting to run it again it tries to get data from the previously created temporary column equipment utilization  
This also causes issues in requesting summaries of other months.  
<img width="1588" height="597" alt="image" src="https://github.com/user-attachments/assets/a04460ea-ed3a-4637-b985-d9c08593275e" />


The LLM is hallucinating the existence of those columns because it created them temporarily only within the scope of the select clause.  
### Possible fix:
Add a validation layer that checks the generated SQL against your real schema before execution.  
Inject the real schema JSON (from database introspection) into the LLM prompt dynamically
  This can be done like this:
```  Available database schema:
{
  "production_summary": ["date", "shift", "qty_ton", "qty_m3", "target_ton", "target_m3"],
  "trip_summary_by_date": ["trip_date", "shift", "tipper_id", "excavator", "route_or_face", "trip_count"]
}
```
Feeding this to the LLM prevents hallucinations

