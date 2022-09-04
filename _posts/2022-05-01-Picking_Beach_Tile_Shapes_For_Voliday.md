---
title: "Picking Beach Tiles Shapes For Voliday"
layout: post
date: 2022-05-01
categories: blogposts old_blogposts
permalink: /:categories/:title
---

## Some Context
As part of the 3rd bachelor year’s curriculum, students of the Games Programming section of the school are tasked with creating a video-game over the span of 7 months (end of september to end of april).
We settled on the creation of a city builder we've named Voliday which can be found here:
https://volcanoteam.itch.io/

The island’s beach tiles bordering water needed to be visually consistent with the geometry of the island as demonstrated below:
<img align="center" width="500" height="232" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/shoreline.png">

This means that any given beach tile could take one of the following aspects:
1. A slope
<img align="center" width="500" height="455" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/slope.png">
2. An external corner
<img align="center" width="478" height="500" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/externCorner.png">
3. Or an internal corner
<img align="center" width="500" height="466" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/internCorner.png">

For the sake of simplicity, beach tiles cannot be full cubes, they can only border water.

My need was to represent the following diagram representing a “neighborhood” of the grid with a struct.

<img align="center" width="500" height="500" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/neighborhood.png">

Below is the declaration of a struct composed of 9 GridCell references. GridCells hold a variable that defines the tile’s terrain type.
I have arranged the references in this order to mirror how a 2D for loop would traverse the grid: from (-X;-Y) to (+X;+Y). Since Unreal Engine uses the left hand rule for its XYZ coordinates with the +Z vector pointing UP, this results in the order shown below.

<img align="center" width="500" height="419" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/gridNeighborhood.png">

Additionally, I needed to transform this list of labeled references to a more useful data structure I would be able to work with more intuitively. So from this single neighborhood struct containing references to GridCells, I’ve created another struct, ordered in the same exact way that only contains the data determining whether the tile is a ground or water tile since this is the only determining factor:

<img align="center" width="500" height="419" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/gridNeighborhoodAdjacencies.png">

In hindsight, I could have simply used the boolean type instead of a custom enum since the resulting enum can only ever have one of two states:

<img align="center" width="500" height="300" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/eGridAdjacency.png">

More states were originally used until I realized that they were redondant.

With the adjacency data laid out in this way, I could implement the following algorithm to compute the correct mesh and rotation to pick for any given tile T within a neighborhood N:

```
T := current tile\n
N := 3x3 neighborhood of tiles centered on T\n
nW := number of water tiles in N\n
nG := number of ground tiles in N\n
BR := bottom right tile of the neighborhood\n
BM := bottom middle tile of the neighborhood\n
BL := bottom left tile of the neighborhood\n
MR := middle right tile of the neighborhood\n
MM := middle middle tile of the neighborhood\n
ML := middle left tile of the neighborhood\n
TR := top right tile of the neighborhood\n
TM := top middle tile of the neighborhood\n
TL := top left tile of the neighborhood\n

if (nW is 2) OR (2*nW is nG AND ((BM and TM are ground) OR (MR and ML are ground))):\n
    set T to slope\n
    if ML is water:\n
        rotate T by 90°\n
    elif TM is water:\n
        rotate T by 180°\n
    elif MR is water:\n
        rotate T by 270°\n
elif nW is not 1:\n
    set T to external corner\n
    if ML and TM are ground:\n
        rotate T by 270°\n
    elif BM and MR are ground:\n
        rotate T by 90°\n
    elif ML and BM are ground:\n
        rotate T by 180°\n
else:\n
    set T to internal corner\n
    if TL is water:\n
        rotate T by 90°\n
    elif TR is water:\n
        rotate T by 180°\n
    elif BR is water:\n
        rotate T by 270°\n
```

Let’s detail the 
```
if (nW is 2) OR (2*nW is nG AND ((BM and TM are ground) OR (MR and ML are ground)))
```
part of the algorithm. If this condition is true, the mesh of the tile is set to that of a slope.

Let’s start with the
```
(nW is 2)
```
part. If the number of water tiles within the neighborhood is 2, in the case of our game, that means that the mesh of the tile must be a slope. Consider the following cases:

<img align="center" width="500" height="226" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/adjacency0.png">

All of these are encountered in our game and for all of them C must be a slope mesh, rotated one way or another.
Here are some more neighborhood layouts that also fulfill this condition, however none of them are encountered in our game, so this isn’t a problem.

<img align="center" width="500" height="229" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/adjacency1.png">

Next, let’s explain:
```
(2*nW is nG AND ((BM and TM are ground) OR (MR and ML are ground))
```
Let’s start by considering the evaluation:
```
2*nW is nG
```
If the number of ground tiles is twice that of water tiles (or conversely the number of water tiles is half that of ground tiles, but a multiplication is generally a simpler operation for a computer), that means we could potentially be in the following situations:

<img align="center" width="500" height="107" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/adjacency2.png">

In all of them, we would like C to take the shape of a slope. However, if we left the condition as is, it would also attribute the slope shape to the following situations that do occur in our game:

<img align="center" width="500" height="105" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/adjacency3.png">

Therefore, we need to restrict our condition to only let the right cases pass. I’ve done so with the remaining part of the conditional:
```
((BM and TM are ground) OR (MR and ML are ground))
```

We know that C must be a ground tile since we’re only adjusting the shape of beach tiles. If we add the condition that either the cardinal neighbors on the X axis OR the cardinal neighbors on the Y axis must be ground tiles, it means that the condition will now filter off the four cases shown above since only the correct cases form a “shoreline” of ground whereas the cases above form a jagged edge, for which a slope isn’t the suitable shape.

With this deceivingly obtuse part of the algorithm explained, the rest is straight forward. For cases where the shape is a slope, we simply inspect one of the cardinal neighbors and based on the neighbor’s terrain type rotate the mesh accordingly. The angle of rotations depend on the handedness of the coordinate system and the rotation on the Z axis of the imported mesh. The values written in the algorithm are the ones suited for our case but would surely change in others.

There is one detail left to explain. The “elif nW is not 1” part of the algorithm. If the algorithm has arrived at this condition, it means that C cannot be a slope. All that is left is either the external or the internal corner shape to pick. But how do we know which one? Consider the following situations:

<img align="center" width="500" height="229" src="/Assets/Blogposts/Old_Blogposts/Picking_Beach_Tile_Shapes_For_Voliday/adjacency4.png">

For all of these, the correct shape for C is either an internal or an external corner. However, we can see that only for the cases where the number of water tiles is not 1 should the shape be that of an external tile. The rest must by deduction be internal tiles.

This simple algorithm would erroneously let pass many other adjacency situations, however for our game, only the ones specified here are encountered.

Conclusion:
The only real challenge of this task was the establishment and creation of a clear enough mental model of the problem. I had initially struggled to wrap my head around all the adjacency possibilities until I took a good hour on the week-end to draw out some possible cases on grid paper, at which point I had started to see a lot of symmetries and from there I could simply pick up on uniquely identifying features of the relevant situations and with that mental model in mind, I was able to design and implement this algorithm with ease.
