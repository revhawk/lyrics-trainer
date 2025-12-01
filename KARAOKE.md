# Karaoke Mode Documentation

## Overview

Karaoke mode adds timestamped lyrics support to the Lyrics Trainer app, allowing lyrics to automatically synchronize with audio playback. This feature uses the LRC (Lyrics) file format for time-synchronized lyrics.

## Features

- **LRC File Support**: Parse and display timestamped lyrics in `.lrc` format
- **Audio Player Integration**: Optional HTML5 audio player for music playback
- **Automatic Synchronization**: Lyrics automatically advance based on audio timestamps
- **Timestamp Display**: Shows the timestamp for each lyric line (e.g., `[00:12]`)
- **Backward Compatible**: Plain `.txt` files continue to work as before
- **Manual Navigation**: All existing navigation features (Previous/Next, slider, keyboard) still work

## LRC File Format

The LRC format is a simple text format for timestamped lyrics:

```
[mm:ss.xx]Lyric text
```

### Example

```lrc
[00:12.00]First line of the song
[00:15.50]Second line of the song
[00:19.00]Third line of the song
[00:23.50]Fourth line of the song
```

### Format Details

- **Timestamps**: `[mm:ss.xx]` where:
  - `mm` = minutes (2 digits)
  - `ss` = seconds (2 digits)
  - `xx` = centiseconds (2 digits, hundredths of a second)
- **Lyrics**: Any text following the timestamp
- **Sorting**: Lines are automatically sorted by timestamp
- **Blank Lines**: Empty lines are ignored

## How to Use

### Option 1: Upload LRC File

1. Click "Upload Lyrics (.txt/.lrc)" button
2. Select an `.lrc` file from your computer
3. The app will automatically detect it's an LRC file and enable karaoke mode
4. You'll see timestamps displayed next to the line counter

### Option 2: Use Playlist

If LRC files are in the `public/lyrics/` folder and listed in `public/lyrics-list.json`:
1. Click the song name in the playlist
2. Karaoke mode activates automatically for `.lrc` files

### Option 3: Add Audio

To sync lyrics with music:
1. Load an LRC file (using either method above)
2. Click "Upload Audio (optional)" button
3. Select an audio file (MP3, WAV, etc.)
4. Press play on the audio player
5. Lyrics will automatically advance based on timestamps

## Creating LRC Files

### Manual Creation

Create a text file with `.lrc` extension:

```lrc
[00:00.00]Happy Birthday to you
[00:03.50]Happy Birthday to you
[00:07.00]Happy Birthday dear friend
[00:11.00]Happy Birthday to you
```

### Using LRC Editors

Several free tools can help create LRC files:
- **LRC Editor** (Windows)
- **Lyrics Master** (Mac)
- **MiniLyrics Editor** (Cross-platform)
- Online LRC generators

### Finding Existing LRC Files

Many songs already have LRC files available:
- Search for "[song name] lrc file"
- Check lyrics websites that offer timestamped lyrics
- Download from karaoke websites

## Technical Implementation

### TypeScript Interfaces

```typescript
interface LyricLine {
    text: string;
    timestamp?: number; // milliseconds
}

interface TrainerState {
    idx: number;
    delay: number;
    isKaraokeMode: boolean;
}
```

### LRC Parser

The `parseLRC()` function:
- Uses regex to match `[mm:ss.xx]` patterns
- Converts timestamps to milliseconds
- Sorts lines by timestamp
- Returns array of `LyricLine` objects

### Audio Synchronization

The `syncLyricsWithAudio()` function:
- Listens to audio player's `timeupdate` event
- Compares current audio time with lyric timestamps
- Automatically advances to the appropriate line
- Updates every ~250ms during playback

### Auto-Detection

Files are detected as LRC format if:
1. File extension is `.lrc`, OR
2. Content contains the pattern `[dd:dd.dd]`

## File Structure

```
lyrics-trainer/
├── public/
│   ├── lyrics/
│   │   ├── this_is_the_moment.txt  # Plain text
│   │   ├── bohemian_rhapsody.txt   # Plain text
│   │   └── happy_birthday.lrc      # Timestamped
│   ├── lyrics-list.json            # Playlist manifest
│   └── index.html                  # Updated UI
├── src/
│   └── script.ts                   # Core logic with LRC support
└── KARAOKE.md                      # This file
```

## Adding Songs to Playlist

To add an LRC file to the playlist:

1. Copy the `.lrc` file to `public/lyrics/`
2. Edit `public/lyrics-list.json`:
   ```json
   [
     "this_is_the_moment.txt",
     "bohemian_rhapsody.txt",
     "happy_birthday.lrc"
   ]
   ```
3. Refresh the app

## Keyboard Shortcuts

All existing shortcuts work in karaoke mode:
- `←` Previous line
- `→` Next line
- `Space` Play/Pause (manual timer, not audio)
- `Home` First line
- `End` Last line

## User Interface Changes

### New Buttons
- **Upload Lyrics (.txt/.lrc)**: Now accepts both formats
- **Upload Audio (optional)**: New button for audio files

### New Elements
- **Audio Player**: Appears when audio is loaded
- **Timestamp Display**: Shows `[mm:ss]` in counter for LRC files

### Behavior Changes
- **Auto-advance**: Disabled when audio plays in karaoke mode
- **Manual timer**: Still available for non-LRC or non-audio usage

## Browser Compatibility

- **Audio Player**: HTML5 audio (all modern browsers)
- **File Upload**: FileReader API (all modern browsers)
- **localStorage**: For persistence (all modern browsers)

## Limitations

- Maximum timestamp: 99:59.99 (99 minutes, 59.99 seconds)
- Audio formats: Depends on browser support (MP3, WAV widely supported)
- File size: Large audio files may be slow to load
- No streaming: Audio must be fully loaded before playback

## Future Enhancements

Potential improvements:
- [ ] Visual progress bar synced with audio
- [ ] Click lyrics to jump to that timestamp in audio
- [ ] Edit mode to create/adjust timestamps
- [ ] Export modified LRC files
- [ ] Support for LRC metadata tags (artist, title, etc.)
- [ ] Multiple language support (dual-language LRC)
- [ ] Karaoke-style word highlighting (requires enhanced LRC format)

## Troubleshooting

### Timestamps not syncing
- Check that audio file is loaded
- Verify LRC timestamps are in correct format `[mm:ss.xx]`
- Ensure audio player is playing (not paused)

### LRC file not recognized
- Check file extension is `.lrc`
- Verify timestamps follow the pattern `[dd:dd.dd]`
- Check file is UTF-8 encoded

### Audio not playing
- Verify browser supports the audio format
- Check browser console for errors
- Try a different audio file format (MP3 is most compatible)

## Example Files

### Sample LRC File

See `lyrics/happy_birthday.lrc` for a complete example:

```lrc
[00:00.00]Happy Birthday to you
[00:03.50]Happy Birthday to you
[00:07.00]Happy Birthday dear friend
[00:11.00]Happy Birthday to you
```

## Contributing

To add more karaoke features:
1. Edit `src/script.ts`
2. Run `npm run build` to compile
3. Test with `npm run serve`
4. Update this documentation

## License

Same as main project (MIT License)
