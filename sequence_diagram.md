# LWT Application Code Execution Sequence Diagram

```mermaid
sequenceDiagram
    participant User
    participant Apache
    participant index.php
    participant settings.inc.php
    participant connect.inc.php
    participant dbutils.inc.php
    participant utilities.inc.php
    participant MySQL
    participant Other_PHP_Files

    Note over User,Other_PHP_Files: Initial Request Flow
    
    User->>Apache: HTTP GET http://localhost:9090/
    Apache->>index.php: Serve index.php (default)
    
    Note over index.php: Entry Point Initialization
    
    index.php->>index.php: Check if connect.inc.php exists
    alt connect.inc.php not found
        index.php->>User: Display fatal error and die
    end
    
    index.php->>settings.inc.php: require_once('settings.inc.php')
    Note over settings.inc.php: Initialize Application Settings
    settings.inc.php->>settings.inc.php: Set debug flags ($debug, $dsplerrors, $dspltime)
    settings.inc.php->>settings.inc.php: Configure error reporting
    settings.inc.php->>settings.inc.php: Set execution time limits (600s)
    settings.inc.php->>settings.inc.php: Set memory limit (999M)
    settings.inc.php->>settings.inc.php: Start PHP session if not exists
    
    index.php->>connect.inc.php: require_once('connect.inc.php')
    Note over connect.inc.php: Load Database Configuration
    connect.inc.php->>connect.inc.php: Set database variables ($server, $userid, $passwd, $dbname)
    
    index.php->>dbutils.inc.php: require_once('dbutils.inc.php')
    Note over dbutils.inc.php: Load Database Utility Functions
    dbutils.inc.php->>dbutils.inc.php: Define do_mysqli_query(), runsql(), get_first_value()
    
    index.php->>utilities.inc.php: require_once('utilities.inc.php')
    Note over utilities.inc.php: Initialize Core Application
    
    utilities.inc.php->>utilities.inc.php: Start execution timer (if $dspltime enabled)
    utilities.inc.php->>utilities.inc.php: Configure mysqli reporting
    utilities.inc.php->>MySQL: mysqli_connect($server, $userid, $passwd, $dbname)
    
    alt Database doesn't exist
        utilities.inc.php->>MySQL: Connect without database
        utilities.inc.php->>MySQL: CREATE DATABASE `lwt`
        utilities.inc.php->>MySQL: Reconnect with database
    end
    
    utilities.inc.php->>MySQL: SET NAMES 'utf8'
    utilities.inc.php->>MySQL: SET SESSION sql_mode = ''
    
    Note over utilities.inc.php: Initialize Table Prefix
    utilities.inc.php->>utilities.inc.php: Check if $tbpref is set
    alt $tbpref not set
        utilities.inc.php->>utilities.inc.php: Set $fixed_tbpref = 0
        utilities.inc.php->>utilities.inc.php: Get prefix from LWTTableGet("current_table_prefix")
        alt No prefix in database
            utilities.inc.php->>utilities.inc.php: Set $tbpref = '' (default)
        end
    else $tbpref is set
        utilities.inc.php->>utilities.inc.php: Set $fixed_tbpref = 1
    end
    
    utilities.inc.php->>utilities.inc.php: Validate table prefix format
    utilities.inc.php->>utilities.inc.php: Add '_' suffix if prefix not empty
    utilities.inc.php->>utilities.inc.php: Save current prefix to database
    
    Note over utilities.inc.php: Database Maintenance
    utilities.inc.php->>utilities.inc.php: check_update_db()
    utilities.inc.php->>MySQL: Check database version
    utilities.inc.php->>MySQL: Run schema updates if needed
    utilities.inc.php->>MySQL: Daily maintenance (score recalculation, cleanup)
    
    Note over index.php: Main Application Logic
    
    index.php->>index.php: Check if database is empty
    alt Database is empty
        index.php->>User: Display hint to install demo or define first language
    end
    
    index.php->>index.php: Get current language and text from settings
    index.php->>MySQL: SELECT count(*) FROM languages
    index.php->>index.php: Display language selector if languages exist
    
    index.php->>index.php: Display main navigation menu
    Note over index.php: Generate HTML for main interface
    index.php->>index.php: Show last text (if exists)
    index.php->>index.php: Display navigation links (My Texts, Languages, etc.)
    
    Note over index.php: Page Footer
    index.php->>MySQL: Calculate database size
    index.php->>index.php: Display system information
    index.php->>index.php: Show version, database info, server info
    
    index.php->>utilities.inc.php: pageend()
    utilities.inc.php->>User: Output closing HTML tags
    
    Note over User,Other_PHP_Files: User Interaction Flow
    
    User->>Other_PHP_Files: Click on navigation links
    Other_PHP_Files->>Other_PHP_Files: Include same core files (settings, connect, dbutils, utilities)
    Other_PHP_Files->>MySQL: Execute specific functionality
    Other_PHP_Files->>User: Return appropriate HTML/JSON response
    
    Note over User,Other_PHP_Files: Alternative Entry Points
    
    User->>Apache: HTTP GET http://localhost:9090/start.php
    Apache->>start.php: Serve start.php
    start.php->>start.php: Include core files
    start.php->>start.php: Check for table set selection
    alt Multiple table sets available
        start.php->>User: Display table set selection form
        User->>start.php: Submit table set selection
        start.php->>start.php: Save selected prefix
        start.php->>index.php: Redirect to index.php
    else Single or no table sets
        start.php->>index.php: Redirect to index.php
    end
```

## Key Components and Their Roles:

### 1. **Entry Points**
- **`index.php`**: Main application entry point
- **`start.php`**: Table set selection entry point

### 2. **Core Initialization Files**
- **`settings.inc.php`**: Application configuration and session management
- **`connect.inc.php`**: Database connection parameters
- **`dbutils.inc.php`**: Database utility functions
- **`utilities.inc.php`**: Core application logic and database connection

### 3. **Database Layer**
- **MySQL**: Database server (MySQL 5.7 in Docker)
- **Connection**: Established via mysqli_connect()
- **Table Prefix**: Supports multiple table sets with prefixes

### 4. **Application Flow**
1. **Initialization**: Load core files and establish database connection
2. **Configuration**: Set up table prefixes and validate settings
3. **Maintenance**: Run daily database maintenance tasks
4. **Interface**: Generate main application interface
5. **Interaction**: Handle user navigation and functionality

### 5. **Error Handling**
- Database connection failures
- Missing configuration files
- Invalid table prefixes
- Session management issues

### 6. **Docker Integration**
- Apache serves files from `/var/www/html` (mapped to `./lwt_html`)
- MySQL container provides database service
- Port 9090 exposed for web access

This sequence diagram shows the complete flow from initial HTTP request through application initialization, database connection, and page rendering, including error handling and alternative entry points. 