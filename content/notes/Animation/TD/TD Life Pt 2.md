---
tags:
  - feature-animation
  - technical-director
---

## Quick and dirty computer graphics

Of course, the really fun part about being a TD, at least for me, was when you got to do quick and dirty graphics programming to solve some problem that R&D couldn't possibly get to in time or couldn't easily justify doing for a one-off production. The interesting thing about TDs at DWA is that they often filled in for a lack of R&D resources, especially on more off the wall productions.

One such production was the now-cancelled Monkeys of Mumbai, which was my last film as a "regular" TD back in 2014 or so. Monkeys was an incredibly ambitious movie, and the crew wanted to be even more ambitious by procedurally generating a fictional version of the city of Mumbai with an incredibly small number of artists. Surprisingly, this worked rather well, but texturing it and scouting camera angles to see how the procedural asset layout looked were separate problems.

### Absolute Approximate Curvature

Our timing on texturing was tricky; at the time, DWA was on the industry standard Mari. Mari was awesome at projection painting, but (at least back then) lacked much in the way of procedural texturing functionality, unlike Substance Designer/Painter. At the time, Substance hadn't really made any headway in the film industry, though that has changed since. Luckily, you could write shaders for Mari in GLSL that could be rasterized down to textures, and if you were clever you could enable artistic control by driving them with their mask. My development process here was to walk over to an artist on the film, ask what they needed, and implement as quickly as humanly possible. 

One fun and notable shader was a curvature shader. On the surface this sounds really simple; curvature isn't that hard to compute if you have mesh data for the geometry surrounding any given fragment of the surface. But in a GLSL fragment shader in Mari, you *don't* have that data - you know your current normal, but not the normals surrounding your fragment. Since surface curvature can be thought of as the spatial derivative of surface normals, you're out of luck.

There was an obvious, view-dependent workaround, which was intended to help with visualizing materials in the Mari viewport. However, if you try to use one that technique for calculating curvature, it'll be horribly wrong when you bake down your shader to write out textures; if I recall correctly, the bake would only happen from a single view of the asset, effectively projecting your view dependent technique to only look right from a single angle. This was unfortunate, because curvature makes an awesome mask in a stack of procedural textures, allowing artists to easily apply weathering on the edges of an object.

While scratching my head about how to tackle this, I realized that Mari had a non-shader technique that could get me information about surrounding fragment's features; namely, Mari had an offline blur function, and there was a procedural texture to visualize normals that baked down rather nicely. I came up with a hacky workaround where a script baked the normals, blurred that baked layer, and fed the results into a shader that could approximate the change in curvature based on the difference between the blurred normal and the normal in the fragment. While this didn't allow for properly calculating if the normal was convex or concave, it worked well enough for the artists to use to great effect.

I never bothered to publish the technique, because it's such a specific limitation to the application, and because it couldn't truly differentiate convex vs concave curvature. I don't think it was used again after Monkeys of Mumbai, but that's totally fine; that's the joy of being a TD.

### Scouting via the joy of octrees

As I mentioned before, we were procedurally generating the of Mumbai for the movie; more accurately, we were procedurally laying it out, combining assets like buildings, vehicles, props, and so on to rapidly fill in the city. It worked pretty well, though sadly I had nothing to do with this part despite my long-standing interest in procedural scene generation. Instead, I helped with an adjacent problem.

You see, when you put assets together in a scene, creative leadership needs to review those assets to make sure everything is working together as a whole, especially when it's for assets that will be close to camera. This is pretty easy in manually constructed scenes, because the artist just manually places a couple of cameras to render from so leadership can see every angle that matters. If you're procedurally generating a city for the camera to fly over, this probably isn't a big challenge. You can just stick a camera in the sky, point it at the ground, and call it a day.

What if the camera is going through the city, however? What if the Layout department is going to look through the city to find places for the characters to be, for scenes to take place? If you only wait for Layout to scout places manually, you'll likely have a lot of incongruent assets that you don't detect until later in the production process. That could be an issue, because it means that iterations to fix these sorts of issues are dependent on Layout scouting them, expanding the iteration loop and potentially wasting artist time.

You can just randomly place assets together and then render them, but we had thousands of assets, some of which we'd imported from older films, and it was hard to notice certain issues in a randomized context. Creative leadership mentioned this problem, and I had an idea to supplement their existing renders with plausible camera angles in a scene.

To do this, I calculated an octree for the city, though to save myself some computation time & memory I only calculated based on bounding boxes above a certain size. In computer graphics, octrees recursively subdivide 3D space by partitioning it into quadrants. If I recall correctly, I created a point region octree (but don't quote me on this, it's been a decade and I don't have my notes) so I could easily separate empty space from filled space. More importantly, I could calculate regions where empty space had filled neighbors. 

I sampled these regions at different places within the octree, and filtered based on some heuristics for where there *probably* were interesting groups of assets. I'd then plop a camera in the empty space, point it at the densest set of neighbors, and render. This wasn't perfect; I didn't have much time to tune my heuristics, so sometimes you got a render of a boring wall and just the slightest bit of a car. Still, this proved much faster than scouting for unique camera angles by hand, which was a tedious process that required a heavy duty workstation, so having about 7-12% of renders be useless was considered a success.