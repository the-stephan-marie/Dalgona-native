# Tap-to-Advance WebAR Experience

A boilerplate for creating interactive "Tap-to-Advance" Augmented Reality experiences using Google's `<model-viewer>` component.

## Features

- ✅ Native AR support for iOS and Android
- ✅ Tap-to-advance animation sequencing
- ✅ Customizable UI overlay
- ✅ Single GLB file with multiple animation clips
- ✅ No third-party subscriptions required

## Project Structure

```
.
├── index.html          # Main HTML file with all logic
├── assets/             # Place your 3D assets here
│   ├── model.glb       # Your 3D model (required)
│   ├── model.usdz      # iOS-optimized version (optional)
│   └── env.hdr         # Custom environment map (optional)
└── README.md           # This file
```

## Setup Instructions

### 1. Prepare Your 3D Assets

Your 3D model should be exported as a single `.glb` file with multiple animation clips (Takes) embedded:

- **Required:** `model.glb` - Your main 3D model with all animation clips
- **Optional:** `model.usdz` - iOS-optimized version for better performance
- **Optional:** `env.hdr` - Custom environment map for lighting

**Important:** Animation clip names in your 3D file must match the names in the `CONFIG.steps` array in `index.html`.

### 2. Configure Your Experience

Open `index.html` and update the `CONFIG` object:

```javascript
const CONFIG = {
    modelPath: 'assets/model.glb',        // Path to your GLB file
    iosModelPath: 'assets/model.usdz',    // Path to USDZ (optional)
    initialAnimation: 'Idle',             // Starting animation clip name
    
    // Update this array with your animation sequence
    steps: [
        { clip: 'Step1_Mix', title: 'Step 1', desc: 'Description...' },
        { clip: 'Step2_Whip', title: 'Step 2', desc: 'Description...' },
        // ... add more steps
    ],
    
    // Customize messages
    welcomeTitle: 'Welcome',
    welcomeDesc: 'Tap anywhere to start!',
    continuePrompt: 'Tap to continue ->',
    resetTitle: 'Replay?',
    resetDesc: 'Tap to start again.'
};
```

### 3. Place Your Assets

1. Export your 3D model as `model.glb` (or rename it in the config)
2. Place it in the `assets/` folder
3. If you have a USDZ version, place it as `model.usdz` in `assets/`

## Running the Experience

### Local Development

You cannot simply open `index.html` in a browser. AR requires a server environment.

**Option A: VS Code Live Server (Recommended)**
1. Install the "Live Server" extension in VS Code
2. Right-click `index.html` → "Open with Live Server"
3. Test on desktop browser first

**Option B: Python Simple Server**
```bash
python -m http.server 8000
```
Then open `http://localhost:8000` in your browser.

### Testing on Mobile (AR)

AR requires HTTPS. Use one of these methods:

**Option 1: ngrok (Quick Testing)**
1. Start your local server (e.g., Live Server on port 5500)
2. Run: `ngrok http 5500`
3. Copy the HTTPS URL and open on your phone

**Option 2: Deploy to Web (Production)**
- **GitHub Pages:** Push to a repo and enable Pages
- **Netlify:** Drag and drop your folder
- **Vercel:** Connect your GitHub repo

## Animation Setup (Cinema 4D)

If you're using Cinema 4D, follow these steps:

1. **Create Takes:** Use the Take System to create separate animation clips:
   - `Idle` - Static starting pose
   - `Step1_Mix` - First animation
   - `Step2_Whip` - Second animation
   - etc.

2. **Export Settings:**
   - File → Export → glTF 2.0 (.glb)
   - Format: Binary (.glb)
   - Textures: ✓ Checked
   - Export Animation: ✓ Checked
   - Takes: Export All Takes ✓

3. **Important:** Ensure the last frame of each animation matches the first frame of the next to prevent snapping.

## Customization

### Styling

Edit the `<style>` section in `index.html` to customize:
- Colors and fonts
- UI card appearance
- AR button styling
- Background colors

### Environment Lighting

Add a custom HDR environment map:
```html
<model-viewer 
    environment-image="assets/kitchen_environment.hdr"
    ...>
```

Or use built-in presets:
- `environment-image="neutral"` (default)
- `environment-image="legacy"`
- `environment-image="sunset"`

## Troubleshooting

**Animation not playing?**
- Check that clip names in `CONFIG.steps` match your 3D file's animation names exactly (case-sensitive)
- Verify your GLB file has animations exported

**Model looks dark?**
- Add lights to your 3D scene, or
- Use `environment-image="legacy"` for default lighting

**Object snapping back?**
- Ensure the last frame of each animation matches the first frame of the next in your 3D software

**AR button not appearing?**
- AR requires HTTPS (secure connection)
- Test on a real device, not desktop browser
- Ensure your model file is loading correctly

## Browser Support

- **iOS:** Safari 12+ (via AR Quick Look)
- **Android:** Chrome 81+ (via Scene Viewer)
- **Desktop:** Chrome/Edge (WebXR preview)

## License

This boilerplate is provided as-is. Customize it for your project needs.
