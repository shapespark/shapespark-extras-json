# Introduction.

This document describes a JSON file format that can be used together
with the main 3D file (FBX, COLLADA, OBJ, GLTF) to import additional
properties to Shapespark.

Shapespark can import basic material settings from the standard 3D
formats (base color, base color texture, opacity). For FBX and COLLADA
Shapespark can also import light positions and types, but not light
strength, size, color and angle. Other material properties (roughness,
metallic, bump, emissive) need to be set in the Shapespark
editor.

The JSON format allows to import these additional properties. The
format is intended for writing exporters.

# JSON file location.

When importing a 3D file Shapespark looks for a JSON file in the same
directory, with the same name, but a `.json` extension. For example,
if `Documents\InteriorA\InteriorA.fbx` is imported, the JSON file
should be in `Documents\InteriorA\InteriorA.json`. If such file is
present, it is used to import additional properties.

# JSON file format.

[Example file](./example/cubes.json)

Entries:
## `materials` list

Adds additional properties to the imported materials, not all materials
need to be included on the list. Each entry has the following
properties:

+ `name`: required, must match material name from the input file.
+ `roughness`: optional, in `[0,1]` range, defaults to `1`.
+ `rougnessTexture`: optional, if set `roughness` property
is ignored.
+ `metallic`: optional, in `[0,1]` range, defaults to `1`.
+ `metallicTexture`: optional, if set `metallic` property is ignored.
+ `bumpTexture`: optional.
+ `bumpScale`: optional, in `[0,1]` range, used when `bumpTexture`
  property is set to scale it.
+ `emissionStrength`: optional, in `[0,1000]` range, if set the
material emits light.
+ `doubleSided`: optional, defaults to `false`, use sparingly
[see the limitations of double sided.
materials](https://www.shapespark.com/docs#materials-tab)


The following material properties are read from the input file, but
can be overwritten by `extras.json`:

+ `baseColor`: optional, three RGB values in `[0,1]` range, in linear
color space.
+ `baseColorTexture`: optional, if set `baseColor` is ignored. Can be
set to `null` to reset the base color texture setting from the input file.
+ `opacity`: optional, in `[0, 1]` range.

[More detailed description of the material
properties](https://www.shapespark.com/docs#materials-tab).

### `views` list

An optional list of views that allow the user to teleport to points of
interest in the scene. If the `views` list is present, the first view
from the list is used as the initial camera placement after the scene
is loaded.

Each entry has the following properties:

+ `name`: optional, a user visible name of the view, defaults to 'viewX'.
+ `position`: required, `[x, y, z]` coordinates of the camera, `z` axis is
  up.
+ `rotation`: required, `[yaw, pitch]` of the camera in degrees.
+ `fov`: optional, field of view in degrees. View can alter the global
  field of view configured in the `camera` object. The altered field of
  view is used until the user teleports to another location in the
  scene.

If the list of views has more than one entry, the scene has an
automatic tour button that automatically teleports the user between
the views.

### `autoTour` object

`autoTour` is an optional object that configures the automatic tour
through all the scene views:

+ `disabled`: optional, disables the automatic tour feature, defaults to
`false`.
+ `startOnLoad`: optional, if `true` the automatic tour is started when
the scene is loaded, defaults to `false`.

### `camera` object

Sets the optional camera settings and initial camera placement. The
initial camera placement is used only if the `views` list is empty:

+ `fov`: optional, field of view in degrees.
+ `exposure`: optional, camera exposure in `[-3,3]` range, defaults to `0`.
+ `position`: optional, `[x, y, z]` coordinates, `z` axis is up.
+ `rotation`: optional, `[yaw, pitch]` of the camera in degrees.

### `lights` list

An entry of the `lights` list either adds a new light to the scene or
sets additional properties for an imported light. If a light with the
given name exists in the input file, all the entry properties except
`name` and `instances` are copied to the existing light. Otherwise,
the entry is treated as a new light.

Each entry has the following properties:

+ `name`: required, any unique string.
+ `type`: required for new light, `"sun"`, `"spot"` or `"point"`.
+ `strength`: required, `[0,1000]`
+ `angle`: required for `spot` lights, `[0,360]`.
+ `photometricProfile`: optional, only for `point` and `spot` lights, path to
IES light profile file.
+ `width` and `height`: required for `area` lights, both properties in
  `[0.01,5]` range
+ `size`: required for all light types except `area`, `[0.01,0.5]`
+ `color`: required, three RGB values in `[0,1]` range, in linear color space.
+ `instances`: a list of light instances that use the settings.

Each instance has the following properties:

 + `position`: required for `spot` and `point` lights, `[x, y, z]` coordinates.
 + `rotation`: required for `spot` and `sun` lights, `[yaw, pitch]`.

[More detailed description of light
properties](https://www.shapespark.com/docs#lights-tab)
