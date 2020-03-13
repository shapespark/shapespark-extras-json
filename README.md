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
### `materials` list

Sets properties of reflective materials, non-reflective materials do not need
to be included. Each entry has the following properties:

+ `name`: required, must match material name from the `FBX`.
+ `roughness`: optional, in `[0,1]` range, defaults to `1`.
+ `rougnessTexture`: optional, if set `roughness` property is ignored.
+ `metallic`: optional, in `[0,1]` range, defaults to `1`.
+ `metallicTexture`: optional, if set `metallic` property is ignored.
+ `bumpTexture`: optional.
+ `bumpScale`: used only when `bumpTexture` is set to scale it, optional,
  in [-0.2,0.2] range, defautls to `0.001`. 
+ `emissionStrength`: optional, minimum value `0`, if set the
material emits light.
+ `doubleSided`: optional, boolean, defaults to `false`, use sparingly
[see the limitations of double sided.
materials](https://www.shapespark.com/docs#materials-tab)

The following material properties are read from the input FBX file,
but can be overwritten by `extras.json`:

+ `baseColor`: optional, three RGB values in `[0,1]` range, in linear
color space, defaults to `[1,1,1]`.
+ `baseColorTexture`: optional, if set `baseColor` is ignored. Can be
set to `null` to reset the base color texture setting from FBX.
+ `opacity`: optional, in `[0,1]` range, defaults to `1`.

Values for `roughnessTexture`, `metallicTexture` and `bumpTexture`
are objects having only one property:

+ `fileName`: required, name of the file inside the archive


[More detailed description of the material
properties](https://www.shapespark.com/docs#materials-tab).


### `views` list

An optional list of views that allow the user to teleport to points of
interest in the scene. If the `views` list is present, the first view
from the list is used as the initial camera placement after the scene
is loaded.

Each entry has the following properties:

+ `name`: optional, a user visible name of the view, defaults to 'viewX'.
+ `mode`: optional, values `"fps"` (default), `"top"`, `"orbit"`.
+ `rotation`: required, `[yaw,pitch]` of the camera in degrees.
+ `fov`: optional, field of view in degrees, in `[1,179]` range.
  View can alter the global field of view configured in the `camera` object.
  The altered field of view is used until the user teleports to another
  location in the scene.


Properties specific for `fps` views:

+ `position`: required, `[x,y,z]` coordinates of the camera,
  `z` axis is up.

Properties specific for `orbit` and `top` views:

+ `target`: required, `[x,y,z]` coordinates of the target the camera
  is looking at.
+ `distance`: required, distance of the camera from the target,
  greater than `0`.
+ `minUpAngle`: for `orbit` views only, optional, minimum elevation
  angle of the camera, in `[-90, 90]` range, defaults to `-90`
+ `maxUpAngle`: for `orbit` views only, optional, maximum elevation
  angle for the camera, in `[-90, 90]` range, defaults to `90`

If the list of views has more than one entry, the scene has an
automatic tour button that automatically teleports the user between
the views.


### `autoTour` object

`autoTour` is an optional object that configures the automatic tour
through all the scene views:

+ `disabled`: optional, boolean, disables the automatic tour feature, defaults to
`false`.
+ `startOnLoad`: optional, boolean, if `true` the automatic tour is started when
the scene is loaded, defaults to `false`.


### `camera` object

Sets the optional camera settings and initial camera placement. The
initial camera placement is used only if the `views` list is empty:

+ `fov`: optional, field of view in degrees in `[1,179]` range, defaults to `70`.
+ `exposure`: optional, camera exposure in `[-3,3]` range, defaults to `0`.
+ `position`: optional, `[x,y,z]` coordinates, `z` axis is up, defaults
  to the center of the scene.
+ `rotation`: optional, `[yaw,pitch]` of the camera in degrees,
  defaults to `[0,0]`.


### `lights` list

If lights are missing, ambient occlusion and sky based lighting can be
used for decent quality illumination with no configuration effort from the
user.

An entry of the `lights` list either adds a new light to the scene or sets
additional properties for a light imported from `FBX`. If a light with the
given name exists in `FBX`, all the entry properties except `name` and
`instances` are copied to the existing light. Otherwise, the entry is
treated as a new light.

Each entry has the following properties:

+ `name`: required, any unique string.
+ `type`: required for new light, `"sun"`, `"spot"`, `"point"` or `"area"`.
+ `strength`: optional, minimum value `0`, defaults to `8` for `sun`
  and `25` for all other light types.
+ `color`: optional, three RGB values in `[0,1]` range, in linear color space,
  defaults to `[1,0.8,0.638]` for `sun` and `[1.0,0.88,0.799]` for all other
  light types.
+ `angle`: for `spot` lights only, optional, in `[1,180]` range,
  defaults to `140`.
+ `photometricProfile`: for `point` and `spot` lights only, optional,
  path to IES light profile file.
+ `width` and `height`: for `area` lights only, optional, both properties in
  `[0.01,5]` range, default to `0.2`.
+ `size`: for all light types except `area`, optional, in `[0.01,0.5]` range,
  defaults to `0.02` for sun and `0.1` for all other light types.
+ `instances`: required, a list of light instances that use the settings.

Each instance has the following properties:

 + `position`: required for `spot`, `point` and `area` lights,
   `[x,y,z]` coordinates.
 + `rotation`: required for `sun`, `spot` and `area` lights, `[yaw,pitch]`.

[More detailed description of light
properties](https://www.shapespark.com/docs#lights-tab)

### `sky` object

`sky` is an optional object that can be included to change the sky
settings. If `sky` is missing, the default sky settings are used, if
`sky` is set to null, the sky is disabled.

+ `strength`: optional, `[0,100]` sky strength that is used for baking,
defaults to `6`.
+ `color`: optional, three RGB values in `[0,1]` range, in linear
color space, defaults to `[0.855,0.863,1]`.
+ `ambientOcclusion`: optional, an object that configures ambient
occlusion parameters. If `ambientOcclusion` is missing, default ambient
occlusion parameters are used, if `ambientOcclusion` is set to null,
ambient occlusion is disabled.
+ `texture`: optional, an object that configures the equirectangular
sky texture that surrounds the scene. The texture is not used for
baking.

`ambientOcclusion` has the following properties:

+ `factor` optional, a float that specifies how strong is the effect
of ambient occlusion, defaults to `0.05`. `0` is an equivalent of
disabled ambient occlusion.
+ `distance` optional, a float that specific how far to search for
occluders, greather than `0`, defaults to `1`. For example, a `distance`
`0.5` means that if there are no occluders within `0.5` meter from
a given point in 3D space, the ambient occlusion has no effect on this point.

`texture` has the following properties:

+ `fileName`: required, name of the file inside the archive that
stores the sky texture.
+ `yawRotation` optional number in `[0,360]` range that specifies the
rotation of the sky texture in degrees. Defaults to `0`.

### `bake` object

`bake` is an optional object that allows to control lightmap baking
quality. It has the following property:

+ `quality`: a string, one of `"draft"`, `"low"`, `"medium"`,
`"high"`, "`super`". Each setting corresponds to different lightmap
baking parameters (the number of samples, bounces and the maximum
number of lightmaps).

### `panoramas` list

`panoramas` is an optional list that allows to specify panorama images
to be generated during the scene import.

Each entry of the list corresponds to one panorama and has the following
properties:

+ `name`: required, unique, must contain only lowercase letters (`a-z`),
  digits (`0-9`) and sepearators (`_` and `-`).
+ `position`: optional, `[x,y,z]` coordinates of the panoramic camera,
  `z` axis is up, defaults to the initial camera position in the scene.
+ `rotation`: optional, yaw rotation for the initial looking direction at
  the panorama image, defaults to `0`.
+ `width`: optional, width in pixels of the panorama image, defaults to
  `8000`, minimum `100`.
+ `width`: optional, height in pixels of the panorama image, defaults to
  `4000`, minimum `100`.

Panoramas are generated in equirectangular (360x180 degree) format. The optimal
`width`:`height` aspect ratio is 2:1.

After the scene has been imported, a generated panorama can be retrieved
by a `GET` request to
`https://[USER_NAME].shapespark.com/[SCENE_NAME]/panoramas/[PANORAMA_NAME]`.
The response for the request is a JPG file containing the panorama image.
