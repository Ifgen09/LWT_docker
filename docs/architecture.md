### Overview

- **Purpose**: Containerized runner for the Learning With Texts (LWT) PHP application.
- **Topology**: Two-container Docker Compose stack:
  - **`lwt`**: PHP 7.4 + Apache web server hosting the LWT PHP app.
  - **`db`**: MySQL 5.7 database with persistent storage.

### Services

- **Application (`lwt`)**
  - Base image: `php:7.4-apache`
  - Installs `mysqli` extension and MySQL client
  - Serves code from bind mount: `./lwt_html -> /var/www/html`
  - HTTP exposed on container port 80; mapped to host port `9090`
  - Built via root `Dockerfile`

- **Database (`db`)**
  - Image: `mysql:5.7`
  - Initializes database `lwt` via environment variables
  - Persists data in named volume `db_data -> /var/lib/mysql`

### Data & Configuration

- **Volumes**
  - `./lwt_html`: Local copy of upstream LWT PHP code; mounted read/write into the web container
  - `db_data`: Named volume for MySQL data persistence across restarts

- **Networking**
  - Default Compose network; `lwt` reaches `db` by service name
  - Host access: `[http://localhost:9090](http://localhost:9090)`

- **Environment** (from `docker-compose.yml`)
  - `MYSQL_DATABASE=lwt`
  - `MYSQL_ROOT_PASSWORD=root`
  - `MYSQL_ALLOW_EMPTY_PASSWORD=yes`

### Runtime flow

- Browser → `localhost:9090` → Apache in `lwt` → LWT PHP app → MySQL (`db`) via `mysqli`
- MySQL data persists in `db_data`

### Usage

- Build and run:
  - `docker compose build lwt`
  - `docker compose up lwt`
- Access the app at: `[http://localhost:9090](http://localhost:9090)` 

### Database schema

- **Engine**: MyISAM (no enforced foreign keys). Relations are logical by naming.
- **Core entities**: `languages`, `texts`, `archivedtexts`, `words`.
- **Tagging**:
  - `tags` ↔ `wordtags` for word-level tags
  - `tags2` ↔ `texttags` and `archtexttags` for text/archived text tags
- **Content structure**:
  - `sentences` and `textitems` index/slice `texts` content and link occurrences to `words`
- **Globals**: `_lwtgeneral`, `settings` key-value pairs

```mermaid
erDiagram
  languages ||--o{ texts : has
  languages ||--o{ archivedtexts : has
  languages ||--o{ words : has

  texts ||--o{ sentences : has
  texts ||--o{ textitems : has
  texts ||--o{ texttags : tagged

  archivedtexts ||--o{ archtexttags : tagged

  tags2 ||--o{ texttags : used_by
  tags2 ||--o{ archtexttags : used_by

  words ||--o{ wordtags : tagged
  tags ||--o{ wordtags : used_by

  sentences ||--o{ textitems : indexes

  _lwtgeneral {
    varchar LWTKey PK
    varchar LWTValue
  }
  settings {
    varchar StKey PK
    varchar StValue
  }
  languages {
    int LgID PK
    varchar LgName
  }
  texts {
    int TxID PK
    int TxLgID FK
    varchar TxTitle
    text TxText
  }
  archivedtexts {
    int AtID PK
    int AtLgID FK
    varchar AtTitle
    text AtText
  }
  sentences {
    int SeID PK
    int SeLgID FK
    int SeTxID FK
  }
  words {
    int WoID PK
    int WoLgID FK
    varchar WoText
  }
  textitems {
    int TiID PK
    int TiTxID FK
    int TiSeID FK
    int TiWoID FK
  }
  tags {
    int TgID PK
    varchar TgText
  }
  tags2 {
    int T2ID PK
    varchar T2Text
  }
  wordtags {
    int WtWoID FK
    int WtTgID FK
  }
  texttags {
    int TtTxID FK
    int TtT2ID FK
  }
  archtexttags {
    int AgAtID FK
    int AgT2ID FK
  }
```

- Keys/fields shown are representative; the live schema includes additional columns (timestamps, annotations, URIs, status fields, etc.). 