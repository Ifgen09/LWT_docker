# Audio Segmentation Processing Time Discussion

This document captures the discussion about processing times for extracting audio segments from full MP3 files, covering both single segment extraction and batch processing scenarios.

---

## Question 1: Single Segment Processing Time

**User:** "so...getting segments - how long this can be for the user ?"

**Answer:**

### Processing Time Estimates

**Server-side FFmpeg extraction:**
- **Fast path** (codec copy, no re-encoding): **< 1 second** for typical word segments (0.5-3 seconds of audio)
- **Re-encoding path**: **1-3 seconds** for short segments
- Scales roughly with segment duration (10 second segment ≈ 2-5 seconds processing)

**Total user wait time (single segment):**
- File upload: ~4 seconds (5MB MP3 on 10 Mbps connection)
- FFmpeg processing: 0.5-3 seconds
- **Total: ~4.5-7 seconds** for a single word segment

**Factors affecting time:**
- Source file size: larger files = longer upload
- Segment duration: longer segments = slightly longer processing
- Network speed: slower connection = longer upload
- Server CPU: busy server = slower FFmpeg execution

**UX considerations:**
- Synchronous processing is acceptable for single segments (< 10 seconds wait)
- Show loading indicator: "Extracting audio segment..."
- For multiple segments from same source: batch extraction is much faster than sequential calls
- **Option A (store source + timestamps)**: Save timestamps immediately, extract later for faster UI response

**Alternative: Client-side extraction**
- Web Audio API: ~5 seconds total (feels faster, no server round-trip)
- Trade-off: browser compatibility and quality may vary

**Bottom line:** Expect about **4-7 seconds** for a single word segment extraction, which is acceptable for a synchronous operation with proper loading feedback.

---

## Question 2: Batch Processing for Hundreds of Words

**User:** "so considering the stage, when user adds new text with hundreds of new words. - how this can be ?"

**Answer:**

### The Problem

If extracting audio for each word sequentially:
- **100 words × 5 seconds = ~8 minutes**
- **500 words × 5 seconds = ~42 minutes**

This is completely unacceptable!

### Solutions for Batch Word Audio Extraction

#### Option 1: Batch FFmpeg Extraction (Recommended)
Extract all segments in one or a few FFmpeg passes:

```bash
# Single FFmpeg command to extract multiple segments
ffmpeg -i source.mp3 \
  -ss 2.5 -t 0.8 -acodec copy word1.mp3 \
  -ss 3.3 -t 0.6 -acodec copy word2.mp3 \
  -ss 3.9 -t 0.7 -acodec copy word3.mp3 \
  ...
```

- **Time:** ~10-30 seconds for 100 segments (vs. 8+ minutes sequential)
- **Requires:** Word timestamps (start/end for each word)

#### Option 2: Deferred Extraction (Fastest UI)
Store timestamps only, extract on-demand:

1. **During text import:** Save `WoAudioSourceURI` + `WoAudioStart` + `WoAudioEnd` (instant)
2. **On first play:** Extract segment if missing, cache it
3. **User experience:** Text import is fast; first playback may take 1-2 seconds per word

#### Option 3: Speech-to-Text Alignment (Automated)
Use alignment tools to automatically get word timestamps:

- **Tools:** Whisper (with word-level timestamps), Gentle, Montreal Forced Aligner
- **Workflow:**
  1. User uploads text + audio
  2. System aligns words to audio timestamps automatically
  3. System extracts all segments in batch
- **Time:** ~30-60 seconds for alignment + ~10-20 seconds for batch extraction

#### Option 4: Manual Timeline Marking (One-time Setup)
User marks word boundaries once in a timeline UI:

1. Show audio player with text synchronized
2. User clicks to mark start/end for each word (or drags)
3. System extracts all segments in batch
- **Time:** ~2-5 minutes manual work + ~10-20 seconds batch extraction

### Recommended Approach

**Hybrid strategy:**

1. **Text import phase:**
   - Upload text + full audio file
   - Parse text and create words (existing flow)
   - If alignment available: auto-generate timestamps
   - Store: `WoAudioSourceURI`, `WoAudioStart`, `WoAudioEnd` (no extraction yet)
   - **Time:** ~1-2 seconds (just database writes)

2. **Background processing (optional):**
   - Queue batch extraction job
   - Extract all segments in parallel/batch
   - Notify user when complete

3. **On-demand extraction (fallback):**
   - If segment missing when word is played, extract it then
   - Cache for future use

### Implementation Considerations

- **Word timestamp source:** Alignment tool, manual marking, or import from external source
- **Batch extraction:** Use FFmpeg filter_complex or multiple parallel processes
- **Progress feedback:** Show "Extracting audio for 100 words... (45/100)" during batch
- **Storage:** Option A (source + timestamps) allows re-extraction if needed

**Bottom line:** Don't extract synchronously word-by-word. Use batch extraction with timestamps, or defer extraction until first play. This keeps text import fast while still providing audio for all words.

---

## Summary

- **Single segment:** ~4-7 seconds (acceptable for synchronous processing)
- **Hundreds of words:** Requires batch processing or deferred extraction
- **Best approach:** Store timestamps during import, extract in batch or on-demand
- **Key insight:** Never do sequential word-by-word extraction for bulk operations

