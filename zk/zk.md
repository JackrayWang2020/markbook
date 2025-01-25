


```mermaid

graph TD;
    A[Start] --> B[Do something];
    B --> C{Is it ok?};
    C -->|Yes| D[Done];
    C -->|No| E[Fix it];
    E --> B;

```