# Lyrics Trainer Architecture

This document describes the architecture and key flows of the Lyrics Trainer application.

## Table of Contents
- [Overall Architecture](#overall-architecture)
- [State Management](#state-management)
- [File Upload Flow](#file-upload-flow)
- [Karaoke Mode & Audio Sync](#karaoke-mode--audio-sync)
- [Theme Management](#theme-management)
- [Data Structures](#data-structures)

## Overall Architecture

```mermaid
graph TB
    subgraph "User Interface"
        HTML[index.html]
        CSS[style.css]
        UI[UI Components]
    end

    subgraph "Application Core"
        TS[script.ts]
        State[TrainerState]
        Lyrics[LyricLine Array]
    end

    subgraph "Storage Layer"
        LS[localStorage]
        SW[Service Worker]
    end

    subgraph "External Resources"
        Files[Lyrics Files]
        Audio[Audio Files]
    end

    HTML --> TS
    CSS --> UI
    UI --> TS
    TS --> State
    TS --> Lyrics
    TS <--> LS
    TS --> SW
    Files --> TS
    Audio --> TS

    style TS fill:#4a9eff
    style State fill:#ffa94d
    style LS fill:#51cf66
```

## State Management

The application uses a centralized state object with localStorage persistence:

```mermaid
graph LR
    subgraph "TrainerState"
        IDX[idx: current line]
        DELAY[delay: ms between lines]
        KARAOKE[isKaraokeMode: boolean]
    end

    subgraph "localStorage Keys"
        STATE[lyricsTrainerState]
        THEME[lyricsTrainerTheme]
        CUSTOM[customLyrics]
        SOURCE[lyricsSource]
        PATH[lyricsPath]
        LASTFILE[lastLyricsFile]
    end

    IDX --> STATE
    DELAY --> STATE
    KARAOKE --> STATE

    STATE --> LS[(localStorage)]
    THEME --> LS
    CUSTOM --> LS
    SOURCE --> LS
    PATH --> LS
    LASTFILE --> LS

    LS --> |loadState| App[Application]
    App --> |saveState| LS

    style STATE fill:#4a9eff
    style LS fill:#51cf66
```

## File Upload Flow

### Plain Text and LRC File Processing

```mermaid
flowchart TD
    Start([User Uploads File]) --> CheckType{File Type?}

    CheckType -->|.lrc file| ParseLRC[Parse LRC Format]
    CheckType -->|.txt file| ParsePlain[Parse Plain Text]

    ParseLRC --> ExtractTimestamps[Extract Timestamps<br/>mm:ss.xx format]
    ExtractTimestamps --> Sort[Sort by Timestamp]
    Sort --> EnableKaraoke[Enable Karaoke Mode]

    ParsePlain --> SplitLines[Split by Newlines]
    SplitLines --> FilterEmpty[Filter Empty Lines]
    FilterEmpty --> DisableKaraoke[Disable Karaoke Mode]

    EnableKaraoke --> Validate{Lines > 0?}
    DisableKaraoke --> Validate

    Validate -->|Yes| SaveToLS[Save to localStorage]
    Validate -->|No| Error[Show Error]

    SaveToLS --> UpdateUI[Update UI]
    UpdateUI --> Render[Render First Line]

    Error --> End([End])
    Render --> End

    style ParseLRC fill:#ffa94d
    style EnableKaraoke fill:#51cf66
    style Error fill:#ff6b6b
```

### File Source Management

```mermaid
stateDiagram-v2
    [*] --> CheckSource: App Init

    CheckSource --> CustomLyrics: lyricsSource = 'custom'
    CheckSource --> PlaylistLyrics: lyricsSource = 'playlist'
    CheckSource --> DefaultLyrics: No source or 'default'

    CustomLyrics --> LoadFromStorage: Load from localStorage
    PlaylistLyrics --> FetchFromPath: Fetch from saved path
    DefaultLyrics --> FetchDefault: Fetch default file

    LoadFromStorage --> Render: Display
    FetchFromPath --> Render
    FetchDefault --> Render

    Render --> [*]

    UserUpload --> ProcessFile: Upload .txt/.lrc
    ProcessFile --> SaveCustom: Save to localStorage
    SaveCustom --> Render

    ResetButton --> ClearCustom: Clear localStorage
    ClearCustom --> DefaultLyrics
```

## Karaoke Mode & Audio Sync

```mermaid
sequenceDiagram
    participant User
    participant UI
    participant App
    participant Audio as Audio Player
    participant Lyrics as Lyrics Array

    User->>UI: Upload .lrc file
    UI->>App: Process file
    App->>App: Parse timestamps
    App->>App: Set isKaraokeMode = true
    App->>UI: Update display

    User->>UI: Upload audio file
    UI->>App: Create audio URL
    App->>Audio: Set audio source
    App->>UI: Show audio player

    User->>Audio: Play audio

    loop Every timeupdate event
        Audio->>App: Current time (ms)
        App->>Lyrics: Find matching line
        Lyrics-->>App: Return line index
        App->>App: Update state.idx
        App->>UI: Render new line
    end

    User->>Audio: Pause audio
    Audio->>App: Stop sync

    Note over App,UI: Manual controls still work<br/>during karaoke mode
```

### Audio Synchronization Algorithm

```mermaid
flowchart TD
    Start([Audio timeupdate event]) --> CheckMode{Karaoke Mode<br/>Enabled?}

    CheckMode -->|No| End([End])
    CheckMode -->|Yes| GetTime[Get currentTime from audio<br/>Convert to milliseconds]

    GetTime --> InitSearch[newIdx = 0<br/>Start search from beginning]

    InitSearch --> Loop{For each<br/>lyric line}

    Loop -->|More lines| CheckTimestamp{Line timestamp ≤<br/>currentTime?}

    CheckTimestamp -->|Yes| UpdateIdx[newIdx = current index]
    CheckTimestamp -->|No| ExitLoop[Break loop]

    UpdateIdx --> Loop

    ExitLoop --> Compare{newIdx ≠<br/>state.idx?}

    Compare -->|Yes| Update[state.idx = newIdx<br/>render]
    Compare -->|No| End

    Update --> End

    style CheckMode fill:#ffa94d
    style Update fill:#51cf66
```

## Theme Management

```mermaid
flowchart TD
    Start([App Init]) --> LoadTheme[Load theme from localStorage]

    LoadTheme --> CheckSaved{Saved<br/>Theme?}

    CheckSaved -->|'dark'| SetDark[Add dark-theme class<br/>Show ☀️ icon]
    CheckSaved -->|'light'| SetLight[Add light-theme class<br/>Show 🌙 icon]
    CheckSaved -->|null| CheckSystem{System<br/>Preference?}

    CheckSystem -->|prefers-dark| ShowSun[Show ☀️ icon]
    CheckSystem -->|prefers-light| ShowMoon[Show 🌙 icon]

    SetDark --> Ready([Theme Ready])
    SetLight --> Ready
    ShowSun --> Ready
    ShowMoon --> Ready

    Ready --> UserClick[User clicks theme toggle]

    UserClick --> CheckCurrent{Current<br/>Theme?}

    CheckCurrent -->|Dark| SwitchLight[Remove dark-theme<br/>Add light-theme<br/>Save 'light']
    CheckCurrent -->|Light| SwitchDark[Remove light-theme<br/>Add dark-theme<br/>Save 'dark']

    SwitchLight --> Ready
    SwitchDark --> Ready

    style SetDark fill:#4a9eff
    style SetLight fill:#ffa94d
    style CheckSystem fill:#51cf66
```

## Data Structures

### LyricLine Interface

```typescript
interface LyricLine {
    text: string;           // The lyric text to display
    timestamp?: number;     // Optional timestamp in milliseconds
}
```

**Usage:**
- Plain text files: Only `text` is populated
- LRC files: Both `text` and `timestamp` are populated
- Timestamps enable karaoke mode with audio synchronization

### TrainerState Interface

```typescript
interface TrainerState {
    idx: number;              // Current line index (0-based)
    delay: number;            // Auto-advance delay in milliseconds
    isKaraokeMode: boolean;   // Whether karaoke mode is active
}
```

**State Persistence:**
- Saved to localStorage on every render
- Loaded on app initialization
- Remembers position between sessions

### localStorage Schema

```mermaid
classDiagram
    class localStorage {
        +lyricsTrainerState: JSON string
        +lyricsTrainerTheme: 'dark' | 'light' | null
        +customLyrics: JSON array
        +lyricsSource: 'custom' | 'playlist' | 'default'
        +lyricsPath: string
        +lastLyricsFile: string
        +hasAudio: 'true' | null
    }

    class TrainerState {
        +idx: number
        +delay: number
        +isKaraokeMode: boolean
    }

    class LyricLine {
        +text: string
        +timestamp: number | undefined
    }

    localStorage --> TrainerState: stores as JSON
    localStorage --> LyricLine: stores array as JSON
```

## Key Components

### DOM Elements (`els` object)

All interactive elements are cached in the `els` object:
- **Display**: `box`, `counter`
- **Controls**: `prev`, `next`, `playPause`, `seek`
- **Settings**: `delayRange`, `delayLabel`, `themeToggle`
- **File handling**: `uploadBtn`, `fileInput`, `resetBtn`
- **Audio**: `audioPlayer`, `uploadAudioBtn`, `audioInput`
- **State**: `loading`, `error`, `content`

### Event Flow

```mermaid
graph LR
    subgraph "User Actions"
        KB[Keyboard]
        Touch[Touch/Swipe]
        Click[Button Clicks]
    end

    subgraph "Event Handlers"
        Keys[Keyboard Handler]
        Swipe[Touch Handler]
        Buttons[Click Handler]
    end

    subgraph "Core Functions"
        Advance[advance]
        Render[render]
        Timer[startTimer/stopTimer]
    end

    KB --> Keys
    Touch --> Swipe
    Click --> Buttons

    Keys --> Advance
    Keys --> Timer
    Swipe --> Advance
    Buttons --> Advance
    Buttons --> Timer

    Advance --> Render
    Timer --> Advance

    Render --> State[Update State]
    State --> LS[(localStorage)]

    style Render fill:#4a9eff
    style State fill:#51cf66
```

## Progressive Web App (PWA)

```mermaid
graph TB
    subgraph "PWA Features"
        Manifest[manifest.json]
        SW[service-worker.js]
        Icons[App Icons]
    end

    subgraph "Caching Strategy"
        Network[Network First]
        Cache[Cache Fallback]
    end

    subgraph "Capabilities"
        Offline[Offline Support]
        Install[Home Screen Install]
        Updates[Background Updates]
    end

    Manifest --> Install
    SW --> Network
    Network --> Cache
    Cache --> Offline
    SW --> Updates
    Icons --> Install

    style SW fill:#4a9eff
    style Offline fill:#51cf66
```

## Module Organization

```
src/script.ts
├── Interfaces & Types
│   ├── LyricLine
│   └── TrainerState
│
├── State Management
│   ├── loadState()
│   └── saveState()
│
├── Parsers
│   ├── parseLRC()
│   └── parsePlainText()
│
├── File Handling
│   ├── loadDefaultLyrics()
│   ├── processLyricsFile()
│   ├── handleFileUpload()
│   └── resetToDefaultLyrics()
│
├── Theme Management
│   ├── loadTheme()
│   └── toggleTheme()
│
├── Audio
│   ├── handleAudioUpload()
│   └── syncLyricsWithAudio()
│
├── Core Functions
│   ├── render()
│   ├── advance()
│   ├── startTimer()
│   ├── stopTimer()
│   └── toggleTimer()
│
├── Touch Gestures
│   ├── handleTouchStart()
│   ├── handleTouchEnd()
│   └── handleSwipe()
│
└── Initialization
    ├── init()
    └── setupUI()
```

## Future Enhancements

See [suggested_improvements.md](suggested_improvements.md) for planned features including:
- Multiple song playlist management
- Lyrics search and filtering
- Advanced animation effects
- Export/import configurations
- Analytics and progress tracking
