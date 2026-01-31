# Testing Mode Technical Documentation

This document provides a comprehensive technical reference for the LWT testing system, covering architecture, algorithms, file responsibilities, and implementation details.

---

## 1. Architecture Overview

### 1.1 Frameset Structure

The test mode uses a **4-frame layout** for desktop and a **4-div layout with iframes** for mobile:

```
+------------------+------------------+
|    Frame 'h'     |    Frame 'ro'    |
|   (header)       |  (right-open)    |
|  Test buttons,   |  Word details,   |
|  progress stats  |  edit panel      |
+------------------+------------------+
|    Frame 'l'     |    Frame 'ru'    |
|    (left)        |  (right-upper)   |
|  Test question   |  (usually empty) |
|  display area    |                  |
+------------------+------------------+
```

**Frame Dimensions** (configurable via settings):
- `set-test-h-frameheight`: Header height in pixels (default: 140)
- `set-test-l-framewidth-percent`: Left column width percentage (default: 50)
- `set-test-r-frameheight-percent`: Right column height split percentage (default: 50)

### 1.2 File Responsibilities

| File | Purpose |
|------|---------|
| `do_test.php` | Main entry point; creates frameset/iframe structure |
| `do_test_header.php` | Header frame; shows test type buttons, initializes session |
| `do_test_test.php` | Question display; selects word, renders test UI |
| `do_test_table.php` | Table mode; displays all terms in sortable list |
| `set_test_status.php` | AJAX handler; updates word status after answer |
| `show_word.php` | Word detail panel; shown in 'ro' frame |
| `edit_tword.php` | Word editor; shown in 'ro' frame when editing |
| `js/jq_pgm.js` | JavaScript; keyboard handlers, click events |
| `js/pgm.js` | JavaScript; overlib popup integration |

---

## 2. Test Flow

### 2.1 Initialization Sequence

```
1. User navigates to do_test.php?lang=X or ?text=X or ?selection=1
2. do_test.php creates frameset
3. do_test_header.php loads in frame 'h':
   - Builds SQL query ($testsql) based on parameters
   - Counts total due terms (WoTodayScore < 0)
   - Initializes session variables:
     $_SESSION['teststart'] = time() + 2
     $_SESSION['testcorrect'] = 0
     $_SESSION['testwrong'] = 0
     $_SESSION['testtotal'] = $totalcountdue
   - Renders 6 test type buttons
4. User clicks test type button
5. do_test_test.php loads in frame 'l'
```

### 2.2 Entry Points

| Parameter | Source | SQL Query |
|-----------|--------|-----------|
| `lang=[id]` | Language page | `words WHERE WoLgID = [id]` |
| `text=[id]` | Text page | `words, textitems WHERE TiLgID = WoLgID AND TiTextLC = WoTextLC AND TiTxID = [id]` |
| `selection=1` | Custom selection | Uses `$_SESSION['testsql']` |

### 2.3 Test Types

| Type | Button | Display | User Recalls |
|------|--------|---------|--------------|
| 1 | `..[L2]..` | Foreign word in sentence | Translation |
| 2 | `..[L1]..` | Translation in sentence | Foreign word |
| 3 | `..[••]..` | Cloze: masked word in sentence | Full word |
| 4 | `[L2]` | Foreign word alone | Translation |
| 5 | `[L1]` | Translation alone | Foreign word |
| Table | `Table` | All terms in sortable table | Self-paced |

**Internal mapping**: Types 4 and 5 are converted to types 1 and 2 with `$nosent = 1` flag.

---

## 3. Spaced Repetition Algorithm

### 3.1 Score Formula

The system uses a custom spaced repetition formula to determine when words are due for review.

**PHP Function** (`utilities.inc.php:2301`):

```php
function getsqlscoreformula($method) {
    // $method = 2 (today), $method = 3 (tomorrow)
    // Formula: {{{2.4^{Status}+Status-Days-1} over Status -2.4} over 0.14325248}
}
```

**SQL Formula (Today's Score)**:
```sql
CASE WHEN WoStatus > 5 THEN 100
ELSE (((POWER(2.4, WoStatus) + WoStatus - DATEDIFF(NOW(), WoStatusChanged) - 1)
       / WoStatus - 2.4) / 0.14325248)
END
```

**SQL Formula (Tomorrow's Score)**:
```sql
CASE WHEN WoStatus > 5 THEN 100
ELSE (((POWER(2.4, WoStatus) + WoStatus - DATEDIFF(NOW(), WoStatusChanged) - 2)
       / WoStatus - 2.4) / 0.14325248)
END
```

### 3.2 Formula Explanation

| Component | Purpose |
|-----------|---------|
| `2.4^Status` | Exponential growth: higher status = longer intervals |
| `+ Status` | Linear adjustment for status level |
| `- Days` | Reduces score as days pass |
| `- 1` (or `-2`) | Offset for today vs tomorrow |
| `/ Status` | Normalize by status level |
| `- 2.4` | Baseline adjustment |
| `/ 0.14325248` | Scaling constant |

### 3.3 Score Interpretation

| Score | Meaning |
|-------|---------|
| `< 0` | **Due for testing today** |
| `>= 0` | Not yet due |
| `100` | Ignored (98) or Well Known (99) - excluded |

### 3.4 Review Intervals

Approximate days until next review based on status:

| Status | Approximate Interval |
|--------|---------------------|
| 1 | ~1 day |
| 2 | ~2-3 days |
| 3 | ~6-7 days |
| 4 | ~14-17 days |
| 5 | ~35-40 days |

---

## 4. Word Selection Algorithm

### 4.1 Selection Query

Located in `do_test_test.php:122`:

```sql
SELECT DISTINCT WoID, WoText, WoTextLC, WoTranslation, WoRomanization,
       WoSentence, WoStatus,
       DATEDIFF(NOW(), WoStatusChanged) AS Days,
       WoTodayScore AS Score
FROM [testsql]
WHERE WoStatus BETWEEN 1 AND 5
  AND WoTranslation != ''
  AND WoTranslation != '*'
  AND WoTodayScore < 0
  [AND WoRandom > RAND()]  -- Pass 1 only
ORDER BY WoTodayScore, WoRandom
LIMIT 1
```

### 4.2 Two-Pass Selection

The system uses a **two-pass approach** for randomization:

1. **Pass 1**: Include `AND WoRandom > RAND()` - attempts to find a random word
2. **Pass 2**: Remove the random condition - selects any due word

This ensures:
- Variety in word selection (pass 1)
- No word is skipped if few are due (pass 2)

### 4.3 Sentence Selection (Types 1-3)

For tests with sentence context:

```php
while ($pass < 3) {
    // 1. Find random sentence containing the word
    SELECT DISTINCT SeID FROM sentences, textitems
    WHERE TiTextLC = [wordlc] AND SeID = TiSeID AND SeLgID = [lang]
    ORDER BY RAND() LIMIT 1

    // 2. Check for unknown words in sentence
    if (AreUnknownWordsInSentence($seid)) {
        // Exclude this sentence, try again
        $sentexcl = ' AND SeID != ' . $seid;
    } else {
        // Use this sentence
        $sent = getSentence($seid, $wordlc, $sentenceCount);
    }
}

// 3. Fallback: use word's own sentence if no clean sentence found
if ($num == 0 && $notvalid) {
    $sent = '{' . $word . '}';
}
```

**AreUnknownWordsInSentence()**: Returns `true` if the sentence contains any words not in the vocabulary (to avoid confusing context).

---

## 5. Status System

### 5.1 Status Codes

| Code | Name | Description | Testing |
|------|------|-------------|---------|
| 1 | Learning (1) | Just started | Active |
| 2 | Learning (2) | Some familiarity | Active |
| 3 | Learning (3) | Moderate familiarity | Active |
| 4 | Learning (4) | Good familiarity | Active |
| 5 | Learned | Mastered | Active |
| 98 | Ignored | Common words, skip | Excluded |
| 99 | Well Known | Already fluent | Excluded |

### 5.2 Status Update Process

When user answers (`set_test_status.php`):

```php
// 1. Get old status and score
$oldstatus = get_first_value("SELECT WoStatus FROM words WHERE WoID = $wid");
$oldscore = get_first_value("SELECT greatest(0, round(WoTodayScore,0)) FROM words WHERE WoID = $wid");

// 2. Calculate new status
if ($stchange == '') {
    // Direct status set (1-5, 98, 99)
    $status = $status + 0;
} else {
    // Increment/decrement (+1/-1)
    $status = $oldstatus + $stchange;
    if ($status < 1) $status = 1;
    if ($status > 5) $status = 5;
}

// 3. Update database
UPDATE words SET
    WoStatus = $status,
    WoStatusChanged = NOW(),
    WoTodayScore = [recalculated],
    WoTomorrowScore = [recalculated],
    WoRandom = RAND()
WHERE WoID = $wid;

// 4. Update session counters
if ($notyettested > 0) {
    if ($stchange >= 0) $_SESSION['testcorrect']++;
    else $_SESSION['testwrong']++;
}
```

### 5.3 UI Update (JavaScript)

After status change, the frame 'l' word element is updated:

```javascript
var context = window.parent.frames['l'].document;
$('.word' + wid, context)
    .removeClass('todo todosty')
    .addClass('done' + (stchange >= 0 ? 'ok' : 'wrong') + 'sty')
    .attr('data_status', status)
    .attr('data_todo', '0');

// Reload test frame after configured delay
setTimeout('window.parent.frames["l"].location.reload();', waittime);
```

---

## 6. Keyboard Shortcuts

### 6.1 Test Mode Shortcuts

Defined in `js/jq_pgm.js:275-328`:

| Key | Code | Action | Requires OPENED |
|-----|------|--------|-----------------|
| Space | 32 | Reveal answer, show word popup | No |
| Up Arrow | 38 | Increase status by 1 | Yes |
| Down Arrow | 40 | Decrease status by 1 | Yes |
| Escape | 27 | Keep current status (no change) | Yes |
| 1-5 | 49-53 / 97-101 | Set status to 1-5 | Yes |
| I | 73 | Set status to 98 (Ignored) | Yes |
| W | 87 | Set status to 99 (Well Known) | Yes |
| E | 69 | Edit word | Yes |

### 6.2 Input Field Protection

When user is typing in the text input field for guessing:

```javascript
function keydown_event_do_test_test(e) {
    // Don't intercept keyboard shortcuts if user is typing in an input field
    var target = e.target || e.srcElement;
    if (target && (target.tagName === 'INPUT' || target.tagName === 'TEXTAREA')) {
        return true;  // Allow normal typing
    }
    // ... handle shortcuts
}
```

---

## 7. Session State Management

### 7.1 Session Variables

| Variable | Purpose | Set In |
|----------|---------|--------|
| `$_SESSION['testsql']` | Custom SQL for selection mode | Various pages |
| `$_SESSION['teststart']` | Test start timestamp (+2 sec buffer) | `do_test_header.php:94` |
| `$_SESSION['testcorrect']` | Count of correct answers | `set_test_status.php:89` |
| `$_SESSION['testwrong']` | Count of wrong answers | `set_test_status.php:91` |
| `$_SESSION['testtotal']` | Total terms due at start | `do_test_header.php:97` |

### 7.2 Progress Calculation

```php
$notyettested = $totaltests - $correct - $wrong;

// Progress bar percentages
$l_notyet = round(($notyettested / $totaltests) * 100, 0);
$l_wrong = round(($wrong / $totaltests) * 100, 0);
$l_correct = round(($correct / $totaltests) * 100, 0);
```

---

## 8. Text Input Enhancement

### 8.1 UI Components

For test types 1 and 2, an optional text input is rendered:

```html
<label for="userGuess">Your Answer:</label>
<input type="text" id="userGuess" autocomplete="off">
<button type="button" onclick="checkAnswer()">Check</button>
<div id="feedback"></div>
<div id="correctAnswer" style="display:none;">
    Correct answer: <span id="theAnswer"></span>
</div>
<div id="hint"></div>
```

### 8.2 Progressive Hint System

```javascript
function getHint(answer) {
    var clean = answer.replace(/^[\[]|[\]]$/g, '').trim();

    if (failedAttempts === 1) {
        // "Starts with X and has Y letters"
        return 'Hint: The answer starts with ' + clean.charAt(0) +
               ' and has ' + clean.length + ' letters.';
    }
    else if (failedAttempts === 2) {
        // "X _ _ _ _" (first letter + underscores)
        return 'Hint: ' + clean.charAt(0) + ' ' +
               Array(clean.length).join('_ ').trim();
    }
    else if (failedAttempts === 3) {
        // First letter + one random letter revealed
        var idx = Math.floor(Math.random() * clean.length);
        var masked = clean.split('').map(function(c, i) {
            return (i === 0 || i === idx ? c : '_');
        }).join(' ');
        return 'Hint: ' + masked;
    }
    else if (failedAttempts >= 4) {
        // Anagram
        var arr = clean.split('');
        // Fisher-Yates shuffle
        for (let i = arr.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [arr[i], arr[j]] = [arr[j], arr[i]];
        }
        return 'Hint: Anagram - ' + arr.join('');
    }
}
```

---

## 9. Database Schema (Testing-Related)

### 9.1 Words Table Fields

| Field | Type | Purpose |
|-------|------|---------|
| `WoID` | INT | Primary key |
| `WoLgID` | INT | Language ID |
| `WoText` | VARCHAR | Word text |
| `WoTextLC` | VARCHAR | Lowercase for matching |
| `WoTranslation` | TEXT | Translation |
| `WoRomanization` | VARCHAR | Romanization |
| `WoSentence` | TEXT | Example sentence with `{word}` markers |
| `WoStatus` | TINYINT | Status code (1-5, 98, 99) |
| `WoStatusChanged` | DATETIME | Last status change timestamp |
| `WoTodayScore` | DOUBLE | Today's spaced repetition score |
| `WoTomorrowScore` | DOUBLE | Tomorrow's projected score |
| `WoRandom` | DOUBLE | Random value for selection variety |

### 9.2 Related Tables

| Table | Purpose |
|-------|---------|
| `sentences` | Stores sentences parsed from texts |
| `textitems` | Maps words to their occurrences in texts/sentences |
| `languages` | Language configuration (dictionaries, parsing rules) |

---

## 10. Configuration Settings

### 10.1 Test-Related Settings

| Setting Key | Default | Purpose |
|-------------|---------|---------|
| `set-test-h-frameheight` | 140 | Header frame height (px) |
| `set-test-l-framewidth-percent` | 50 | Left frame width (%) |
| `set-test-r-frameheight-percent` | 50 | Right frame height (%) |
| `set-test-main-frame-waiting-time` | 0 | Delay before test reload (ms) |
| `set-test-edit-frame-waiting-time` | 0 | Delay before edit panel clears (ms) |
| `set-test-sentence-count` | varies | Number of sentences to fetch |
| `set-mobile-display-mode` | 0 | Mobile detection (0=auto, 1=desktop, 2=mobile) |

### 10.2 Table Test Settings

| Setting Key | Purpose |
|-------------|---------|
| `currenttabletestsetting1` | Edit column visibility |
| `currenttabletestsetting2` | Status column visibility |
| `currenttabletestsetting3` | Term text visibility (color toggle) |
| `currenttabletestsetting4` | Translation visibility |
| `currenttabletestsetting5` | Romanization visibility |
| `currenttabletestsetting6` | Sentence visibility |

---

## 11. CSS Classes

### 11.1 Word Status Classes

| Class | Purpose |
|-------|---------|
| `todo` | Word not yet tested |
| `todosty` | Styling for untested word |
| `doneoksty` | Styling for correctly answered |
| `donewrongsty` | Styling for incorrectly answered |
| `word` | Base class for clickable words |
| `wsty` | Word styling |
| `word[ID]` | Word-specific class for targeting |

### 11.2 Data Attributes

| Attribute | Purpose |
|-----------|---------|
| `data_wid` | Word ID |
| `data_text` | Word text |
| `data_trans` | Translation |
| `data_rom` | Romanization |
| `data_sent` | Clean sentence |
| `data_status` | Current status |
| `data_todo` | 1 = not tested, 0 = tested |

---

## 12. Error Handling

### 12.1 Common Error Conditions

| Condition | Handling |
|-----------|----------|
| Multiple languages in selection | Error message displayed |
| No words due | "Nothing to test here!" with tomorrow's count |
| No sentence found | Falls back to word's own sentence |
| Sentence has unknown words | Excluded, tries another sentence |

### 12.2 Fallback Behavior

```php
// If no valid sentence found, use word wrapped in braces
if ($num == 0) {
    if ($notvalid) $sent = '{' . $word . '}';
}
```

---

## 13. Related Documentation

- `product_features.md` - User-facing feature descriptions
- `testing_mode_text_input.md` - Text input enhancement spec
- `audio_in_testing_mode.md` - Audio playback feature spec
- `impl_plan_audio_in_testing_mode.md` - Audio implementation plan
- `architecture.md` - Overall system architecture
