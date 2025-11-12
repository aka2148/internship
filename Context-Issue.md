# Context Issue
The copilot struggles to maintain context that isn't just the table being queried.

 ### It has no problems giving follow up inofrmation on the same table.  
ex:
<img width="1593" height="547" alt="image" src="https://github.com/user-attachments/assets/e1ca1296-a974-47eb-a021-97490b5a3ff4" />
It retains context of the table without issue here.  

### In a case where the context extends to more than just that data being worked on:  
<img width="1592" height="755" alt="image" src="https://github.com/user-attachments/assets/87999f16-a984-4086-873d-c44aa8be5f8d" />
Ideally the copilot should be able the understand that If the user queried about the quantity value just before a similar query without the specific value should be answered.

## This is mainly an issue with session memory
This can be improved by creating a session context json object to retain the details of the last query and then use it to improve the context being handed to the llm through the query.

```
type SessionContext = {
  last_question?: string;
  last_task?: string;
  last_tables?: string[];
  last_columns?: string[];
  last_filters?: string[];
  last_metrics?: string[];
  last_time_range?: string;
};
```
### Above is an example of how the json object would look.


This is more of a temporary fix and can easily be replaced by a more practical solution like redis or semantic context through pinecone/vectorDB

