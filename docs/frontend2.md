# Frontend Architecture

## Overview

The LWT frontend is a traditional server-rendered PHP application with progressive JavaScript enhancement. It uses a multi-frame layout for the reading interface (desktop) and a responsive iframe-based layout for mobile devices. AJAX is used extensively for dynamic updates without full page reloads.

## Architecture Pattern

- **Server-Side Rendering**: PHP generates HTML directly
- **Progressive Enhancement**: Works without JavaScript; enhanced with it
- **Multi-Frame Layout**: Desktop uses HTML framesets; mobile uses absolute-positioned divs
- **AJAX Communication**: Dynamic updates via jQuery AJAX calls

## Page Structure & Rendering

### Template System

- **`pagestart()`** and **`pageend()`** functions in `utilities.inc.php` wrap all pages
- **`pagestart_nobody()`** outputs HTML head with CSS/JS includes
- Pages are server-rendered PHP that output HTML directly

### Frameset Architecture (Reading Interface)

The main reading interface (`do_text.php`) uses a 4-frame layout:

- **Frame `h`**: Header with controls (`do_text_header.php`)
- **Frame `l`**: Main text content (`do_text_text.php`)
- **Frame `ro`**: Right-top (word details/editing)
- **Frame `ru`**: Right-bottom (additional info)

**Desktop**: Uses HTML `<frameset>` elements
**Mobile**: Uses absolute-positioned divs with iframes instead of `<frameset>`

## JavaScript Architecture

### Core Libraries

- **jQuery** (`jquery.js`) - DOM manipulation and AJAX
- **jQuery UI** (`jquery-ui.min.js`) - UI components
- **Overlib** (`overlib/`) - Popup tooltips for word interactions
- **Tag-it** (`tag-it.js`) - Tag management UI
- **jEditable** (`jquery.jeditable.mini.js`) - Inline editing
- **ScrollTo** (`jquery.scrollTo.min.js`) - Smooth scrolling

### Custom JavaScript Files

#### `pgm.js` - Core LWT Functions
- Overlib popup generation for word interactions
- HTML escaping utilities
- Link generation for word operations

#### `jq_pgm.js` - jQuery-Based Functionality
- Global state variables (`TEXTPOS`, `OPENED`, `WID`, `TID`, etc.)
- Event handlers for word clicks, keyboard navigation
- AJAX functions for dynamic updates
- Form validation
- Text editing and annotation functions

### Global State Management

Global JavaScript variables track UI state:

```javascript
var TEXTPOS = -1;  // Current position in text
var OPENED = 0;    // Whether word popup is open
var WID = 0;       // Current word ID
var TID = 0;       // Current text ID
var ANN_ARRAY = {}; // Annotations array
```

## AJAX Communication

### AJAX Endpoints

All AJAX endpoints are in `ajax_*.php` files:

- `ajax_add_term_transl.php` - Add translation to term
- `ajax_chg_term_status.php` - Change word status
- `ajax_edit_impr_text.php` - Edit improved text
- `ajax_save_impr_text.php` - Save improved text annotations
- `ajax_save_setting.php` - Save user settings
- `ajax_show_sentences.php` - Show example sentences
- `ajax_show_similar_terms.php` - Show similar terms
- `ajax_update_media_select.php` - Update media selector
- `ajax_word_counts.php` - Get word statistics

### AJAX Pattern

- Uses jQuery `$.post()` for POST requests
- Responses are JSON (some endpoints return HTML fragments)
- Loading indicators using `waiting2.gif`

## CSS Architecture

### Stylesheets

- **`styles.css`** - Main stylesheet with:
  - Word status color coding (status0-5, status98, status99)
  - Typography and layout
  - Form styling
  - Table styles
- **`jquery-ui.css`** - jQuery UI theme
- **`jquery.tagit.css`** - Tag-it plugin styles

### Status-Based Styling

Words are colored by learning status:

- `status0` (unknown): Light blue (`#ADDFFF`)
- `status1-5`: Red to green gradient (learning progression)
  - `status1`: `#F5B8A9` (red)
  - `status2`: `#F5CCA9` (orange)
  - `status3`: `#F5E1A9` (yellow)
  - `status4`: `#F5F3A9` (light yellow)
  - `status5`: `#DDFFDD` (green)
- `status98` (ignore): Gray with dashed border (`#F8F8F8`)
- `status99` (well-known): Gray with solid border (`#F8F8F8`)

## User Interaction Patterns

### Word Interaction

- **Click**: Shows overlib popup with word details and actions
- **Double-click**: Navigates audio player to word position
- **Keyboard shortcuts**:
  - Arrow keys: Navigate known words
  - Number keys (1-5): Set word status
  - `E`: Edit word
  - `I`: Ignore word (status 98)
  - `W`: Mark as well-known (status 99)
  - `A`: Set audio position
  - `Esc`: Reset selection

### Text Reading Flow

1. User clicks word → `word_click_event_do_text_text()` fires
2. Overlib popup shows word details
3. Frame `ro` loads word details via `show_word.php`
4. User can edit, change status, or view dictionary links
5. AJAX updates persist changes without page reload

## Mobile Support

- **`Mobile_Detect.php`** detects mobile devices
- Conditional rendering: framesets (desktop) vs. absolute-positioned divs (mobile)
- **`mobile.php`** provides experimental mobile interface
- **iUI framework** (`iui/`) for mobile UI patterns

## Data Flow

```
User Action → JavaScript Event Handler → AJAX Request → PHP Endpoint → 
Database Query → Response (JSON/HTML) → DOM Update
```

### Example: Word Status Change

1. User presses keyboard shortcut (e.g., "1")
2. `keydown_event_do_text_text()` intercepts
3. Navigates frame `ro` to `set_word_status.php?wid=X&status=1`
4. PHP updates database
5. Page refreshes to show new status color

## Key Frontend Features

1. **Progressive Enhancement**: Works without JavaScript; enhanced with it
2. **Frame Communication**: Cross-frame navigation via `window.parent.frames`
3. **Real-time Updates**: AJAX for settings, word counts, similar terms
4. **Inline Editing**: jEditable for quick edits
5. **Tag Management**: Tag-it for word and text tagging
6. **Audio Integration**: jPlayer for audio playback with word synchronization

## File Structure

```
lwt_html/
├── js/
│   ├── jquery.js
│   ├── jquery-ui.min.js
│   ├── pgm.js              # Core LWT functions
│   ├── jq_pgm.js            # jQuery-based functionality
│   ├── tag-it.js
│   ├── jquery.jeditable.mini.js
│   ├── jquery.scrollTo.min.js
│   ├── overlib/             # Popup library
│   └── ...
├── css/
│   ├── styles.css           # Main stylesheet
│   ├── jquery-ui.css
│   └── jquery.tagit.css
├── ajax_*.php               # AJAX endpoints
├── do_text.php              # Main reading interface (frameset)
├── do_text_text.php         # Text content frame
├── do_text_header.php       # Header frame
└── utilities.inc.php        # Template functions (pagestart/pageend)
```

