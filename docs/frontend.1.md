# Frontend Architecture

## Overview

The LWT frontend is a traditional multi-page application (MPA) built with PHP server-side rendering, jQuery for client-side interactivity, and a frameset-based layout system. It follows a classic web application pattern rather than a modern single-page application (SPA) architecture.

## Layout System: Multi-Frame Architecture

The application uses a frameset-based layout system (with iframes for mobile compatibility):

### Desktop (Framesets)

The main reading interface (`do_text.php`) uses HTML framesets to create a persistent multi-pane layout:

- **Frame `h`** (top-left): Header with audio player and controls
  - Source: `do_text_header.php`
  - Contains audio playback controls, text navigation, and settings
  
- **Frame `l`** (bottom-left): Main text content area
  - Source: `do_text_text.php`
  - Displays the annotated text with clickable words
  - Handles word interactions and keyboard navigation
  
- **Frame `ro`** (top-right): Word details/definitions panel
  - Initially loads `empty.htm`
  - Populated dynamically when words are clicked
  - Shows word translations, status, and related information
  
- **Frame `ru`** (bottom-right): Additional content area
  - Initially loads `empty.htm`
  - Used for supplementary content display

**Frame Configuration:**
- Left frame width: Configurable percentage (default via `set-text-l-framewidth-percent`)
- Right frame height: Configurable percentage (default via `set-text-r-frameheight-percent`)
- Header height: Varies based on audio player presence
  - Without audio: `set-text-h-frameheight-no-audio` (pixels)
  - With audio: `set-text-h-frameheight-with-audio` (pixels)

### Mobile (Iframes)

For mobile devices, the application uses positioned `<div>` containers with iframes instead of framesets:

- Uses `Mobile_Detect.php` to detect mobile devices
- Responsive sizing handled via jQuery
- Same four-pane layout structure maintained
- Touch-friendly scrolling enabled

## Technology Stack

### JavaScript Libraries

**Core:**
- **jQuery** (`jquery.js`) - Core DOM manipulation and AJAX
- **jQuery UI** (`jquery-ui.min.js`) - UI components and interactions

**Plugins:**
- `jquery.jeditable.mini.js` - Inline editing functionality
- `jquery.jplayer.min.js` - Audio playback (MP3 support)
- `jquery.scrollTo.min.js` - Smooth scrolling to elements
- `tag-it.js` - Tag input/management interface

**Third-Party Libraries:**
- **Overlib** (`overlib/`) - Tooltip and popup system
  - Multiple modules: crossframe, shadow, exclusive, followscroll, etc.
- **SortTable** (`sorttable/`) - Table sorting functionality

**Custom JavaScript:**
- `jq_pgm.js` (700 lines) - Main application logic
  - Word interaction handlers
  - Keyboard navigation
  - AJAX communication
  - Form validation
  - Global state management
- `pgm.js` - Additional utility functions
- `floating.js` - Floating UI elements
- `countuptimer.js` - Timer functionality
- `unloadformcheck.js` - Form change detection

### CSS Architecture

**Stylesheets:**
- `styles.css` - Main application stylesheet
  - Status-based color coding (status0-5, status98, status99)
  - Layout and typography
  - Form styling
- `jquery-ui.css` - jQuery UI theme
- `jquery.tagit.css` - Tag input styling
- `jplayer_skin/` - Audio player skin

**Status Color System:**
- `status0` - Unknown words (light blue: #ADDFFF)
- `status1-5` - Learning progression (red → orange → yellow → light yellow → green)
- `status98` - Ignored words (gray with dashed border)
- `status99` - Well-known words (gray with solid green border)

## Frontend-Backend Communication

### AJAX Endpoints

The frontend communicates with the backend via jQuery AJAX calls to PHP endpoints:

**Text Management:**
- `ajax_save_impr_text.php` - Save improved text annotations
- `ajax_edit_impr_text.php` - Edit improved text content
- `ajax_add_term_transl.php` - Add term translations

**Word/Term Management:**
- `ajax_chg_term_status.php` - Change term learning status
- `ajax_show_sentences.php` - Retrieve example sentences
- `ajax_show_similar_terms.php` - Find similar terms
- `ajax_word_counts.php` - Get word statistics

**Settings & UI:**
- `ajax_save_setting.php` - Save user preferences
- `ajax_update_media_select.php` - Update media file selector

### Communication Pattern

**Request Format:**
```javascript
$.post('ajax_endpoint.php', { 
    param1: value1, 
    param2: value2 
}, function(data) {
    // Handle response
});
```

**Response Handling:**
- HTML snippets: Directly inserted into DOM
- JSON: Parsed and used to update UI
- Status strings: Simple success/error messages

**Cross-Frame Communication:**
- Uses `window.parent.frames['frame_name']` to access other frames
- Navigation: `window.parent.frames['ro'].location.href = 'url'`
- Function calls: `window.parent.frames['h'].new_pos(percentage)`

## Key Frontend Features

### Word Interaction System

**Click Handlers:**
- Single click: Show word details in right panel
- Double click: Navigate audio player to word position
- Status-based behavior:
  - Status 0 (unknown): Opens word creation dialog
  - Status 1-5: Shows word details with learning status
  - Status 98 (ignored): Shows word with ignore indicator
  - Status 99 (well-known): Shows word with known indicator

**Keyboard Navigation:**
- **Arrow keys (←/→)**: Navigate between known words
- **Home/End**: Jump to first/last known word
- **Number keys (1-5)**: Set word status
- **I**: Set status to 98 (ignored)
- **W**: Set status to 99 (well-known)
- **E**: Edit word
- **A**: Set audio position to word
- **ESC**: Reset selection
- **Enter**: Jump to next unknown word
- **Space**: Show solution (in test mode)

**Tooltip System:**
- Uses Overlib library for word tooltips
- Displays translation, romanization, and status
- Cross-frame tooltip support

### Global State Management

The application uses global JavaScript variables for state:

```javascript
var TEXTPOS = -1;        // Current position in known words list
var OPENED = 0;          // Whether word detail is open
var WID = 0;             // Current word ID
var TID = 0;             // Current text ID
var WBLINK1 = '';        // Dictionary link 1
var WBLINK2 = '';        // Dictionary link 2
var WBLINK3 = '';        // Dictionary link 3 (Google Translate)
var SOLUTION = '';       // Test solution text
var ADDFILTER = '';      // Additional CSS filter for word display
var RTL = 0;             // Right-to-left script flag
var ANN_ARRAY = {};      // Annotations array
```

### Form Validation

**Validation Function:** `check()` in `jq_pgm.js`

**Validation Rules:**
- Required fields (`.notempty` class)
- URL validation (`.checkurl`, `.checkdicturl` classes)
- Integer validation (`.posintnumber`, `.zeroposintnumber` classes)
- Unicode BMP validation (`.checkoutsidebmp` class)
- Text length validation (`.checklength`, `.checkbytes` classes)
- No spaces/commas (`.noblanksnocomma` class)

**Validation Classes:**
Applied via HTML attributes:
- `class="notempty"` - Field must not be empty
- `class="checkurl"` - Must be valid URL
- `data_info` - Field name for error messages
- `data_maxlength` - Maximum length constraint

## Page Organization

### Entry Points

**Main Pages:**
- `index.php` - Home page and main menu
- `do_text.php` - Text reading interface (frameset)
- `do_test.php` - Testing interface (frameset)
- `mobile.php` - Mobile version (iUI framework)

**Component Pages:**
- `do_text_header.php` - Header frame content
- `do_text_text.php` - Text content frame
- `do_test_test.php` - Test interface frame
- `do_test_header.php` - Test header frame

**CRUD Pages:**
- `edit_texts.php` - Text management
- `edit_words.php` - Word/term management
- `edit_languages.php` - Language management
- `edit_tags.php` - Tag management
- Various other `edit_*.php` files

### Page Structure Pattern

**Standard Page Flow:**
1. Include core files:
   ```php
   require_once('settings.inc.php');
   require_once('connect.inc.php');
   require_once('dbutils.inc.php');
   require_once('utilities.inc.php');
   ```

2. Page header:
   ```php
   pagestart($title, $close);
   // or
   pagestart_nobody($title);
   ```

3. Page content (PHP-generated HTML)

4. Embedded JavaScript:
   ```php
   <script>
   // Global variables
   TID = '<?php echo $textid; ?>';
   // Event handlers
   $(document).ready(function() { ... });
   </script>
   ```

5. Page footer:
   ```php
   pageend();
   ```

### Utility Functions

**Page Rendering** (`utilities.inc.php`):
- `pagestart($title, $close)` - Standard page header with navigation
- `pagestart_nobody($title)` - Minimal page header
- `framesetheader($title)` - Frameset page header
- `pageend()` - Page footer

**HTML Generation:**
- `tohtml($text)` - Escape HTML entities
- `prepare_textdata_js($text)` - Prepare text for JavaScript
- `annotation_to_json($ann)` - Convert annotations to JSON

## Architecture Characteristics

### Strengths

1. **Simplicity**: No build process, direct file serving
2. **Server-Side Rendering**: Fast initial page loads
3. **Persistent Layout**: Framesets maintain layout across navigation
4. **Progressive Enhancement**: Works without JavaScript (basic functionality)
5. **Lightweight**: Minimal dependencies, small footprint

### Limitations

1. **Legacy Technology**: Framesets are deprecated HTML feature
2. **Tight Coupling**: PHP and JavaScript are tightly integrated
3. **Global State**: No modern state management patterns
4. **No Component Architecture**: Monolithic JavaScript files
5. **Mixed Concerns**: Business logic mixed with presentation
6. **Limited Reusability**: Code duplication across pages
7. **Browser Compatibility**: Framesets have limited mobile support

### Modernization Considerations

**Potential Improvements:**
- Replace framesets with CSS Grid/Flexbox layout
- Implement component-based architecture
- Use modern state management (Redux, Vuex, etc.)
- Separate concerns (MVC/MVP pattern)
- Implement build process for asset optimization
- Add TypeScript for type safety
- Consider migrating to SPA framework (React, Vue, Angular)

**Migration Challenges:**
- Extensive cross-frame communication
- Tight PHP-JavaScript integration
- Global state dependencies
- Legacy browser support requirements

## File Structure

```
lwt_html/
├── js/
│   ├── jq_pgm.js          # Main application logic
│   ├── pgm.js             # Additional utilities
│   ├── jquery.js          # jQuery core
│   ├── jquery-ui.min.js   # jQuery UI
│   ├── jquery.jeditable.mini.js
│   ├── jquery.jplayer.min.js
│   ├── jquery.scrollTo.min.js
│   ├── tag-it.js
│   ├── overlib/           # Tooltip library
│   └── sorttable/         # Table sorting
├── css/
│   ├── styles.css         # Main stylesheet
│   ├── jquery-ui.css
│   ├── jquery.tagit.css
│   └── jplayer_skin/      # Audio player styles
├── *.php                  # PHP pages (server-side rendering)
└── empty.htm              # Empty frame placeholder
```

## Summary

The LWT frontend is a traditional multi-page application that uses server-side PHP rendering with jQuery-based client-side interactivity. The frameset architecture provides a persistent multi-pane interface optimized for the reading/learning workflow. While this architecture is functional and lightweight, it represents a legacy approach that could benefit from modernization for better maintainability and user experience.

