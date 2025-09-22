```javascript
<!DOCTYPE html>
<html>
<body>
    <canvas width="750" height="800" id="my_Canvas"></canvas>

    <script>
        var canvas = document.getElementById('my_Canvas');
        var gl = canvas.getContext('webgl') || canvas.getContext('experimental-webgl');

//  Building
var building = [
    -0.7, 0.7, 0,
    -0.7, -0.7, 0,
     0.7, -0.7, 0,
     0.7, 0.7, 0
];
var wallIndices = [0,1,2, 0,2,3];

// Door
var doorVertices = [
    -0.2, -0.7, 0,
    -0.2, -0.3, 0,
     0.1, -0.3, 0,
     0.1, -0.7, 0
];
var doorIndices = [0,1,2, 0,2,3];


//Windows 
var windowVertices = [
    -0.5, -0.5, 0, -0.5,      -0.3, 0, -0.3, -0.3,       0, -0.3, -0.5, 0, // Lower left
     0.2, -0.5, 0, 0.2,       -0.3, 0, 0.4, -0.3,        0,0.4,-0.5, 0,       // Lower right
    -0.5, 0.2, 0, -0.5,       0.4, 0, -0.3, 0.4,         0, -0.3, 0.2, 0,     // Upper left
     0.2, 0.2, 0, 0.2,        0.4, 0, 0.4, 0.4,         0, 0.4, 0.2, 0          // Upper right
];
var windowIndices = [
    0,1,2, 0,2,3,
    4,5,6, 4,6,7,
    8,9,10, 8,10,11,
    12,13,14, 12,14,15
];

// Roof Slab
var slabVertices = [
    -0.7, 0.7, 0,
    -0.7, 0.65, 0,
     0.7, 0.65, 0,
     0.7, 0.7, 0
];
var slabIndices = [0,1,2, 0,2,3];

// Middle Slab
var midSlabVertices = [
    -0.7, 0.025, 0,
    -0.7, -0.025, 0,
     0.7, -0.025, 0,
     0.7, 0.025, 0
];
var midSlabIndices = [0,1,2, 0,2,3];

// Base Slab (bottom)
var baseSlabVertices = [
    -0.7, -0.7, 0,
    -0.7, -0.65, 0,
     0.7, -0.65, 0,
     0.7, -0.7, 0
];
var baseSlabIndices = [0,1,2, 0,2,3];

// Road vertices (below building)
var roadVertices = [
    -1, -0.7, 0,  // top-left
    -1, -1, 0,  // bottom-left
     1, -1, 0,  // bottom-right
     1, -0.7, 0   // top-right
];

// Road indices (two triangles)
var roadIndices = [
    0, 1, 2,
    0, 2, 3
];

var dividerVertices = [];
var dividerIndices = [];
var dashWidth = 0.4; // width of each dash
var gap = 0.1;       // gap between dashes
var yTop = -0.84;
var yBottom = -0.86;
var index = 0;

// Loop across the road width
for (var x = -1; x < 1; x += dashWidth + gap) {
    dividerVertices.push(
        x,       yTop,    0,  // top-left
        x,       yBottom, 0,  // bottom-left
        x+dashWidth, yBottom, 0, // bottom-right
        x+dashWidth, yTop, 0     // top-right
    );

    dividerIndices.push(
        index, index+1, index+2,
        index, index+2, index+3
    );

    index += 4; 
}

// Function to draw shapes 
function drawShape(vertices, indices, color){
    var vertex_buffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, vertex_buffer);
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);

    var index_buffer = gl.createBuffer();
    gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, index_buffer);
    gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(indices), gl.STATIC_DRAW);

    //shaders
    var vertCode = `
        attribute vec3 coordinates;
        void main(void){
            gl_Position = vec4(coordinates,1.0);
        }`;

    var fragCode = `
        precision mediump float;
        uniform vec4 Color;
        void main(void){
            gl_FragColor = Color;
        }`;

    var vertShader = gl.createShader(gl.VERTEX_SHADER);
    gl.shaderSource(vertShader, vertCode);
    gl.compileShader(vertShader);

    var fragShader = gl.createShader(gl.FRAGMENT_SHADER);
    gl.shaderSource(fragShader, fragCode);
    gl.compileShader(fragShader);

    var shaderProgram = gl.createProgram();
    gl.attachShader(shaderProgram, vertShader);
    gl.attachShader(shaderProgram, fragShader);
    gl.linkProgram(shaderProgram);
    gl.useProgram(shaderProgram);

    gl.bindBuffer(gl.ARRAY_BUFFER, vertex_buffer);
    gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, index_buffer);

    var coord = gl.getAttribLocation(shaderProgram, "coordinates");
    gl.vertexAttribPointer(coord, 3, gl.FLOAT, false, 0, 0);
    gl.enableVertexAttribArray(coord);

    var colorLoc = gl.getUniformLocation(shaderProgram, "Color");
    gl.uniform4fv(colorLoc, color);

    gl.drawElements(gl.TRIANGLES, indices.length, gl.UNSIGNED_SHORT,0);
}

// Draw  shapes
gl.clearColor(0.7, 0.9, 1, 1); // dim greenish color
gl.clear(gl.COLOR_BUFFER_BIT);
gl.viewport(0,0,canvas.width,canvas.height);

drawShape(building, wallIndices, [0.92,0.8,0.7,1]);        // wall
drawShape(doorVertices, doorIndices, [0.6,0.4,0.2,1]);     // door
drawShape(windowVertices, windowIndices, [0.5,0.8,1,1]);   // windows
drawShape(slabVertices, slabIndices, [0.89,0.2,0,1]);     // roof slab
drawShape(midSlabVertices, midSlabIndices, [0.89,0.2,0,1]); // middle slab
drawShape(baseSlabVertices, baseSlabIndices, [0.89,0.2,0,1]); // base slab
drawShape(roadVertices, roadIndices, [0.48, 0.48, 0.48, 1]); // dark gray road
drawShape(dividerVertices, dividerIndices, [0, 0.3, 0.3, 0]);


    </script>
</body>
</html>
```
