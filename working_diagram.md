# LWT Code Execution Flow

```mermaid
sequenceDiagram
    participant User
    participant Apache
    participant IndexPHP
    participant MySQL

    User->>Apache: HTTP GET /
    Apache->>IndexPHP: Serve index.php
    IndexPHP->>IndexPHP: Load core files
    IndexPHP->>MySQL: Connect to database
    IndexPHP->>IndexPHP: Generate interface
    IndexPHP->>User: Return HTML page
```

## Flow Summary:
1. User requests page
2. Apache serves index.php
3. index.php loads core files
4. Database connection established
5. Interface generated
6. HTML returned to user 