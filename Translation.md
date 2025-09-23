# WebGL Translation Explanation

## 1. Vertex Shader Line
```glsl
gl_Position = vec4(aPosition.xy + uTranslation, 0.0, 1.0);
```

### Breakdown

- `gl_Position` → Built-in variable in the vertex shader; tells WebGL where to draw each vertex.
- `vec4(...)` → Creates a 4-component vector `(x, y, z, w)`.
  - `x`, `y` → Horizontal and vertical position on canvas.
  - `z` → Depth (usually 0.0 for 2D).
  - `w` → Homogeneous coordinate (keep 1.0).
- `aPosition.xy` → Original x and y coordinates of each vertex.
- `uTranslation` → Vector from JS telling how much to move the square.
- `aPosition.xy + uTranslation` → Adds the translation to the original position, moving the square.

## 2. Translation Variable in JS
```javascript
let translation = { x: 0.0, y: 0.0 };
```
- Tracks how far the square has moved.
- Initially `{x:0, y:0}` → square at original position.

## 3. Move Functions
```javascript
function moveUp() { translation.y += 0.1; updateTranslation(); }
function moveDown() { translation.y -= 0.1; updateTranslation(); }
function moveLeft() { translation.x -= 0.1; updateTranslation(); }
function moveRight() { translation.x += 0.1; updateTranslation(); }
```
- Pressing buttons changes `translation`.
- Up/Down → changes `y`, Left/Right → changes `x`.
- Each call sends the new translation to the GPU and redraws the square.

## 4. Update Translation Function
```javascript
function updateTranslation() {
    const loc = gl.getUniformLocation(program, "uTranslation");
    gl.uniform2f(loc, translation.x, translation.y);
    render();
}
```
- Sends `translation.x` and `translation.y` to shader (`uTranslation`).
- Calls `render()` to draw the square in new position.

## 5. Summary
- `vec4(aPosition.xy + uTranslation, 0.0, 1.0)` moves the square.
- `aPosition.xy` = original vertex positions.
- `uTranslation` = movement vector.
- `translation` in JS starts at 0,0 and changes with buttons.
- `updateTranslation()` applies the movement and redraws.

This is how **translation is implemented in WebGL** using JS and shaders.
