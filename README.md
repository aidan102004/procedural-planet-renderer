# procedural planet generator
A real-time, procedurally generated planet rendered with C++ and OpenGL. Every pixel is shaded on the GPU using layered 3D Perlin noise mapped onto a sphere. A second, semi-transparent noise layer acts as a rotating cloud cover, and a moving light source gives the planet a day/night cycle.

Press Left Ctrl to generate a new planet with randomised parameters such as size, pattern, colour ect.

## features

- Fragment-shader sphere: a flat quad is drawn and the shader reconstructs sphere normals from the UV coordinates (z = sqrt(1 - x² - y²)), discarding everything outside the unit circle.
- Procedural terrain: fractal (multi-octave) Perlin noise with a hash-based gradient function, smoothstep interpolation, and a configurable seed, grid size, and octave count.
- Seamless spherical mapping: the rotated normal is converted to longitude/latitude, which is used as the noise-space coordinate, so the planet can spin freely.
- Animated clouds: the third noise dimension (zComp) is driven by time, so cloud formations evolve as they rotate. The cloud layer rotates faster than the surface.
- Threshold-based palettes: noise values are bucketed into four colors (e.g. deep water, shallow water, sand, land) using a vec4 of thresholds.
- Lighting: simple diffuse lighting with a slowly orbiting light direction and a brightening pass that keeps lit areas true to the texture color instead of washing out to white.
- Random planet generator: colors, size, grid size, octave count, cloud parameters, and seeds are sampled from Gaussian distributions.

## how it works
 
### rendering
 
Two circles are drawn each frame using the same quad and shader:
 
1. **planet**: scale `1.0` (or randomized), opaque, high-detail noise (grid size ~180, 6 octaves).
2. **clouds**: slightly larger than the planet, partially transparent (`alpha`), lower-frequency noise (grid size ~80, 4 octaves). Only the first palette color is visible; the other three have zero alpha, so clouds appear only where noise falls below the first threshold.
Alpha blending is enabled with `GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA`.
 
### noise
 
For each fragment:
 
1. The surface normal is rotated by `rotateMatrix` (a Y-axis rotation that advances with time).
2. Longitude and latitude are computed with `atan` and `asin`, then remapped to `[0, 1]` and scaled to an 800×800 virtual pixel grid.
3. `createPerlin` sums `octCount` octaves, doubling frequency and halving amplitude each time. Each octave blends two integer slices of the z axis using `zComp`, which makes the noise animate smoothly over time.
4. The result is clamped to `[-1, 1]` and compared against `thresholds` to choose a palette color.
### shader uniforms
 
| Uniform | Type | Purpose |
| --- | --- | --- |
| `rotateMatrix` | `mat3` | Rotates the sphere normals (planet spin) |
| `lightMatrix` | `mat3` | Rotates the light direction |
| `uLightDir` | `vec3` | Base light direction |
| `alpha` | `float` | Overall layer opacity |
| `octCount` | `int` | Number of noise octaves |
| `gridSize` | `int` | Noise cell size (larger = smoother, bigger features) |
| `seed` | `int` | Hash seed for gradients |
| `zComp` | `float` | Third noise dimension, used as time |
| `col1`–`col4` | `vec4` | Palette colors (0–255 RGBA) |
| `thresholds` | `vec4` | Noise cutoffs between palette colors |