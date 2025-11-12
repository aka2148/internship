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

