# Missia Monster - Face Mask App

## Project Overview

A web-based face mask app built for a friend who makes kids' costumes. Users can:
1. Photograph hand-drawn paper masks
2. Cut out the background automatically
3. Add anchor points (eyes, mouth) for face alignment
4. Wear masks in real-time using face tracking
5. Share masks via a cloud gallery

**Single-file architecture**: Everything is in `index.html` - HTML, CSS, and JavaScript. No build process needed.

## Key User Flows

### Creating a Mask
1. User draws a mask on paper
2. Takes photo with rear camera
3. App removes paper background (flood fill algorithm)
4. User taps to cut eye holes (transparency)
5. User adds anchor points: left eye, right eye, (optional) left mouth, right mouth
6. Saves as JSON or publishes to gallery

### Wearing a Mask
1. App loads random mask on startup
2. Front camera tracks face using MediaPipe (478 landmarks)
3. Mask overlays on face, aligned by eye anchors
4. Tap screen to cycle through gallery masks
5. Gear icon shows/hides UI controls

## Technical Decisions

### Why Single File?
- Deploys instantly to GitHub Pages
- Works offline (except MediaPipe CDN)
- Easy to edit from any device
- No npm/webpack complexity

### Coordinate System (IMPORTANT)
- **Selfie view is mirrored**: User sees themselves as in a mirror
- **Anchor points are from VIEWER's perspective**: "Left eye" = left side of screen
- **Code flips X coordinates**: `(1 - x)` when rendering to match mirrored view
- This was a major bug fix - originally left/right were swapped

### MediaPipe Landmarks Used
```
LEFT_EYE_CENTER = 468
RIGHT_EYE_CENTER = 473
MOUTH_LEFT = 61
MOUTH_RIGHT = 291
```

### Mask Data Structure
```json
{
  "version": 1,
  "image": "data:image/png;base64,...",  // or URL path
  "width": 800,
  "height": 600,
  "anchors": {
    "leftEye": { "x": 0.35, "y": 0.4 },
    "rightEye": { "x": 0.65, "y": 0.4 },
    "leftMouth": { "x": 0.38, "y": 0.7 },   // optional
    "rightMouth": { "x": 0.62, "y": 0.7 }   // optional
  }
}
```
Coordinates are normalized 0-1 (percentage of image dimensions).

## Branches

### `main` (v5.0)
- Stable version for public use
- Eye anchors only
- Uses `masks/` folder for gallery

### `mouth-anchors` (v5.1-mouth)
- Experimental mouth tracking
- 4-point anchors (eyes + mouth corners)
- Uses `masks-mouth/` folder (separate gallery)
- Mask stretches horizontally based on mouth width

## GitHub Integration

### Publishing Flow
1. App prompts for GitHub Personal Access Token (stored in localStorage)
2. Uploads PNG to `masks/images/` (or `masks-mouth/images/`)
3. Updates `manifest.json` with mask metadata
4. Token needs `repo` scope

### Gallery Loading
1. First tries GitHub API (if token exists) - no caching
2. Falls back to raw.githubusercontent.com
3. Falls back to local file (for offline/development)

## Known Quirks

### Safari Caching
- Aggressive caching required cache-busting (`?t=Date.now()`)
- Also needed `cache: 'no-store'` fetch option
- GitHub Pages CDN adds another layer of caching

### Canvas pointer-events
- `#debugCanvas` has `pointer-events: none` for overlay rendering
- Tap detection uses a separate `#tapOverlay` div on top

### L/R Markers in Saved Images
- Edit mode shows LE/RE/LM/RM markers for placement
- Must redraw canvas WITHOUT markers before saving/publishing
- Easy to forget and bake markers into saved image

## Debug Mode

Hidden debug overlay can be enabled by changing:
```html
<div id="debugInfo" style="display:none;...">
```
to `display:block`. Shows tap events, gallery loading, mask index changes.

## File Structure

```
/
├── index.html          # The entire app
├── masks/              # Main branch gallery
│   ├── manifest.json
│   └── images/
├── masks-mouth/        # Mouth-anchors branch gallery
│   ├── manifest.json
│   └── images/
├── overlays/           # SVG overlays (not currently used)
└── .github/workflows/  # GitHub Pages deploy
```

## Future Ideas (Not Implemented)

- Nose anchor point
- Ear anchor points
- Animated masks (GIF support)
- Sound effects on expression changes
- Multi-face support
- Mask categories/tags

## Rollback Points

- **v5.0**: Last stable before mouth feature
- **v4.9**: Before debug was hidden
- **v4.1**: Before delete feature

## Contact

Built with Claude Code for Durrell Bishop's friend's costume business.
