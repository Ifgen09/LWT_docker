# Questions: Audio Playback for Words in Testing Mode (LWT)

This document lists clarifying questions to finalize scope, design, and acceptance criteria for adding audio playback in testing mode.

---

### Database and schema
- **Column design**: Should `WoAudioURI` allow more than 255 chars (e.g., 512/1024) for long URLs? Any need for an index?
- **Multiplicity**: Can a word have multiple audios (dialects, slow/normal speed)? If yes, should we use a separate `word_audio` table with `WordID`, `type`, `variant`, `speed`?
- **Migration**: Do we need an up/down migration script? Target MySQL/MariaDB versions?
- **Validation**: Enforce URL scheme (http/https) or allow relative paths only?

### Uploads and storage
- **Storage location**: Local `media/` (container/volume) or external (S3, CDN)?
- **URL shape**: Prefer absolute URLs, app-relative paths, or pre-signed links?
- **Limits**: Max file size per upload and total quota per user/language?
- **Types**: Allowed MIME/types (mp3, ogg, wav)? Any transcoding/conversion required?
- **CORS**: Will audio be cross-origin? If so, what CORS config is required?

### Backend integration
- **Endpoints**: Uploads only via existing forms, or also via an API endpoint?
- **Sanitization**: Validate MIME by inspecting file headers (magic numbers) vs extension only?
- **Access control**: Who can upload/modify audio (roles, ownership rules)?
- **Cleanup**: On term delete or file replacement, should old/orphaned files be deleted?

### Audio segmentation from full MP3 files
- **Workflow**: Should users be able to upload a full MP3 and extract word segments, or only upload pre-segmented files?
- **Storage strategy**: 
  - Store full file + timestamps (allows re-extraction, multiple words from same source)?
  - Or extract segments immediately and delete source (saves space)?
- **UI approach**: 
  - Simple timeline with start/end markers?
  - Waveform visualization (requires additional processing)?
  - Click-to-set vs. drag handles for selection?
- **Processing**: Server-side FFmpeg vs. client-side Web Audio API?
- **Source file management**: 
  - Keep source files indefinitely for re-extraction?
  - Auto-delete after X days or when all segments extracted?
  - Allow manual deletion of source files?
- **Multiple words from same source**: 
  - Can one source file be used for multiple words?
  - How to track which words share a source?
- **Extraction limits**: Max segment duration? Min duration? Max source file size?
- **Schema changes**: If storing timestamps, add `WoAudioSourceURI`, `WoAudioStart`, `WoAudioEnd` columns?

### Testing mode data flow
- **Fetch strategy**: Include `WoAudioURI` in existing queries or lazy-load via a dedicated endpoint?
- **Fallbacks**: If missing audio, show disabled control, hide it, or generate TTS?
- **Preload policy**: Use `preload="none"` or `metadata` to limit bandwidth?

### Frontend UX
- **Play control**: Inline button per word vs shared player component?
- **Concurrency**: Should starting playback for one word pause any others?
- **Keyboard access**: Hotkeys (e.g., space to replay current term), focus order?
- **Visual states**: Loading, playing, paused, failed states and retry affordance?
- **Speed/loop**: Need speed controls (0.75x/1x/1.25x) or repeat N times?

### Accessibility
- **ARIA**: Labels like “Play audio for WORD”; toggle button semantics?
- **Transcripts**: Any requirement to display a transcript or duration?

### Internationalization
- **Per-language behavior**: Different defaults or formats by language?
- **RTL**: Any layout changes for RTL languages?

### Security
- **URL safety**: Disallow `javascript:`/data URIs; allow only http(s) and app-relative?
- **Upload scanning**: Any antivirus/malware scanning step?
- **Rate limiting**: Limit upload attempts to prevent abuse?

### Performance
- **Lazy rendering**: Render or mount audio elements only when visible?
- **Caching**: Cache headers/ETag for static audio? Recommended TTLs?
- **CDN**: Serve audio via CDN in production?

### Compatibility
- **Browser support**: Minimum versions; preferred formats per browser?
- **Mobile**: Ensure tap-to-play; respect autoplay restrictions on iOS/Android.

### LWT-specific integration
- **Touchpoints**: Confirm files to update: `edit_words.php`, `edit_tword.php`, `new_word.php`, `edit_mword.php`, `do_test_test.php`, `do_test_table.php`. Any others (export/import, batch editor)?
- **Export/Import**: Include `WoAudioURI` in CSV/TSV export/import?
- **Backwards compatibility**: Older exports without the column should still import cleanly?

### Observability
- **Telemetry**: Track play clicks, failures, and load times?
- **Error logging**: Where should playback/load errors be logged (client vs server)?

### DevOps and Docker
- **Static serving**: If local storage, will `media/` be served with correct MIME via nginx/Apache?
- **Volumes**: Persist uploads with Docker volumes; path convention?
- **Environment config**: Env vars for base media URL, size/type limits?

### Acceptance criteria
- **Definition of done**: Checklist covering migration, form updates, test UI with working playback, mobile verification, a11y checks, docs updated.
- **Tests**: Unit/e2e tests for upload, render, and playback paths. 