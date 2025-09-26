```html
<!DOCTYPE html>
<html>
<body>
<canvas width="750" height="800" id="my_Canvas"></canvas>

<script src="app.js"></script>
</body>
</html>

```

```javascript

// Get WebGL element and context
var canvas = document.getElementById('my_Canvas');
var gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');

// Building (main wall) vertices
var building = [
    -0.7, 0.7, 0,
    -0.7, -0.7, 0,
     0.7, -0.7, 0,
     0.7, 0.7, 0
];

// Building indices
var wallIndices = [0,1,2, 0,2,3];

//  Door vertices
var doorVertices = [
    -0.2, -0.7, 0,
    -0.2, -0.3, 0,
     0.1, -0.3, 0,
     0.1, -0.7, 0
];
// Door indices
var doorIndices = [0,1,2, 0,2,3];

//  Windows vertices
var windowVertices = [
    -0.5, -0.5, 0, -0.5, -0.3, 0, -0.3, -0.3, 0, -0.3, -0.5, 0, // Lower left
     0.2, -0.5, 0,  0.2, -0.3, 0,  0.4, -0.3, 0,  0.4, -0.5, 0, // Lower right
    -0.5, 0.2, 0,  -0.5, 0.4, 0,  -0.3, 0.4, 0,  -0.3, 0.2, 0, // Upper left
     0.2, 0.2, 0,   0.2, 0.4, 0,   0.4, 0.4, 0,   0.4, 0.2, 0  // Upper right
];

// Window indices
var windowIndices = [
    0,1,2, 0,2,3,
    4,5,6, 4,6,7,
    8,9,10, 8,10,11,
    12,13,14, 12,14,15
];

//  Roof Slab vertices
var slabVertices = [
    -0.7, 0.7, 0,
    -0.7, 0.65, 0,
     0.7, 0.65, 0,
     0.7, 0.7, 0
];

// Roof Slab indices
var slabIndices = [0,1,2, 0,2,3];

//  Middle Slab vertices
var midSlabVertices = [
    -0.7, 0.025, 0,
    -0.7, -0.025, 0,
     0.7, -0.025, 0,
     0.7, 0.025, 0
];

// Middle Slab indices
var midSlabIndices = [0,1,2, 0,2,3];

//  Base Slab vertices
var baseSlabVertices = [
    -0.7, -0.7, 0,
    -0.7, -0.65, 0,
     0.7, -0.65, 0,
     0.7, -0.7, 0
];

// Base Slab indices
var baseSlabIndices = [0,1,2, 0,2,3];

//  Road vertices
var roadVertices = [
    -1, -0.7, 0,
    -1, -1, 0,
     1, -1, 0,
     1, -0.7, 0
];

// Road indices
var roadIndices = [0,1,2, 0,2,3];

//  Road Divider Dashes 
var dividerVertices = []; // will be filled in loop
var dividerIndices = [];  // will be filled in loop
var dashWidth = 0.4;      // width of each dash
var gap = 0.1;            // gap between dashes
var yTop = -0.84;         // y position of top of dashes
var yBottom = -0.86;      // y position of bottom of dashes
var index = 0;            // index for indices array

// Create dashes across the road(x from -1 to 1)
for (var x = -1; x < 1; x += dashWidth + gap) {

    // Each dash is a rectangle made of two triangles(4 vertices, 6 indices)
    dividerVertices.push(x, yTop, 0, x, yBottom, 0, x+dashWidth, yBottom, 0, x+dashWidth, yTop, 0);

    // Add indices for the two triangles(0,1,2 and 0,2,3)
    dividerIndices.push(index, index+1, index+2, index, index+2, index+3);

    // Increment index by 4 for next dash
    index += 4;
}

// Vertex shader code
//  (passes through vertex positions)
var vertCode = `
attribute vec3 coordinates;
void main(void){
    gl_Position = vec4(coordinates,1.0);
}`;

// Fragment shader code
var fragCode = `
precision mediump float;  // set float precision
uniform vec4 Color;       // solid color
uniform int Mode;         // 0=brick pattern, 1=solid color
uniform vec2 Resolution;  // canvas resolution
void main(void){
    if (Mode == 0) {

        // Brick pattern
        // Normalize pixel coordinates (from 0 to 1)
        vec2 uv = gl_FragCoord.xy / Resolution;
        float brickCols = 65.0;
        float brickRows = 25.0;

        // Scale UVs to number of bricks
        // and offset every other row
        vec2 scaled = vec2(uv.y * brickCols, uv.x * brickRows);

        // Offset every other row by half a brick
        if (mod(floor(scaled.y), 2.0) == 1.0) {
            scaled.x += 0.7;
        }

        // Get fractional part to find position within brick
        // (0 to 1 in both x and y)
        vec2 f = fract(scaled);
        float gap = 0.08;

        // Define colors
        vec3 brickColor = vec3(0.7, 0.1, 0.1);
        vec3 gapColor = vec3(0.9);

        // If within gap area, use gap color, else brick color
        // (gap on left and bottom edges of each brick)
        // Note: f.x and f.y are in range 0.0 to 1.0
        vec3 color = (f.x < gap || f.y < gap) ? gapColor : brickColor;
        gl_FragColor = vec4(color, 1.0);
    } else {
        // Solid color
        gl_FragColor = Color;
    }
}`;

//  Compile shaders 
function compileShader(src, type){
    var shader = gl.createShader(type);
    gl.shaderSource(shader, src);
    gl.compileShader(shader);
    return shader;
}

//  Create and use shader program
var vertShader = compileShader(vertCode, gl.VERTEX_SHADER);
//  Create and use Fragment shader
var fragShader = compileShader(fragCode, gl.FRAGMENT_SHADER);

//  Create shader program
//  Attach vertex and fragment shaders
//  Link program
//  Use program
var shaderProgram = gl.createProgram();
gl.attachShader(shaderProgram, vertShader);
gl.attachShader(shaderProgram, fragShader);
gl.linkProgram(shaderProgram);
gl.useProgram(shaderProgram);


//  Get attribute and uniform locations
//  Set resolution uniform
//  Enable vertex attribute array
//  Set attribute pointer
//  Set uniform values before drawing
var coord = gl.getAttribLocation(shaderProgram, "coordinates");
var colorLoc = gl.getUniformLocation(shaderProgram, "Color");
var modeLoc = gl.getUniformLocation(shaderProgram, "Mode");
var resLoc = gl.getUniformLocation(shaderProgram, "Resolution");
gl.uniform2f(resLoc, canvas.width, canvas.height);

// Helper function to draw a shape given vertices, indices, color, and mode
function drawShape(vertices, indices, color, mode){
    var vertex_buffer = gl.createBuffer();

    // Bind vertex buffer and upload data
    gl.bindBuffer(gl.ARRAY_BUFFER, vertex_buffer);
    // Fill buffer with vertex data
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);

    // Create, bind, and fill index buffer
    var index_buffer = gl.createBuffer();
    // Bind index buffer and upload data
    gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, index_buffer);
    // Fill buffer with index data
    gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(indices), gl.STATIC_DRAW);

    // Point attribute to currently bound vertex buffer
    gl.vertexAttribPointer(coord, 3, gl.FLOAT, false, 0, 0);
    // Enable attribute
    gl.enableVertexAttribArray(coord);

    // Set uniform values   
    gl.uniform1i(modeLoc, mode);
    // Set color uniform
    gl.uniform4fv(colorLoc, color);

    // Draw the shape using the index buffer
    gl.drawElements(gl.TRIANGLES, indices.length, gl.UNSIGNED_SHORT, 0);
}

//Draw the scene
// Clear canvas and set viewport
// Set background color and clear
// Set viewport to canvas size
gl.clearColor(0.82, 0.9, 1, 1); // sky blue background
gl.clear(gl.COLOR_BUFFER_BIT); 
gl.viewport(0,0,canvas.width,canvas.height);

//Call drawShape for each part of the scene
// Parameters: vertices, indices, color [r,g,b,a], mode (0=brick pattern, 1=solid color)

drawShape(building, wallIndices, [0.92,0.8,0.9,1], 0); // brick wall
drawShape(doorVertices, doorIndices, [0.6,0.4,0.2,1], 1); // door
drawShape(windowVertices, windowIndices, [0.5, 0.8, 1.0, 1], 1); // windows
drawShape(slabVertices, slabIndices, [0.92,0.8,0.7,1], 1); // roof slab
drawShape(midSlabVertices, midSlabIndices, [0.92,0.8,0.7,1], 1); // mid slab
drawShape(baseSlabVertices, baseSlabIndices, [0.92,0.8,0.7,1], 1); // base slab
drawShape(roadVertices, roadIndices, [0.48, 0.48, 0.48, 1], 1); // road
drawShape(dividerVertices, dividerIndices, [0, 0, 0, 0], 1); // divider

```
