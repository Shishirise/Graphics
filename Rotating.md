# WebGL Square Rotation - Step by Step

## Step 1 – Declare the rotation angle (uniform float)
```glsl
uniform float uTheta;
```
- Holds the **rotation angle in radians**.
- Same angle used for all vertices.

## Step 2 – Declare vertex positions (attribute)
```glsl
attribute vec3 aPosition;
```
- Stores each vertex’s original `x, y, z` coordinates.

## Step 3 – Declare vertex color (attribute)
```glsl
attribute vec3 aColor;
```
- Allows each vertex to have its own color.

## Step 4 – Pass color to fragment shader (varying)
```glsl
varying vec3 vColor;
```
- Sends color from vertex shader to fragment shader.

## Step 5 – Compute sine and cosine of rotation angle
```glsl
float s = sin(uTheta);
float c = cos(uTheta);
```
- Needed for rotation formula:
  - `x' = x*cosθ - y*sinθ`
  - `y' = x*sinθ + y*cosθ`

## Step 6 – Calculate rotated x and y
```glsl
gl_Position.x = -s*aPosition.y + c*aPosition.x;
gl_Position.y =  s*aPosition.x + c*aPosition.y;
```
- Applies 2D rotation to each vertex.

## Step 7 – Set depth and homogeneous coordinate
```glsl
gl_Position.z = 0.0;
gl_Position.w = 1.0;
```
- Ensures correct 2D rendering in WebGL.

## Step 8 – Pass color to fragment shader
```glsl
vColor = aColor;
```
- Fragment shader uses this to color the triangles.

## Step 9 – In JavaScript: Create a theta variable
```javascript
let theta = 0.0; // initial rotation angle
```
- Starts the square at original position.

## Step 10 – Get uniform location in JS
```javascript
const thetaLoc = gl.getUniformLocation(program, "uTheta");
```
- Gets memory location of `uTheta` in GPU.

## Step 11 – Send theta to GPU
```javascript
gl.uniform1f(thetaLoc, theta);
```
- Updates the vertex shader with current rotation angle.

## Step 12 – Increment theta for animation
```javascript
theta += 0.01;  // small rotation increment
gl.uniform1f(thetaLoc, theta);
```
- Increases the rotation angle slightly each frame for smooth rotation.

## Step 13 – Render the square
```javascript
gl.clear(gl.COLOR_BUFFER_BIT);
gl.drawArrays(gl.TRIANGLES, 0, 6);
```
- Clears the canvas and draws the rotated square.

---
**Summary:**
- Vertex shader rotates each vertex using `sin` and `cos`.
- JS updates `uTheta` and re-renders.
- Incrementing theta over time creates a smooth rotation animation.


```uniform float uTheta;
attribute vec3 aPosition;
attribute vec3 aColor;
varying vec3 vColor;

void main() {
    float s = sin(uTheta);
    float c = cos(uTheta);

    gl_Position.x = -s*aPosition.y + c*aPosition.x;
    gl_Position.y =  s*aPosition.x + c*aPosition.y;
    gl_Position.z = 0.0;
    gl_Position.w = 1.0;
    vColor = aColor;
}
```
