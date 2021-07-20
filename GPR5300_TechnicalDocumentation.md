# GPR5300 Technical Document

## Introduction
This repository contains a simple [C++20](https://en.cppreference.com/w/cpp/20) OpenGL rendering engine capable of loading [.obj(+ it's .mtl)](https://en.wikipedia.org/wiki/Wavefront_.obj_file) 3D model files with [.ktx](http://github.khronos.org/KTX-Specification/) textures using [etc1](https://en.wikipedia.org/wiki/Ericsson_Texture_Compression) compression. [GLSL](https://www.khronos.org/opengl/wiki/Core_Language_(GLSL)) shaders were used.
## External libraries used
* [Windows 10's built in OpenGL library](https://docs.microsoft.com/en-us/windows/win32/opengl/opengl)
* [glad](https://github.com/Dav1dde/glad)
* [sdl2](https://www.libsdl.org)
* [imgui](https://github.com/ocornut/imgui)
* [glm](https://github.com/g-truc/glm)
* [gli](https://github.com/g-truc/gli)
* [xxhash](https://github.com/Cyan4973/xxHash)
* [tinyobjloader](https://github.com/tinyobjloader/tinyobjloader)
* [tracy](https://github.com/wolfpld/tracy)
* [stb](https://github.com/nothings/stb)

All the libraries are linked together using [vcpkg](https://vcpkg.io/) and [cmake](https://cmake.org/). The basis for the engine and the vcpkg and cmake configurations has been provided by [Elias Farhan](https://github.com/EliasFarhan).

## Hardware/Software used
The engine has been made to run reasonably (consistent 60 FPS) on the following platform:
* CPU: [i7-6500U @ 2.50[GHz], 2 physical core, 4 logical.](https://ark.intel.com/content/www/us/en/ark/products/88194/intel-core-i7-6500u-processor-4m-cache-up-to-3-10-ghz.html)
* RAM: 16[GB].
* GPU: [Nvidia GeForce 940M with 2[GB] of VRAM, with driver version ~466.47](https://www.techpowerup.com/gpu-specs/geforce-940m.c2643)
* Microsoft Windows 10 Home x64, ~version 10.0.19042 .

## Techniques used
These are the techniques used in the demo in no particular order:
### Instancing
Instancing simply consists in re-using the same per-vertex data to draw multiple instances of a mesh in different places using a single draw call. This is the default behaviour of the engine. The only exception is the drawing of the skybox.

<p align="left">
  <img width="960" height="523" src="../assets/instancing.png">
</p>

This technique comes in particularly handy for things like particle systems. In this demo, a single particle consists of a mesh of 3 intersecting planes with an alpha texture used to draw the shape of a star on each plane:

<p align="left">
  <img width="539" height="487" src="../assets/particle_mesh.png">
</p> 

This same mesh is simply instanced 512 times in my case: the per-vertex data is loaded into the GPU's memory once at initialization and the dynamic per-instance data of 3 floats representing the position of the mesh is transfered to the GPU once every frame, resulting in a total transfer of 6'144 bytes. Each particle is then colored in the fragment shader using it's gl_InstanceId as input to give them differing colors.
The vertex shader input ends up therefore looking something like this:

```
layout (location = 0) in vec3 VertexPosition; // Changes for every vertex.
layout (location = 1) in vec3 ModelPosition; // Changes for every instance.
```

### Frustum culling
Although the scene is simple enough not to warrant an such an optimization, every time a Model is drawn, the user is given the option to perform a culling pass using the camera's frustum to determine which instances of the Model need to be updated and displayed. In the case of this engine the use of this pass is less to limit the number of draw calls, as with instancing this isn't an issue, but rather to limit the amount of dynamic data that needs to be transferred to the GPU every frame.
This optimization should do little when the number of instances of a Model is small as the most lengthy operation in such a scenario would be the wait on the GPU, but should get more effective as the number of instances grow.

<p align="left">
  <img width="1160" height="1378" src="../assets/dynamic_data_transfer.png">
</p> 

A list of instances to draw can be easily established thusly: only the instances whose position relative to the camera's position lays inside the camera's frustum need to be drawn. To know whether an object is within the frustum, all we need to do is examine the geometric projection of the object's position vector onto the vectors normal to the planes that make up the frustum. This is easilly comprehensible when considering the frustum's Z axis:

<p align="left">
  <img width="600" height="580" src="../assets/frustum_z.png">
</p> 

We can see that the object COULD only be within the frustum's bound only if the projection of the blue vector is greater than the Z coordinate of the near plane AND smaller than the Z coordinate of the far plane. The blue vector is given by: ObjectPosition - CameraPosition. The red vector is the Front vector of the camera.
Similarly, the same comparison is to be done for each of the side planes of the frustum:

<p align="left">
  <img width="600" height="480" src="../assets/frustum_sides.png">
</p> 

As you can see, this time if the projection of the blue vector on the red one is greater than zero, there's no way the object could be inside the frustum. This time, the red vector is given by rotating the camera's Left vector by FOV/2 degrees counter-clockwise. This can easily be done using glm::angleAxis(radian, axis).

### Shape interpolation
Shape interpolation is a very simple technique that allows a shape to be interpolated between two other shapes on the condition that the vertices of the two other shapes have a one-to-one correspondance:

<p align="left">
  <img width="500" height="440" src="../assets/morphing.png">
</p>

Here, we have two targets. One is a red pentagon with 5 vertices. The other a blue triangle with 5 vertices as well but two of them are located on the triangle's sides. If we pass these two sets of data to the vertex shader as well as an arbitrary interpolation factor, we can create a third shape, drawn here in black that is a mixture of the pentagram and the triangle.

This was how the roof collapsing effect was done for the Half Life 2's E3 2003 tech demo: https://youtu.be/4ddJ1OKV63Q?t=112
This very simple animation technique is still the foundation of Morph Target Animation, ubiquitous to all modern game engines that can animate character expressions.

<p align="left">
  <img width="486" height="421" src="https://www.site.uottawa.ca/~wslee/SITEimages/diagonalView_AnimationVideo.gif">
</p>

### Normalmapping
All objects in the scene that are shaded using the Blinn-Phong's lighting model are using a normal map: a texture that instead of colors contains the xyz components of a vector that would be normal to the surface of the object at it's texture coordinate. Using these normals instead of the ones generated from the .obj's data gives us a much better resolution of the object's surface at very little cost.

### Deferred shading
The rendering pass is separated between a geometry pass and a shading pass. The geometry pass gathers all the data relevant for computing the lighting and writes it to 4 separate targets: one for a fragment's albedo, one for it's position, one for it's normal and one for the material's shininess.
The shading pass then takes these inputs as textures and computes the lighting for the whole screen, so no needless lighting computation is performed: all pixels thusly rendered are guaranteed to be the ones displayed on screen.

<p align="left">
  <img width="500" height="358" src="../assets/deferred_albedo.png">
</p>
<p align="left">
  <img width="500" height="358" src="../assets/deferred_positions.png">
</p>
<p align="left">
  <img width="500" height="358" src="../assets/deferred_normals.png">
</p>

### Bloom effect
During the shading pass, a bloom effect is applied globally to the whole screen. The basic idea behind a bloom effect is simple: in addition to drawing the scene as usual, draw a copy of it onto a second texture but only the pixels that are brighter than some arbitrary value. Blur this second texture and combine it with the first result of the shading pass and you end up with something like this:

<p align="left">
  <img width="703" height="313" src="http://3.bp.blogspot.com/-xTb-8Z79dkY/Ux3J4DlfhnI/AAAAAAAACtw/2Ie_-aJlhuY/s1600/bloom.jpg">
</p>

The result is that light appears to bleed into it's surroundings and appears brighter than it actually is. While there's multiple ways to implement this effect, the differences lay in the way this second texture that only contains the bright colors is blurred. Some might perform the blurring effect per fragment, others write the texture back and forth between two framebuffers that reduce in size to then upscale the minified texture, or it is even possible to simply sample the unblurred texture from a smaller mipmap with the proper filtering parameters to obtain a blurring effect.
In this engine's case, the method is very simple: During the post-processing pass, once all the shading has been performed once per pixel during the deferred shading pass, a simple mean blur is applied to the brights texture and then added to the post-process pass' result before tone mapping and gamma correction.
```
void main()
{
    vec3 result = MeanBlur();
    result = ExtendedReinhard(result, MaxBrightness);
    result = pow(result, vec3(1.0/GAMMA));
    FragColor = vec4(result, 1.0);
}
```

This gives satisfying results and simply adds a small extra step into the existing rendering pipeline without the need to switch render targets, without generating mipmaps and without performing the blur on a per-fragment basis, everything written to the brights texture will be used for the final result.

### Shadow mapping
This demo also has three brick textured spheres that cast shadows on each other. To achieve this, a simple shadow map for a directional light was used.
The idea is simple: draw the scene once before the lighting pass and keep only the depth values but with the scene seen from the directional light's point of view. Once at the shading stage, compare the pixel's current depth value with the one stored in the shadow map. If the fragment's current depth as seen from the light's perspective is greater than the one that is stored in the depth map at the same location, it means that the current fragment is in shadow.


<p align="left">
  <img width="240" height="210" src="../assets/shadowmap.png">
</p>

<p align="left">
  <img width="435" height="253" src="../assets/scene_spheres.png">
</p>

## The Demo
A .zip package is provided in the [release](https://github.com/LoshkinOleg/gameEngine/releases/tag/1) section of the repository containing a standalone demo that uses the rendering engine to showcase a simple scene.
### The demo's layout
Once the demo is launched, the camera automatically moves along the world's Z axis in the -Z direction. The camera rotates to show each of the 5 elements in the scene as the scene progresses. The whole demo lasts ~55 seconds and then resets.
#### 1. The morphing horse
A disembodied [horse's head](https://free3d.com/3d-model/a-horse-with-a-big-tush-498195.html) is shown to transition between it's default aspect and a "spherifyed" version of the mesh:

<p align="left">
  <img width="256" height="256" src="../assets/scene_horse.gif">
</p>

This is a demonstration of [shape interpolation](https://en.wikipedia.org/wiki/Morph_target_animation), a very simple technique where the values of a vertex are interpolated between two shapes with the exact same amount of vertices. The vertex shader simply interpolated between the two vertices to obtain the final vertex:

```
in vec3 Position0;
in vec3 Position1;
uniform float factor;
void main()
{
  gl_Position = vec4(mix(Position0, Position1, factor), 1.0);
}
```

#### 2. The particle system
A bunch of stars are shown to fly up into the air in a "trumpet" like shape:

<p align="left">
  <img width="563" height="639" src="../assets/scene_particles.png">
</p>

This is done with some simple instancing and trigonometric functions. Each particle's position is updated every frame and the buffer containing the positions of all the particles is sent over to the GPU once per frame. An instancing command is then issued and the particles are drawn using the new positions.

#### 3. The reflective diamond
A spinning diamond is shown to reflect the skybox. The reflected color is boosted to make the diamond appear brighter and cause some blooming effect when the bright sky is reflected.

<p align="left">
  <img width="337" height="273" src="../assets/scene_diamond.png">
</p>

The reflection is done via the use of glgl's built-in [reflect](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/reflect.xhtml) function, the resulting reflection vector being used to sample the skybox's cubemap.

#### 4. The shadow casting spheres
Three orbiting spheres are shown to cast shadows on each other as they orbit:

<p align="left">
  <img width="435" height="253" src="../assets/scene_spheres.png">
</p>

This is done via [shadowmapping](https://learnopengl.com/Advanced-Lighting/Shadows/Shadow-Mapping).

#### 5. The normalmapped cube
A brick textured cube is shown to reflect light according to a geometry that is more complex than that contained in it's .obj file:

<p align="left">
  <img width="428" height="370" src="../assets/scene_cube.png">
</p>

This is done via simple [normalmapping](https://learnopengl.com/Advanced-Lighting/Normal-Mapping).

### The demo's execution flow
The high level view of the program's flow is as such:

<p align="left">
  <img width="500" height="907" src="../assets/demo_flow.png">
</p>
