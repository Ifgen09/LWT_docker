# Adding Text Input for User Guesses in LWT Test Modes

This document describes how to enhance the L2→L1 and L1→L2 test modes in Learning With Texts (LWT) by adding a text input field, allowing users to type their answer before revealing the correct one.

---

## Overview

In the current LWT test modes, users mentally recall the answer and then reveal it. By adding a text input, users can actively type their guess, which can then be checked against the correct answer for immediate feedback. This approach increases engagement and reinforces learning.

---

## 1. UI Changes

- Add a text input box to the test interface for L2→L1 and L1→L2 modes.
- The user types their answer before clicking “Show Answer” or “Check.”
- After revealing the correct answer, compare the user’s input to the actual answer and provide feedback (e.g., correct/incorrect, show the correct answer).

---

## 2. Implementation Steps

### A. Update Test Rendering (PHP)
- In the files responsible for rendering the test UI (`do_test_test.php` and possibly `do_test_table.php`), locate the code that displays the term and the “Show Answer” button.
- For L2→L1 and L1→L2 modes, insert an HTML `<input type="text">` field (or `<textarea>` for multi-word answers) above or below the “Show Answer” button.

### B. Add JavaScript Logic
- When the user clicks “Show Answer” or “Check,” use JavaScript to:
  - Reveal the correct answer.
  - Optionally, compare the user’s input to the correct answer (case-insensitive, trimmed, etc.).
  - Display feedback (e.g., “Correct!” or “Try again. The correct answer is: ...”).

### C. Optional: Auto-Check
- Add an “Enter” key handler so that pressing Enter in the input field triggers the check.
- Optionally, allow for minor typos or alternate answers (using string similarity or a list of accepted answers).

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

<script>
function checkAnswer() {
  var userInput = document.getElementById('userGuess').value.trim().toLowerCase();
  var correct = "expected answer".toLowerCase(); // Replace with PHP echo of the correct answer
  document.getElementById('theAnswer').textContent = correct;
  document.getElementById('correctAnswer').style.display = 'block';
  if (userInput === correct) {
    document.getElementById('feedback').textContent = "Correct!";
  } else {
    document.getElementById('feedback').textContent = "Try again.";
  }
}
</script>
```

---

## 4. Where to Integrate

- **do_test_test.php**: Main file for individual test questions. Add the input and logic in the section that renders the test question for L2→L1 and L1→L2.
- **do_test_table.php**: If you want this feature in table/list mode, add a similar input for each row.

---

## 5. Accessibility and Usability

- Automatically focus the input field when the question loads.
- Optionally, allow users to skip or reveal the answer if they’re stuck.

---

## 6. Summary of Steps

1. Add a text input field to the test UI for L2→L1 and L1→L2 modes.
2. Add a “Check” button (or use Enter key).
3. On check, compare input to the correct answer and show feedback.
4. Reveal the correct answer for learning reinforcement.

---

For further customization or integration help, see the main documentation or contact the project maintainers. 