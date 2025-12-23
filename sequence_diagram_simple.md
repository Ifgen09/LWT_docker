# LWT Application Code Execution Sequence Diagram

```mermaid
sequenceDiagram
    participant User
    participant Apache
    participant IndexPHP
    participant Settings
    participant Connect
    participant DButils
    participant Utilities
    participant MySQL

    User->>Apache: HTTP GET http://localhost:9090/
    Apache->>IndexPHP: Serve index.php
    
    IndexPHP->>IndexPHP: Check connect.inc.php exists
    alt File not found
        IndexPHP->>User: Display fatal error
    end
    
    IndexPHP->>Settings: require_once('settings.inc.php')
    Settings->>Settings: Set debug flags
    Settings->>Settings: Configure error reporting
    Settings->>Settings: Set execution limits
    Settings->>Settings: Start PHP session
    
    IndexPHP->>Connect: require_once('connect.inc.php')
    Connect->>Connect: Set database variables
    
    IndexPHP->>DButils: require_once('dbutils.inc.php')
    DButils->>DButils: Define database functions
    
    IndexPHP->>Utilities: require_once('utilities.inc.php')
    Utilities->>Utilities: Start execution timer
    Utilities->>MySQL: mysqli_connect()
    
    alt Database doesn't exist
        Utilities->>MySQL: Connect without database
        Utilities->>MySQL: CREATE DATABASE lwt
        Utilities->>MySQL: Reconnect with database
    end
    
    Utilities->>MySQL: SET NAMES 'utf8'
    Utilities->>MySQL: SET SESSION sql_mode = ''
    
    Utilities->>Utilities: Initialize table prefix
    Utilities->>Utilities: Validate prefix format
    Utilities->>Utilities: Save prefix to database
    
    Utilities->>Utilities: check_update_db()
    Utilities->>MySQL: Check database version
    Utilities->>MySQL: Run schema updates
    Utilities->>MySQL: Daily maintenance
    
    IndexPHP->>IndexPHP: Check if database is empty
    alt Database empty
        IndexPHP->>User: Display setup hints
    end
    
    IndexPHP->>MySQL: SELECT count(*) FROM languages
    IndexPHP->>IndexPHP: Display language selector
    
    IndexPHP->>IndexPHP: Generate main interface
    IndexPHP->>IndexPHP: Show navigation menu
    IndexPHP->>IndexPHP: Display last text info
    
    IndexPHP->>MySQL: Calculate database size
    IndexPHP->>IndexPHP: Display system info
    IndexPHP->>Utilities: pageend()
    Utilities->>User: Output closing HTML
```

## Application Flow Summary:

1. **Entry Point**: `index.php` serves as main entry point
2. **Initialization**: Loads core files in sequence:
   - `settings.inc.php` - Application configuration
   - `connect.inc.php` - Database parameters  
   - `dbutils.inc.php` - Database utilities
   - `utilities.inc.php` - Core application logic
3. **Database Connection**: Establishes MySQL connection with error handling
4. **Configuration**: Sets up table prefixes and validates settings
5. **Maintenance**: Runs daily database maintenance tasks
6. **Interface**: Generates main application interface
7. **User Interaction**: Handles navigation and functionality

## Key Files:
- **Entry**: `index.php`, `start.php`
- **Core**: `settings.inc.php`, `connect.inc.php`, `dbutils.inc.php`, `utilities.inc.php`
- **Database**: MySQL 5.7 (Docker container)
- **Web Server**: Apache (serves from `/var/www/html`) 