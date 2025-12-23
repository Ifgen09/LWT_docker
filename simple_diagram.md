# Simple LWT Code Execution Flow

```mermaid
sequenceDiagram
    participant U as User
    participant A as Apache
    participant I as index.php
    participant M as MySQL

    U->>A: HTTP GET /
    A->>I: Serve index.php
    I->>I: Load core files
    I->>M: Connect to database
    I->>I: Generate interface
    I->>U: Return HTML page
```

## Flow Summary:
1. User requests page
2. Apache serves index.php
3. index.php loads core files
4. Database connection established
5. Interface generated
6. HTML returned to user 