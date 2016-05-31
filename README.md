I just wanted to share an idea here quickly. This is about "VR video" that allows for stereoscopic rendering from both eyes and free rotation and positioning in a limited viewing area (e.g. a 1m cube) for viewing static or movie scenes. Not sure if this will work - but if someone wants to steal this idea and run with it you are welcome (just don't even think about patenting it, defensive publication on [reddit](https://www.reddit.com/r/oculusdev/comments/4lx0um/sparse_froxel_octree_rendering_for_vr_video/) and [github](https://github.com/DejayRezme/SparseFroxelOctreeRendering) on the 3/31/2016). 

**TL;DR:** VR needs a new truly 3D video format, compression and rendering algorithm that allows position head tracking

I proudly (or foolishly in case this doesn't work) present:

#Sparse Froxel Octree Rendering for VR video

Froxels are like voxels, but they are a subdivision of the viewing frustrum into truncated pyramids (in 6 directions around the origin). Basically you represent the scene as a layer of semi transparent cubemaps. Since you don't want to store all the empty or hidden froxels, you store them in an sparse octree structure, and throw away all froxels that you can't see from the viewing area.

The advantage of this structure is that it is easier to traverse and sort for cache alignment. Hence it should be faster to render or raytrace than a generic sparse voxel octree. Theoretically this should not be that much more memory intensive than a normal image, maybe in the order of 2 and betwee 10 times more data (depth information, semi transparent pixels and pixels for parallax). It might also be easier to compress.

You could rasterize this structure with a giant number of transparent cubemaps. You could generate polygons to only render non transparent pixels of the cube maps. You could use texture lookups like mega textures do and optimize them for video memory cache alignment.

You could also 

##Advantages / disadvantages:

* Free Order-dependent transparency 
* Can use existing rasterization hardware or raytracing
* Sparse texture lookup
* Can correctly represent filtered, fuzzy or semitransparent objects like fur or vegetation
* Cannot easily render objects correctly close to you / inside the viewing area

##Algorithm:

* Convert a 3D scene (Polygons + textures, SVO, photogrammetry) into a sparse froxel octree.
* Raytrace / raymarch / cone trace through the sparse froxel octree - but rasterization might be faster.
* Distortion pass for VR lenses maybe could be skipped (chroma aberration?)
* If rasterizing:
* Generate quads / complex polygons instead of full mapped cubes, could also be skewed up to the max possible viewing angle from the viewing area. This wouldn't be an octree but a sorted froxel tree
* Generate sparse 3D texture and UV mapping for polygons
* Potentially use parallax mapping to render hard surfaces, or use algorithms for parallax mapping to render multiple "cubemap layers" at once. Additional depth value might be more efficient and less data than multiple layers
* Maybe use distance field to reuse textures for shells of fur rendering? Might already be integral to something like a 3D wavelet compression
* Can render correctly / performant only for a limited viewing area e.g. 1m or 2m cube - otherwise you could see cracks and other artifacts
* Use image or video compression like wavelet / jpeg / x265 to store the sparse texture on disk
* Polygons represent "blocks" in the sparse texture that can be animated / transformed as well - might need to adapt the motion estimation part of an video compression algorithm, but could still decompress using existing algorithms (e.g. hardware accelerated codecs) - block movement would be an additional data stream in video files
* Rotation or scaling of blocks would be very cheap. Rotation of objects more complicated. Might be able to rotate 3D wavelets representing the object in 3D?
* You'd want an unencumbered open source codec implementation and an open data format to further innovation

Like I said this is just an idea that I don't know will work or whether it will be useful, or what image quality and performance you could get or what compression ratios you could achieve. But ideally I'd like to see a free "3D video codec" for true 3D scenes for VR video instead of the 180° or 360° stereoscopic videos. And after reading about those lightfields or that MS flashback thingy I think they are approaching the problem all wrong. Or maybe this is very much in the same vein and I'm just reinventing the wheel here and making a fool out of myself.

Credit obviously goes to Alex Evans or whoever else came up with the idea of froxels (someone from frostbite?).

# Video photogrammetry

Another idea I had to generate SVOs and VR video is to use video photogrammetry (videogrammetry?). You could film VR video using a 1m cube with wide angle cameras at the corners. But instead of using polygons and textures to representing the resulting 3D scene, you represent it directly with a sparse froxel octree. The biggest advantage is that you could represent semi transparent objects and filter objects that are too fine to represent with polygons - for example hair, vegetation like grass, fur. I only have cursory knowledge about computer vision and photogrammetry, maybe this is much easier than I think it is to resolve this into a point cloud and convert that into a SFRO.

But another way to do this might be to define a multidimensional error function of the rendering equation of every ray from every pixel of the captured images and what foxels they are hitting as input parameters. Then solve it using global optimization, least squares or LBFGS and random restart. Obviously this is prohibitively bonkers but you could simply downsample all the images and the froxel octree to a low resolution (mip mapping solves everything!) and then first solve that, then subdivide the images and froxel octree and resolve it in higher resolution - using the lower resolution solution of the previous step as a starting point of the optimization algorithm, and peeling the onion from the inside out to resolve overlaps. For following video frames you could use the previous froxel octree as a starting point of your solution. Theoretically you might even be able to extract lighting from the shading of surfaces, since the rendering equation could include cone tracing to represent specular highlights and global illumination. Obviously nothing of this is simple but with massive parallelization on a GPU or a cluster of GPUs this could be fast. 

Keywords: sparse froxel octree rendering / raytracing, SFRO, cone tracing, VR video, 4D photogrammetry, mip mapped, streaming 360 3D video, sparse voxel octree / SVO wavelet compression.
