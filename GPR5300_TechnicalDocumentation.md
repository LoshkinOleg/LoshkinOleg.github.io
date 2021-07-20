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

## The Demo
A .zip package is provided in the [release](https://github.com/LoshkinOleg/gameEngine/releases/tag/1) section of the repository containing a standalone demo that uses the rendering engine to showcase a simple scene.
### The demo's layout
Once the demo is launched, the camera automatically moves along the world's Z axis in the -Z direction. The camera rotates to show each of the 5 elements in the scene as the scene progresses:
#### The morphing horse
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

#### The particle system
A bunch of stars are shown to fly up into the air in a "trumpet" like shape:

<p align="left">
  <img width="563" height="639" src="../assets/scene_particles.png">
</p>

This is done with some simple instancing and trigonometric functions. Each particle's position is updated every frame and the buffer containing the positions of all the particles is sent over to the GPU once per frame. An instancing command is then issued and the particles are drawn using the new positions.

#### The reflective diamond
A spinning diamond is shown to reflect the skybox. The reflected color is boosted to make the diamond appear brighter and cause some blooming effect when the bright sky is reflected.

<p align="left">
  <img width="337" height="273" src="../assets/scene_diamond.png">
</p>

The reflection is done via the use of glgl's built-in [reflect](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/reflect.xhtml) function, the resulting reflection vector being used to sample the skybox's cubemap.

#### The shadow casting spheres
Three orbiting spheres are shown to cast shadows on each other as they orbit:

<p align="left">
  <img width="435" height="253" src="../assets/scene_diamond.png">
</p>

This is done via [shadowmapping](https://learnopengl.com/Advanced-Lighting/Shadows/Shadow-Mapping).

#### The normalmapped cube
A brick textured cube is shown to reflect light according to a geometry that is more complex than that contained in it's .obj file:

<p align="left">
  <img width="428" height="370" src="../assets/scene_diamond.png">
</p>

This is done via simple [normalmapping](https://learnopengl.com/Advanced-Lighting/Normal-Mapping).
