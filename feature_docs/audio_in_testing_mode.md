# Adding Audio Playback for Words in Testing Mode (LWT)

This document describes how to add the ability to play audio for a word during testing mode in the Learning With Texts (LWT) application.

---

## 1. Audio Data Storage
- **Add a new column to the `words` table** (e.g., `WoAudioURI`) to store the audio file path or URL for each word.
- Example SQL:
  ```sql
  ALTER TABLE words ADD COLUMN WoAudioURI VARCHAR(255) DEFAULT NULL;
  ```

## 2. Audio Input in the UI
- **Update the term creation/edit forms** (e.g., `edit_words.php`, `edit_tword.php`, `new_word.php`, `edit_mword.php`) to allow users to upload or specify an audio file/URL for each word.
- **Save the audio file/URL** to the new database column.

## 3. Expose Audio in Testing Mode
- **In the testing mode code** (`do_test_test.php` and `do_test_table.php`):
  - Fetch the audio URI along with other word data.
  - When rendering the test UI, add an HTML `<audio>` element or a play button for the word if an audio URI is present.

## 4. Frontend: Play Audio
- **Add a play button or icon** next to the word in the test UI.
- **On click**, use JavaScript to play the audio using the HTML5 `<audio>` API.
- Example HTML:
  ```html
  <button onclick="document.getElementById('audioWORDID').play()">🔊</button>
  <audio id="audioWORDID" src="AUDIO_URL"></audio>
  ```
  Replace `WORDID` and `AUDIO_URL` with the actual word ID and audio URI.

## 5. Optional: Audio File Management
- If allowing file uploads, handle file storage (e.g., in a `media/` directory) and ensure uploaded files are accessible via URL.

## 6. Security and Usability
- Sanitize and validate audio URLs/paths.
- Restrict allowed file types to audio formats (e.g., mp3, ogg, wav).
- Handle missing or broken audio gracefully in the UI.

---

## Summary
- **Database:** Add `WoAudioURI` to `words`.
- **Forms:** Update add/edit forms to accept audio.
- **Testing UI:** Fetch and display audio play button in test views.
- **Frontend:** Use HTML5 `<audio>` for playback.

---

*For further details or code examples, see the main documentation or contact the project maintainers.* 