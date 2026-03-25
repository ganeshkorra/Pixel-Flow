# Exact Color System Guide

## Overview
The game now extracts **exact colors directly from your Level Image** instead of mapping to a predefined palette. Every pixel color in the image becomes a unique game element.

---

## How It Works (New Flow)

### 1. **Image Processing**
```
Level Image (PNG/JPG)
    ↓
    For each pixel:
    - If transparent → Mark as empty (null)
    - If visible → Extract exact RGB → Convert to hex (#RRGGBB)
    ↓
autoPattern[][] = array of hex colors (e.g., "#ff5733", "#00aa99", etc.)
```

### 2. **Color Census (Count cubes per color)**
```javascript
colorCensus = {
    "#ff5733": 25,  // 25 red cubes
    "#00aa99": 18,  // 18 teal cubes
    "#ffdd00": 12   // 12 yellow cubes
}

activeLevelColors = ["#ff5733", "#00aa99", "#ffdd00"]  // All unique colors found
```

### 3. **Pig Ammo Assignment**
```
GRID = 3 (9 pigs in 3x3 grid)

Pig #0 → Color: #ff5733 → Ammo: floor(25/3) = 8 ✓
Pig #1 → Color: #00aa99 → Ammo: floor(18/3) = 6 ✓
Pig #2 → Color: #ffdd00 → Ammo: floor(12/3) = 4 ✓
Pig #3 → Color: #ff5733 → Ammo: 8 + remainder = 9 ✓
... (distributes remainder evenly)

Total: 25 + 18 + 12 = exactly all cubes covered
```

### 4. **Coloring**
- **Cubes:** Colored with exact hex from image
- **Characters (Pigs):** Colored with their assigned hex
- **Labels:** Show ammo count in white

---

## Key Changes Made

| Aspect | Before | After |
|--------|--------|-------|
| **Color Source** | Palette IDs (0,1,2...) | Exact hex colors from image |
| **Color Mapping** | `Palette0`, `Palette1`, etc. | Direct RGB extraction |
| **Pattern Array** | Integer IDs (-1 for empty) | Hex strings or null |
| **Color Lookup** | `GameThemes['Palette' + id]` | Direct from array |
| **Flexibility** | Limited to ~6 palette colors | Unlimited colors in image |

---

## Image Creation Guidelines

For best results with exact color extraction:

### ✅ DO:
- Use **solid colors** (fills cells completely)
- Use **true RGB colors** in your image editor
- **Avoid anti-aliasing** on borders
- Use PNG with proper transparency for empty spaces
- Size image to match `LevelRow × LevelCol` (e.g., 35×35 pixels)

### ❌ DON'T:
- Use gradients or blended colors
- Mix multiple colors in one cell
- Use semi-transparent pixels (will be treated as empty)
- Compress with lossy formats (JPEG changes colors)

---

## Example: Assigning Colors to Cubes

```javascript
for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
        const colorHex = autoPattern[r][c];  // Get exact color from image
        
        if (colorHex === null) continue;     // Skip empty spaces
        
        // Create cube
        const clone = cubeModel.clone();
        clone.traverse(n => {
            if (n.isMesh) {
                n.material = new THREE.MeshStandardMaterial({
                    color: colorHex,  // ← Use exact hex directly
                    roughness: 0.1, 
                    metalness: 0.2
                });
            }
        });
        cubesToSmash.push(clone);
    }
}
```

---

## Example: Assigning Ammo Based on Color Count

```javascript
// Count each color in the image
const colorCensus = {};
autoPattern.forEach(row => {
    row.forEach(colorHex => {
        if (colorHex === null) return;
        colorCensus[colorHex] = (colorCensus[colorHex] || 0) + 1;
    });
});

// Get all unique colors
activeLevelColors = Object.keys(colorCensus);

// Assign pigs to colors and calculate ammo
const pigPlan = [];
const pigColorTallies = {};

for (let i = 0; i < 9; i++) {  // 9 pigs for 3x3 grid
    const color = activeLevelColors[i % activeLevelColors.length];
    pigColorTallies[color] = (pigColorTallies[color] || 0) + 1;
    
    pigPlan.push({
        color: color,
        orderIndex: pigColorTallies[color] - 1
    });
}

// Calculate exact ammo
pigPlan.forEach(plan => {
    const totalCubes = colorCensus[plan.color];
    const totalPigs = pigColorTallies[plan.color];
    
    const baseAmmo = Math.floor(totalCubes / totalPigs);
    const remainder = totalCubes % totalPigs;
    
    plan.ammo = (plan.orderIndex < remainder) ? baseAmmo + 1 : baseAmmo;
});
```

---

## Debugging

### To see exact colors extracted:
```javascript
console.log('All colors found:', activeLevelColors);
console.log('Color census:', colorCensus);
console.log('Pig plan:', pigPlan);
```

### Check if color is being applied:
Look in the browser console for:
- `activeLevelColors` - all hex colors found
- `pigPlan` - each pig's color and ammo
- Check scene for cube colors matching your image

---

## Summary
✅ **You now have:**
- Direct color extraction from images
- No palette mapping needed
- Accurate color-to-ammo distribution
- Unlimited color support
- Exact RGB rendering
