# WebGL - Drawing a Triangle
## Step 1: Prepare the Canvas and Get WebGL Rendering Context
```html
<canvas id="canvas1" width="500" height="500"></canvas>

<script>
//Get WebGL Rendering Context
  const canvas = document.getElementById('canvas1');

  console.log(canvas); // test: should log the <canvas> element
</script>

```
## Step 2: Define the Geometry and Store it in Buffer Objects
```js
// Geometry: 3 vertices (x, y, z)
var vertices = [
   -0.5,  0.5, 0.0,   // top-left
   -0.5, -0.5, 0.0,   // bottom-left
    0.5, -0.5, 0.0    // bottom-right
];

// Indices: use vertex 0,1,2 to form one triangle
var indices = [0, 1, 2];
```

## Step 3: Create and Compile the Shader Programs
```js
(Vertex Shader= gl_position)

var vertCode =
   'attribute vec3 coordinates;' + // input attribute for positions
	
   'void main(void) {' +
      ' gl_Position = vec4(coordinates, 1.0);' + // set vertex position
   '}';

(Fragment Shader = gl_FragColor)

var fragCode = 'void main(void) {' +
   ' gl_FragColor = vec4(1, 0.5, 0.0, 1);' +
'}';
```
## Step 4 − Associate the Shader Programs to the Buffer Objects

