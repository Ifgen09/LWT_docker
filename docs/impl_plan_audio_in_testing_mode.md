## Implementation Plan: Audio Playback for Words in Testing Mode (LWT)

This plan describes the steps to add audio playback for words in testing mode, covering schema, backend, frontend, security, DevOps, testing, rollout, and acceptance criteria.

---

### Phase 0 — Finalize key decisions
- **Storage**: Local `media/` via Docker volume (default), configurable to S3/CDN via env.
- **Multiplicity**: Start with single audio per word (`WoAudioURI`), design extensible upload code.
- **Input**: Support file upload and URL; prefer upload in UI.
- **Segmentation**: Support extracting word segments from full MP3 files (Phase 3b, optional).
  - Decision: Store full file + timestamps (Option A) vs. extract immediately (Option B).
  - Default: Option A (preserves source for re-extraction, supports multiple words from same source).
- **Preload**: Use `preload="none"`; lazily create `<audio>` on first play.
- **Fallback**: If no audio, hide control in v1 (no TTS).

### Phase 1 — Schema migration
- Add column:
  ```sql
ALTER TABLE words ADD COLUMN WoAudioURI VARCHAR(1024) DEFAULT NULL;
  ```
- Optional index only if querying by URI (unlikely needed).
- Migration scripts: up (add column), down (drop column). Take DB backup.

### Phase 2 — Storage and serving
- **Local storage**
  - Directory: `media/words/` (Docker volume).
  - Path: `media/words/<language>/<word_id>_<uuid>.<ext>`.
  - Public URL base: `MEDIA_BASE_URL` (e.g., `/media/`).
- **Web server example (nginx)**
  ```nginx
location /media/ {
  alias /var/www/app/media/;
  add_header Cache-Control "public, max-age=31536000, immutable";
  types { audio/mpeg mp3; audio/ogg ogg; audio/wav wav; }
}
  ```
- **Env config**
  - `MEDIA_BASE_PATH=/var/www/app/media`
  - `MEDIA_BASE_URL=/media`
  - `MEDIA_MAX_SIZE_MB=10`
  - `MEDIA_ALLOWED_TYPES=audio/mpeg,audio/ogg,audio/wav`

### Phase 3 — Backend (PHP) changes
- **Touchpoints**: `edit_words.php`, `new_word.php`, `edit_tword.php`, `edit_mword.php`, `do_test_test.php`, `do_test_table.php`.
- **Forms**
  - Add file input `WoAudioFile` and text input `WoAudioURI` (URL).
  - Add "Remove audio" checkbox.
- **Upload handler (shared)**
  - CSRF check; size limit from env.
  - Validate MIME with `finfo_file` and extension allowlist.
  - Sanitize filename; generate UUID; write into `MEDIA_BASE_PATH`.
  - Return app-relative URI (e.g., `/media/words/...`).
- **Save logic**
  - If remove checked: delete local file if inside `MEDIA_BASE_PATH`; set `WoAudioURI=NULL`.
  - Else if file uploaded: run handler; set `WoAudioURI`.
  - Else if URL provided: validate scheme http/https or app-relative; set `WoAudioURI`.
- **Permissions**: Restrict upload/edit to authorized users for the language.
- **Cleanup**: On word delete or replacement, delete old file if local.

### Phase 3b — Audio segmentation from full MP3 files (optional)
- **Use case**: Allow users to upload a full MP3 (e.g., sentence/paragraph) and extract word-level segments.
- **Storage strategy**: Two options:
  - **Option A (recommended)**: Store full MP3 + timestamps; extract segments on-demand or on-save.
    - Add columns: `WoAudioSourceURI VARCHAR(1024)`, `WoAudioStart DECIMAL(10,3)`, `WoAudioEnd DECIMAL(10,3)`.
    - Store full file in `media/sources/<language>/<source_id>_<uuid>.mp3`.
    - Extract segment to `media/words/...` when user saves, or serve via range requests.
  - **Option B**: Extract and store segments immediately; delete source after extraction.
- **UI components**:
  - **Audio upload area**: File input for full MP3 in word edit forms.
  - **Audio player with timeline**:
    - HTML5 `<audio>` with `controls` and custom timeline overlay.
    - Waveform visualization (optional, requires Web Audio API or server-side analysis).
    - Visual markers for start/end selection (draggable handles or click-to-set).
    - Display current selection: "Selected: 2.5s - 4.8s (2.3s duration)".
  - **Controls**:
    - Play/pause, seek, loop selection.
    - "Set start" / "Set end" buttons (or click on timeline).
    - "Play selection" to preview extracted segment.
    - "Extract and save" to process and store segment.
- **JavaScript (frontend)**:
  - Load uploaded MP3 into `<audio>` element.
  - Track selection state: `startTime`, `endTime`.
  - Update timeline markers on user interaction.
  - Preview selected segment: `audio.currentTime = startTime; audio.play(); setTimeout(() => audio.pause(), (endTime - startTime) * 1000)`.
  - On "Extract and save": POST to backend with `sourceFile`, `startTime`, `endTime`, `wordId`.
- **Backend processing (PHP + FFmpeg)**:
  - **Endpoint**: `extract_audio_segment.php` (or method in upload handler).
  - **Input validation**:
    - Verify source file exists and is accessible.
    - Validate `startTime >= 0`, `endTime > startTime`, `duration = endTime - startTime <= MAX_SEGMENT_DURATION` (e.g., 10s).
    - CSRF check.
  - **FFmpeg command**:
    ```bash
    ffmpeg -i /path/to/source.mp3 \
           -ss START_TIME \
           -t DURATION \
           -acodec copy \
           -avoid_negative_ts make_zero \
           /path/to/output.mp3
    ```
    - Use `-acodec copy` for speed (no re-encoding) if format matches.
    - Use `-acodec libmp3lame -b:a 128k` if re-encoding needed.
    - Validate output file exists and has reasonable size.
  - **Error handling**:
    - Check FFmpeg exit code; log stderr on failure.
    - Return JSON: `{success: true, uri: "/media/words/..."}` or `{success: false, error: "..."}`.
  - **Cleanup**: If Option B, delete source file after successful extraction.
- **Dependencies**:
  - FFmpeg installed in Docker container (add to `Dockerfile`).
  - PHP exec permissions (or use `proc_open` with proper escaping).
  - Env var: `ENABLE_AUDIO_SEGMENTATION=true` (feature flag).
- **Alternative (client-side extraction)**:
  - Use Web Audio API + MediaRecorder to extract segment in browser.
  - Pros: No server processing, faster feedback.
  - Cons: Browser compatibility, larger JS bundle, quality may vary.
  - Only viable for short segments (< 30s typically).

### Phase 4 — Testing mode integration
- **Data**: Include `words.WoAudioURI` in queries in `do_test_test.php` and `do_test_table.php`.
- **Render**
  - If URI present, render a play button and a status span.
  - Example:
    ```html
<button class="play-audio" data-audio-src="/media/words/..." aria-label="Play audio for TERM">🔊</button>
<span class="audio-status" aria-live="polite"></span>
    ```
- **JavaScript behavior**
  - On click: lazily create `<audio preload="none">` with `src` from `data-audio-src`.
  - Ensure only one audio plays at a time; pause others on play.
  - Update button state (playing/paused); handle `ended` and `error` events; show status messages.
  - Keyboard: Enter on focused row plays; Space replays last.
- **Mobile**: Require user gesture; no autoplay.

### Phase 5 — Accessibility and i18n
- ARIA labels (e.g., `aria-label="Play audio for <term>"`) and `aria-live="polite"` status.
- Use `aria-pressed` if button is a toggle.
- Localize labels/tooltips; verify RTL placement.

### Phase 6 — Security
- URL validation: allow only http(s) or app-relative; deny `javascript:`/`data:`.
- MIME validation via `finfo`; extension allowlist.
- Basic rate limiting of uploads per user/session.
- If policy requires, integrate antivirus scanning.
- Update CSP to allow media domain in `media-src`.

### Phase 7 — Performance
- Lazy instantiate audio elements; do not preload.
- Long-lived cache headers (immutable UUID filenames).
- If external storage, set ETag and CDN caching.
- Auto-pause other audio to reduce parallel bandwidth.

### Phase 8 — Export/Import (optional)
- Export: include `WoAudioURI` in CSV/TSV.
- Import: accept `WoAudioURI` if present; ignore when absent for backward compatibility.

### Phase 9 — DevOps/Docker
- Add Docker volume for `media/` and file permissions for app user.
- Configure web server to serve `MEDIA_BASE_URL`.
- Add env vars to `.env` and deployment secrets.
- Optional: backup job for `media/` volume.

### Phase 10 — QA and tests
- **Unit (PHP)**: upload validator (size, MIME, ext), URL validator, cleanup on delete/replace.
- **Integration**: form submit flows (upload, URL, remove).
- **E2E**: render play control, play/pause, single-audio-at-a-time, error states, mobile gestures.
- **Cross-browser**: Chrome, Firefox, Safari, Edge.

### Phase 11 — Rollout
- Feature flag `ENABLE_AUDIO=true` to toggle UI.
- Deploy order: (1) DB migration, (2) backend with flag off, (3) web server media, (4) enable flag in staging → prod.
- Monitoring: capture JS playback errors; server logs for upload failures.

### Acceptance criteria (Definition of Done)
- `WoAudioURI` column exists and is nullable.
- Forms support upload, URL input, and removal with validation.
- Test views show play control only when audio exists.
- Clicking play works on desktop/mobile, pauses other audio, and updates UI states.
- Accessibility labels and keyboard support implemented; RTL verified.
- URL and MIME validation enforced; unsafe schemes blocked.
- Media served with correct MIME and cache headers; persisted in Docker volume.
- Unit, integration, and E2E tests passing; basic cross-browser verified.
- Docs updated: `docs/audio_in_testing_mode.md` and admin/storage guide.
- **If Phase 3b enabled**: Audio segmentation UI allows selecting time ranges from full MP3; FFmpeg extraction works; extracted segments play correctly in testing mode.

### Effort estimate
- Backend + schema: 0.5–1 day
- Forms + upload handling: 1–1.5 days
- Testing UI + JS: 1 day
- DevOps/config: 0.5 day
- Tests + QA: 1–1.5 days
- Docs/polish: 0.5 day
- **Phase 3b (Audio segmentation)**: +2–3 days
  - UI components (player, timeline, selection): 1 day
  - FFmpeg integration + backend processing: 0.5–1 day
  - Testing + edge cases: 0.5–1 day
- **Total (without segmentation)**: ~4–5.5 days
- **Total (with segmentation)**: ~6–8.5 days 