# Adding Text Input for User Guesses in LWT Test Modes

This document describes how to enhance the L2→L1 and L1→L2 test modes in Learning With Texts (LWT) by adding a text input field, allowing users to type their answer before revealing the correct one. It also describes the progressive hint system that helps guide users toward the answer after incorrect attempts.

---

## Overview

In the current LWT test modes, users mentally recall the answer and then reveal it. With this enhancement, users actively type their guess, receive immediate feedback, and are guided by hints if their answer is incorrect. The correct answer is only shown after a correct attempt, reinforcing learning and recall.

---

## 1. UI Changes

- Add a text input box to the test interface for L2→L1 and L1→L2 modes.
- The user types their answer before clicking “Check” or pressing Enter.
- After each attempt, the user receives feedback and, if incorrect, a hint. The correct answer is only revealed after a correct attempt.

---

## 2. Implementation Steps

### A. Update Test Rendering (PHP)
- In the files responsible for rendering the test UI (`do_test_test.php` and possibly `do_test_table.php`), locate the code that displays the term and the “Show Answer” button.
- For L2→L1 and L1→L2 modes, insert an HTML `<input type="text">` field (or `<textarea>` for multi-word answers) above or below the “Check” button.

### B. Add JavaScript Logic
- When the user clicks “Check” or presses Enter:
  - Compare the user’s input to the correct answer (case-insensitive, trimmed, etc.).
  - If correct, display “Correct!” in green and reveal the correct answer.
  - If incorrect, display “Try again.” in red and show a hint (see below).
  - The correct answer is only shown after a correct attempt.

### C. Progressive Hint System
- After each failed attempt, a progressively more helpful hint is shown:
  1. **First failed attempt:** Show the first letter and the length of the answer.
  2. **Second failed attempt:** Show the first letter and mask the rest (e.g., `c _ _ _ _ _ _` for "correct").
  3. **Third failed attempt:** Reveal the first letter and one random letter, masking the rest.
  4. **Fourth and subsequent attempts:** Show an anagram of the answer.
- Hints are displayed below the feedback message.

---

## 3. Example UI Snippet

```html
<!-- Before showing the answer -->
<label for="userGuess">Your Answer:</label>
<input type="text" id="userGuess" autocomplete="off">
<button onclick="checkAnswer()">Check</button>

<div id="feedback"></div>
<div id="correctAnswer" style="display:none;">
  Correct answer: <span id="theAnswer"></span>
</div>
<div id="hint" style="margin-top:8px; color: #555;"></div>
```

---

## 4. Where to Integrate

- **do_test_test.php**: Main file for individual test questions. Add the input and logic in the section that renders the test question for L2→L1 and L1→L2.
- **do_test_table.php**: If you want this feature in table/list mode, add a similar input for each row.

---

## 5. Accessibility and Usability

- Automatically focus the input field when the question loads.
- Optionally, allow users to skip or reveal the answer if they’re stuck (not implemented by default).

---

## 6. Summary of Steps

1. Add a text input field to the test UI for L2→L1 and L1→L2 modes.
2. Add a “Check” button (or use Enter key).
3. On check, compare input to the correct answer and show feedback.
4. If incorrect, show a progressively more helpful hint.
5. Reveal the correct answer only after a correct attempt for learning reinforcement.

---

For further customization or integration help, see the main documentation or contact the project maintainers. 