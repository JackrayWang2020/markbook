


```mermaid

graph TD;
    A[Start] --> B[Do something];
    B --> C{Is it ok?};
    C -->|Yes| D[Done];
    C -->|No| E[Fix it];
    E --> B;

```


```vega
{
  "data": {
    "values": [
      {"x": 1, "y": 2},
      {"x": 2, "y": 3},
      {"x": 3, "y": 4}
    ]
  },
  "mark": "line",
  "encoding": {
    "x": {"field": "x", "type": "quantitative"},
    "y": {"field": "y", "type": "quantitative"}
  }
}
```