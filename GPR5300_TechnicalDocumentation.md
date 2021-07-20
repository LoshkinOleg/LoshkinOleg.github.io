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

## The demo
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
