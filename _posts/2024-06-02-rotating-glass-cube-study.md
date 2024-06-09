---
layout: post
title: "Rotating Glass Cube Study"
image: /assets/2024-04-24-noise-opacity-glass-material.jpg
categories: cinema-4d, redshift, davinci-resolve
---

I've long been a fan of [Gleb Kuznetsov's work](https://dribbble.com/glebich). I'm insterested in learning how these 3D artifacts are created, so [I chose an example](https://dribbble.com/shots/7295607-Cube-for-Dark-mode-by-Milkinside) to try to recreate.

<video controls poster="/assets/2024-06-02-rotating-glass-cube-study.jpg">
  <source src="/assets/2024-06-02-rotating-glass-cube-study.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>

*My recreation of Gleb Kuznetsov's ["Cube for Dark mode by Milkinside"](https://dribbble.com/shots/7295607-Cube-for-Dark-mode-by-Milkinside)*

Outside of a little bit of color color adjustment in [DaVinci Resolve](https://www.blackmagicdesign.com/products/davinciresolve) after exporting, this was otherwise done entirely in [Cinema 4D](https://www.maxon.net/en/cinema-4d) and rendered with [Redshift](https://www.maxon.net/en/redshift).

## The Elements of the Scene

1. There's a sphere contained in a rotating cube.
2. The sphere has a procedural texture, probably using noise.
3. The sphere is seen as a different gradient color based on which face of the cube it's viewed through.

## Sphere Material

The sphere's texture was made using scaled up noise ([`Type: Stupl`](https://help.maxon.net/r3d/cinema/en-us/Content/html/Maxon+Noise.html)).

## Cube Material

Each of the cube's faces has one of three different gradients on it.

The `Transmisson > Weight` is set to the maximum of `1` to make the cube fully transparent. The `Reflection > Weight` is also set to `1` to emphasize the reflections of the sphere on the inner surface of the cube.

To prevent the cube's outer surface from reflecting light, which would make it more difficult to see through, the cube itself is excluded from our lighting (with the result being the only indirect light, reflected off of the sphere, illumates the cube).

A separate material was used to emphasize the cube's beveled edges which helps reinforce the structure of the cube.

## Mapping the Gradients to Faces

The most interesting part of the scene is mapping the gradients to the cube's faces. A [`TriPlanar`](https://help.maxon.net/r3d/3dsmax/en-us/Content/html/Tri+Planar.html) node was used to map the three gradients to the X, Y, and Z axis of the cube. The `TriPlanar` node has three possible `Project Space Type` settings: `World`, `Object`, or `Reference`. These determine how the texture is mapped to each axes:

- `World` uses world-space coordinates to figure out which image to display on each face. In other words, based on the cube's current orientation, each face of the cube gets the X, Y, or Z image.
- `Object` uses object-space coordinates this means that the gradients get mapped to X, Y, and Z based on the orientaton of the geometry without `Coordinates > Transforms` applied.
- `Reference` is similar to `Object` but uses a [reference pose](https://en.wikipedia.org/wiki/T-pose) to place the texture consistently regardless of how the model is animated.

None of these is exactly what we want:

- With `World`, as the cube rotates which gradient is displayed on which face occassionaly snaps to another gradient, as the orientation of a face suddently resolves to another axes.
- With `Object`, the gradient itself changes orientation as the cube rotates. If you look at the finished animation, you'll see that the gradient itself remains at the same orientation on the sphere even as the cube rotates.
- It's possible `Reference` could be configured to do what we want, but by default its behavior is similar to `Object` in that it keeps the faces nad orientation consistent as the model rotates relative to object space.

Since I couldn't find a setting that did exactly what I wanted, I used a [hack](https://en.wikipedia.org/wiki/Kludge#Computer_science) where I manually adjusted the `TriPlanar > Coordinates > Rotation` of each face with keyframes every second to re-align them so that the gradient is orientated vertically in world space.
