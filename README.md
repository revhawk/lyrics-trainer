# Lyrics Trainer

A feature-rich offline-capable Progressive Web App that steps you through song lyrics one line at a time. Includes "This Is the Moment" by default, with support for uploading custom lyrics from text files. Features manual and automatic navigation, keyboard support, persistent state, and mobile-friendly interface.

## Features

- **Display lyrics one line at a time** with enhanced visibility
- **Upload custom lyrics** from text files (.txt format) or timestamped LRC files (.lrc format)
- **Karaoke mode** 🎤 with timestamped lyrics and audio synchronization (NEW!)
- **Audio player** 🎵 for optional music playback synced with lyrics (NEW!)
- **Manual navigation** (Next, Previous, Seek slider, Keyboard shortcuts)
- **Mobile-friendly** with swipe gestures and responsive design
- **Theme support** with manual dark/light toggle and system preference detection
- **Auto-advance** with configurable delay (0.5-10 seconds)
- **Persistent state** via `localStorage` (remembers position, settings, and custom lyrics)
- **Offline support** via service worker (full PWA capabilities)
- **Error handling** with user-friendly messages
- **Touch-optimized** interface with proper touch targets
- **Stable layout** that doesn't jump with varying text lengths

## Getting Started

### Prerequisites

- Node.js (>=14)
- npm

### Installation

```bash
npm install
```

### Build

Compile the TypeScript sources to the `public/` directory:

```bash
npm run build
```

### Development

Rebuild on changes:

```bash
npm run watch
```

### Serve

Serve the compiled app locally on http://localhost:5173:

```bash
npm run serve
```

### Testing

Run the test suite:

```bash
npm test
```

Run tests in watch mode (for development):

```bash
npm run test:watch
```

Run tests with UI dashboard:

```bash
npm run test:ui
```

Generate test coverage report:

```bash
npm run test:coverage
```

See [TESTING.md](TESTING.md) for more details on the testing approach.

### Mobile Access

To access on mobile devices on the same network:
1. Find your computer's IP address
2. Navigate to `http://[YOUR-IP]:5173` on your mobile device

## Usage

### Desktop
- **Navigate**: Use Previous/Next buttons, arrow keys, or the seek slider
- **Play/Pause**: Press spacebar or click the Play button
- **Theme**: Click the moon/sun icon to toggle dark/light mode
- **Keyboard shortcuts**:
  - `←` Previous line
  - `→` Next line
  - `Space` Play/Pause
  - `Home` First line
  - `End` Last line

### Mobile
- **Swipe gestures**: Swipe left/right on the lyrics to navigate
- **Touch-friendly**: All buttons are optimized for touch
- **Theme**: Tap the moon/sun icon to switch themes

### Custom Lyrics
- **Upload**: Click "Upload Lyrics (.txt/.lrc)" and select a text or LRC file
- **Format**: One line per row in text files, or `[mm:ss.xx]` timestamps in LRC files
- **Lyrics Folder**: All lyrics files are stored in the `lyrics` folder
- **Reset**: Click "Reset to Default" to return to the original lyrics
- **Persistence**: Last used lyrics file is remembered between sessions

### Karaoke Mode (NEW!)
- **LRC Files**: Upload `.lrc` files with timestamped lyrics
- **Audio Upload**: Click "Upload Audio (optional)" to add music
- **Auto-Sync**: Lyrics automatically advance based on audio playback
- **Timestamps**: View exact timing for each line (e.g., `[00:12]`)
- **Manual Control**: All navigation features still work in karaoke mode
- **Documentation**: See [KARAOKE.md](KARAOKE.md) for detailed instructions

## Project Structure

```
.
├── lyrics/                     # Lyrics text files
│   ├── this_is_the_moment.txt  # Default lyrics (plain text)
│   ├── happy_birthday.lrc      # Example LRC file with timestamps
│   └── ...                     # User-added lyrics files
├── public/                     # Compiled assets & static files
│   ├── index.html
│   ├── style.css
│   ├── script.js
│   ├── lyrics/                 # Served lyrics files
│   │   └── this_is_the_moment.txt  # Default lyrics
│   ├── manifest.json
│   ├── service-worker.js
│   ├── icon-192.png            # PWA icon
│   └── icon-512.png            # PWA icon
├── src/                        # TypeScript source files
│   └── script.ts
├── tests/                      # Test files
│   ├── setup.ts                # Test setup and mocks
│   ├── utils.test.ts           # Utility function tests
│   ├── state.test.ts           # State management tests
│   └── theme.test.ts           # Theme functionality tests
├── suggested_improvements.md   # Roadmap & future enhancements
├── TESTING.md                  # Testing documentation
├── KARAOKE.md                  # Karaoke mode documentation
├── vitest.config.ts            # Vitest configuration
├── package.json
├── tsconfig.json
└── README.md
```

## PWA / Offline Support

The app is a full Progressive Web App with:
- Service worker for offline functionality
- App manifest for installation
- Icons for home screen
- Network-first caching strategy for lyrics

## Recent Improvements

- ✅ **Karaoke mode with LRC file support** - Timestamped lyrics with audio sync
- ✅ **Audio player integration** - Upload and play music files
- ✅ **Automatic synchronization** - Lyrics advance with audio playback
- ✅ Added loading states and error handling
- ✅ Implemented manual theme toggle
- ✅ Enhanced mobile experience with swipe gestures
- ✅ Improved lyrics visibility
- ✅ Fixed localStorage handling
- ✅ Added responsive design
- ✅ Created PWA icons
- ✅ Added custom lyrics upload from text files
- ✅ Created dedicated lyrics folder for better organization
- ✅ Fixed layout jumping with stable dimensions

## Suggested Improvements

See [suggested_improvements.md](suggested_improvements.md) for ideas on future enhancements, including:
- Multiple song support
- Lyrics search functionality
- Animation transitions
- Progress indicators
- And more...

## Deployment

This app can be deployed to any static hosting service:

- **GitHub Pages**: Push to a `gh-pages` branch or configure in repository settings
- **Netlify**: Connect your GitHub repo and deploy the `public` directory
- **Vercel**: Import your GitHub repository
- **Any web server**: Upload the contents of the `public` directory

The app is completely static after building and requires no backend services.

## License

MIT License - see [LICENSE](LICENSE) file for details.