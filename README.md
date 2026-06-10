# Raysetta

The Raysetta Project is a [chrestomathy](https://en.wikipedia.org/wiki/Chrestomathy) of raytracing implementations based on [Ray Tracing in One Weekend](https://raytracing.github.io/). Every implementation is required to injest a scene definition and output an image that is approximately equivalent to any other implementation injesting the same file (some noise is expected due to montecarlo path tracing). These implementations may be used for learning, benchmarking, or fun. Raysetta raytracers are not intended for serious CGI business.

## Scene format

Spec version: `0.1.0`

Scenes are defined in JSON. Some examples can be found in the [/examples] folder.

A scene is made up of a `world` of objects that reference `materials` which may reference `textures` that may also reference `images` or `noises`. The scene also requires a `camera`, as well as a `background` possibly referencing one or more `textures`. Vectors are repesented as an array of three numbers.

### World

The `world` is a collection of objects key'ed by an arbitrary string identifier. The identifiers are somewhat meaningless to the implementation, but may be useful for things like debugging. Each object must have a `type` matching one of `Sphere`, `MovingSphere`, `Quad`, `Tri`, or `Box`. The rest of the properties are type-dependant. Every type currently requires a `material` reference.

Example:
```json
{
  "world": {
    "ball": {
      "type": "Sphere",
      "center": [0.0, 1.0, -3.0],
      "radius": 1.5,
      "material": "some-material"
    },
    "thing": {
      "type": "Box",
      "a": [0, 0, 0], "b": [1, 2, 3],
      "material": "yet-another-material"
    }
  }
}
```

#### Sphere

Represents a sphere. Requires a `center`, `radius`, and `material` reference.

Example:

```json
{
  "type": "Sphere",
  "center": [0.0, 1.0, -3.0],
  "radius": 1.5,
  "material": "some-material"
}
```

#### MovingSphere

Same as a sphere but in motion. Instead of a `center`, it requires a `center1` and `center2` corresponding to position at time t0 and t1 respectively.

Example:

```json
{
  "type": "MovingSphere",
  "center1": [0.0, 1.0, -3.0],
  "center1": [-1.0, 0.0, -2.0],
  "radius": 1.5,
  "material": "some-material"
}
```

#### Quad

Represents a quadrilateral. Requires three vectors `q`, `u`, and `v`, as well as a `material` reference. Property `q` is the origin corner, while `u` and `v` dictate the direction and length of the the two sides connected to the corner.

Example:

```json
{
  "type": "Quad",
  "q": [10.0, 5.0, -3.0],
  "u": [0, 1, 0],
  "v": [0, 0, 0.5],
  "material": "some-material"
}
```

#### Tri

Like a Quad, but just a triangle. Represented by corner points `a`, `b`, `c`, and a `material` reference.

Example:

```json
{
  "type": "Tri",
  "a": [10.0, 5.0, -3.0],
  "b": [10.0, 6.0, -3.0],,
  "c": [10.0, 5.0, -4.5],,
  "material": "some-material"
}
```

#### Box

Six quadritaterals (rectangles in particular) arranged to form a rectangular prism. Defined by its two opposite corner points `a` and `b`, along with a `material` reference.

```json
{
  "type": "Tri",
  "a": [-2.0, 3,0, -2.0],
  "b": [2.0, 3,0, 2.0],
  "material": "some-material"
}
```

### Materials

Similar to `world`, the materials collection is also a JSON object with arbitrary string keys. The `material` property in each object must point to the key in `materials`. Type must be one of `Lambertian`, `Metal`, `Dielectric`, or `DiffuseLight`.

Example:

```json
{
  "materials": {
    "dirt": {
      "type": "Lambertian",
      "texture": "some-other-tex"
    },
    "shiny": {
      "type": "Metal",
      "fuzz" 0.1,
      "texture: "some-tex"
    }
  }
}
```

### Lambertian

Completely diffuse material. Takes only a `texture` property.

Example:

```json
{
  "type": "Lambertian",
  "texture": "some-tex"
}
```

### Metal

Reflective material, taking a `fuzz` property indicating how blurry the reflection is, from a scale of 0 to 1, and a `texture` reference.

Example:

```json
{
  "type": "Metal",
  "fuzz" 0.4,
  "texture: "some-tex"
}
```

### Dielectric

A transparent material, taking only a `refraction_index` property.

Example:

```json
{
  "type": "Dielectric",
  "refraction_index": 1.33
}
```

### Diffuse Light

Emission-only. Takes only a `texture` reference.

```json
{
  "type": "DiffuseLight",
  "texture": "some-tex"
}
```

## Textures

Textures may be sampled via 2D UV coordinates or a 3D point. Types include `SolidColor`, `Checker`, `Image`, and `Noise`.

### Solid Color

A single uniform `albedo`, regardless of sample coordinates. 

Example:

```json
{
  "type": "SolidColor",
  "albedo": [0.5, 0.7, 1.0]
}
```

### Checker

Repeating alternating squares of two different textures, `even` and `odd`, with a `scale`. Textures are specified in full the same way as any other textures, not via reference. Uses 3D point sampling.

Example:

```json
{
  "type": "Checker",
  "scale": 2.3,
  "even": { "type": "SolidColor", "albedo": [1, 0, 1] },
  "odd": { "type": "SolidColor", "albedo": [0, 1, 1] }
}
```

### Image

References an `image`, sampling in 2D UV coordinates.

```json
{
  "type": "Image",
  "image": "some-img"
}
```

### Noise

References a `noise` source. The `scale` property zooms the noise in or out, `depth` adds a number of fractal levels, and `marble_axis` (if present) applies a sine-wave across one of the `x`, `y` or `z` axis.

Example:

```json
{
  "type": "Noise",
  "scale": 4.0,
  "depth": 7,
  "marble_axis": "z",
  "noise": "some-noise"
}
```

## Images

There is only one type of image, `Image`, containing a PNG stored in a base64-encoded data-URL. Some implementations may choose to support other file formats.

Example:

```json
{
  "type": "Image",
  "data": "data:image/png;base64,[Base64 Data ...]"
}
```

## Noises

There is only one type of noise, `Perlin`. Represents Perlin Simplex noise with a baked set of 256 random vectors (`ranvec`) and permutations for each 3D axis (`perm_x`, `perm_y`, `perm_z`). Implementations may choose to support different numbers of points.

Example:

```json
{
  "type": "Perlin",
  "randvec", [
    [0.6385160781453911, -0.7516119887010739, -0.1654588661590717],
    [-0.20794026852296088, 0.886558638764635, -0.4132488654048504],
    // ...
  ],
  "perm_x": [114, 229, /* ... */],
  "perm_y": [97, 9, /* ... */],
  "perm_z": [140, 233, /* ... */]
}
```

## Background

Rays that don't hit any scene objects should sample the background as a spherical map. Possible background types are `Solid`, `Gradient`, `SphereMap` and `CubeMap`.

Example:

```json
{
  "background": {
    "type": "Solid",
    "albedo": [0.5, 0.7, 1.0]
  }
}
```

### Solid backgorund

Make the background a solid color `albedo`, regardless of ray direction.

Example:

```json
{
  "type": "Solid",
  "albedo": [0.5, 0.7, 1.0]
}
```

### Gradient

Smooth gradient from `top` to `bottom`.

Example;

```json
{
  "type": "Gradient",
  "top": [1.0, 1.0, 1.0],
  "bottom": [0.5, 0.7, 1.0]
}
```

### Sphere map

Sample background from a `texture` (typically a panoramic image). Textures are sampled as if on the surface of a sphere of radius 2.

Example:

```json
{
  "type": "SphereMap",
  "texture": "some-tex"
}
```

### Cube map

Sample from an array of six `textures` corresponding to the six sides of a cube.

Example:

```json
{
  "type": "CubeMap",
  "textures": [
    "right-tex",
    "left-tex",
    "top-tex",
    "bottom-tex",
    "front-tex",
    "back-tex"
  ]
}
```

## Camera

Contains various camera optoins:

- `vfov`: Vertical field of view in degrees. (default `90`)
- `lookfrom`: Center of the camera. (default origin `[0, 0, 0]`)
- `lookat`: Point the camera is facing. (default facing down Z-axis `[0, 0, -1]`)
- `vup`: Which way is up. (default Y-up `[0, 1, 0]`)
- `defocus_angle`: How much defocus blur. (default no blur `0.0`)
- `focus_dist`: Focal distance. (default `10.0`)

Example:

```json
{
  "vfov": 60.0,
  "lookfrom": [12.0, 2.0, 3.0],
  "lookat": [0.0, 0.0, 0.0],
  "vup": [0.0, 1.0, 0.0],
  "defocus_angle": 3.4,
  "focus_dist": 10.0
}
```

## CLI

Other than a path to the scene file, implementations are expected to take the following arguments (typically via command-line):

- `-w`, `--width`: Width of the output image. (default `256`)
- `-h`, `--height`: Height of the output image. (default `256`)
- `-s`, `--samples`: Number of rays to send out per pixel for super-sampling. (default `10`)
- `-d`, `--depth`: Maximum number of ray bounces. (default `10`)

CLI is expected output the resulting image to STDOUT in [PPM P3-format](https://en.wikipedia.org/wiki/Netpbm#PPM_example). However, these optional arguments may be accepted to facilitate other forms of output:

- `-f`, `--format`: Output file format. (e.g. `png`)
- `-o`, `--output`: Output file path.

Other implementation-specifc arguments may also be present to control other aspects (e.g. concurrency).

These are merely recommendations, some implementations may choose to receive these parameters in other ways (e.g. via an HTML form in a browser).

## Implementations

- [Ruby](https://github.com/danini-the-panini/raysetta-ruby)
- [Rust](https://github.com/danini-the-panini/raysetta-rust)
- [Java](https://github.com/danini-the-panini/raysetta-java)
- [C++](https://github.com/danini-the-panini/raysetta-cpp)
- [TypeScript](https://github.com/danini-the-panini/raysetta-ts)

Feel free to write your own and make a PR to add it to this list!

## Contributing

Contibutions to this spec are welcome. Add issues for new additions or changes to the spec you'd like to see. Create PRs to improve the clarity of the spec, fix errors, or to add your implementation to the list above.

If you'd like to propose a change to the spec, a link to a reference implementation in the PR description is required. This is so other implementations have something to work against.

## TODO

The following features are planned:

1. Transformations such as translation, rotation, and scale.
2. Density mediums, i.e. Fog

The following features would be cool, too:

1. Generic motion, not just "moving spheres". Possibly even animation.
2. UV coordinates (particularly for triangles)
3. Loading arbitrary triangle meshes (with vertex normals and UVs)
4. Parametric surfaces, NURBs, Catmull-Clark

