# LWT Application Code Execution Sequence Diagram (Text Version)

## Execution Flow

```
User Request
    ↓
HTTP GET http://localhost:9090/
    ↓
Apache Web Server
    ↓
Serve index.php (default entry point)
    ↓
index.php
    ↓
Check if connect.inc.php exists
    ↓
[If not found] → Display fatal error and die
    ↓
[If found] → Continue
    ↓
require_once('settings.inc.php')
    ↓
settings.inc.php
    ├── Set debug flags ($debug, $dsplerrors, $dspltime)
    ├── Configure error reporting
    ├── Set execution time limits (600s)
    ├── Set memory limit (999M)
    └── Start PHP session if not exists
    ↓
require_once('connect.inc.php')
    ↓
connect.inc.php
    └── Set database variables ($server, $userid, $passwd, $dbname)
    ↓
require_once('dbutils.inc.php')
    ↓
dbutils.inc.php
    └── Define database utility functions (do_mysqli_query, runsql, get_first_value)
    ↓
require_once('utilities.inc.php')
    ↓
utilities.inc.php
    ├── Start execution timer (if enabled)
    ├── Configure mysqli reporting
    ├── mysqli_connect($server, $userid, $passwd, $dbname)
    │   ↓
    │   [If database doesn't exist]
    │   ├── Connect without database
    │   ├── CREATE DATABASE `lwt`
    │   └── Reconnect with database
    │   ↓
    ├── SET NAMES 'utf8'
    ├── SET SESSION sql_mode = ''
    ├── Initialize table prefix ($tbpref)
    ├── Validate prefix format
    ├── Save prefix to database
    ├── check_update_db()
    │   ├── Check database version
    │   ├── Run schema updates if needed
    │   └── Daily maintenance (score recalculation, cleanup)
    └── Return to index.php
    ↓
index.php (Main Application Logic)
    ├── Check if database is empty
    │   ↓
    │   [If empty] → Display hint to install demo or define first language
    │   ↓
    ├── Get current language and text from settings
    ├── SELECT count(*) FROM languages
    ├── Display language selector if languages exist
    ├── Generate main navigation menu
    │   ├── Show last text (if exists)
    │   ├── Display navigation links (My Texts, Languages, etc.)
    │   └── Show system information
    ├── Calculate database size
    ├── Display system information
    ├── pageend()
    └── Output closing HTML tags
    ↓
User receives rendered HTML page
```

## Alternative Entry Point: start.php

```
User Request
    ↓
HTTP GET http://localhost:9090/start.php
    ↓
Apache Web Server
    ↓
Serve start.php
    ↓
start.php
    ├── Include core files (same as index.php)
    ├── Check for table set selection
    │   ↓
    │   [If multiple table sets available]
    │   ├── Display table set selection form
    │   ├── User submits selection
    │   ├── Save selected prefix
    │   └── Redirect to index.php
    │   ↓
    │   [If single or no table sets]
    │   └── Redirect to index.php
    ↓
Redirect to index.php (main flow continues)
```

## User Interaction Flow

```
User clicks navigation links
    ↓
Other PHP files (edit_texts.php, edit_languages.php, etc.)
    ├── Include same core files (settings, connect, dbutils, utilities)
    ├── Execute specific functionality
    ├── Query database as needed
    └── Return appropriate HTML/JSON response
    ↓
User receives updated page content
```

## Key Components Summary

### Entry Points
- **Primary**: `index.php` - Main application entry point
- **Secondary**: `start.php` - Table set selection entry point

### Core Initialization Files
1. **settings.inc.php** - Application configuration and session management
2. **connect.inc.php** - Database connection parameters
3. **dbutils.inc.php** - Database utility functions
4. **utilities.inc.php** - Core application logic and database connection

### Database Layer
- **Server**: MySQL 5.7 (Docker container)
- **Connection**: Established via mysqli_connect()
- **Features**: Table prefix support, UTF-8 encoding, daily maintenance

### Web Server
- **Server**: Apache (serves from `/var/www/html`)
- **Port**: 9090 (exposed in Docker)
- **Mapping**: `./lwt_html` → `/var/www/html`

### Error Handling
- Database connection failures
- Missing configuration files
- Invalid table prefixes
- Session management issues

## Application Flow Phases

1. **Initialization Phase**
   - Load core configuration files
   - Establish database connection
   - Set up application settings

2. **Configuration Phase**
   - Initialize table prefixes
   - Validate system settings
   - Run database maintenance

3. **Interface Phase**
   - Generate main application interface
   - Display navigation menu
   - Show system information

4. **Interaction Phase**
   - Handle user navigation
   - Process form submissions
   - Execute specific functionality

This text-based sequence diagram shows the complete execution flow without requiring Mermaid rendering, making it accessible in any environment. 