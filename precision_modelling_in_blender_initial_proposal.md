# Precision Modelling in Blender [Initial Proposal]

In this document I'm writing an initial proposal for how precision modelling could be integrated natively into Blender. As a disclaimer, I have **virtually no experience** with precision modelling and CAD in general, besides doing a single simple project with [CAD Sketcher](https://www.cadsketcher.com/). So everything I say here may be obviously wrong to more experienced people.

I found that it might be useful to write an initial proposal before I expose myself to the more standard way of doing things. I'm coming from the perspective of knowing Blender and wanting to do some precision modelling instead of the other other way around (knowing CAD and wanting to use Blender for it). 

## Scheme Geometry

At the core of the proposal is a new geometry and object type: the `Scheme`. This is what it often called a sketch in CAD software. It's purpose is to let the user specify real-world constraints as a basic for geometry generation.

### Name

I picked the term "scheme" because it's the one I liked best for now. Potential alternative names could be:
* Blueprint: Nice name but I didn't choose it because it also has other meanings and might be confused with a specific rendering style or the node system in Unreal Engine.
* Specification: An accurate name but is annoying to say too often.
* Constraint: Also an accurate name but can easily be confused with other existing constraints in Blender.
* Sketch: In the context of Blender the term "sketch" feels like it means exactly the opposite of what its supposed to mean (a rough and inexact drawing).
* Draft: A nice name, but has a similar problem like "sketch" and might also conflict with upcoming drafts in the asset system.
* Plan: Could work but I liked "scheme" more.

### Content of a Scheme

A scheme contains a set of `entities` and `constraints`. For example, an entity can be a point, line, plane or arc. A constraint could be a specified distance between two points. For a more exhaustive list, check the set of entities and constraints in [solvespace](https://github.com/solvespace/solvespace/blob/master/exposed/DOC.txt).

Vertices in a mesh are identified by their index. This does not work here. Instead, each entitity and constraint has an identifier as well as a user-editable name. This is necessary, because these elements are referenced later on.

### Manual Editing

The editing experience of a scheme would in many ways be like what [CAD Sketcher](https://www.cadsketcher.com/) does with some differences. For example, a scheme can contain multiple planes that points can be snapped to.

For reusability it's also important to be able to link one scheme geometry into another. That even works when the original scheme still has some degrees of freedom. When linking one scheme into another, all its entities and constraints become part of the bigger scheme. It's possible to create constraints and entities involving points from separately linked schemes.

## Geometry Nodes Integration

A scheme itself does not have a volume and is not renderable by e.g. EEVEE and Cycles. Geometry nodes is used to generate geometry that can be used for manifacturing or rendering.

The integration into geometry nodes starts with a new scheme component type.

![image](https://hackmd.io/_uploads/r1PUvpN46.png)

### Retrieving Scheme Data

The first way to futher process scheme data is to simply convert entities to curves. The 2D variant of the node would output curves on the XY plane from the local space of the given plane. The selection can be based on attributes stored on the entities.

![image](https://hackmd.io/_uploads/H1S-2R4Vp.png)

Another way to retrieve data is to get the positions of various entities. For example the position of a point.

![image](https://hackmd.io/_uploads/H1mBCRE46.png)

The point and circle ID are identifiers used by the scheme internally. One can retrieve these IDs by name. We could also support showing a dropdown menu that shows the names inside of the integer socket.

![image](https://hackmd.io/_uploads/HydyyyBNa.png)

It's also possible to get for example the start and end point IDs from a line ID.

![image](https://hackmd.io/_uploads/HJ_xxJSN6.png)

Using these nodes it's also possible to build higher level nodes that generate primitive meshes based on the scheme. Those primitives could then be combined using boolean nodes for example.

![image](https://hackmd.io/_uploads/SJ8x-1BNa.png)

### Procedural Schemes

Modifying existing schemes or creating entirely new ones seems useful as well. For example, one might want to generate multiple variations of a model where a single distance constraint is updated for each variation. Alternatively, one might want to create a procedural scheme based on some settings.

Some basic nodes to add to a scheme would just insert additional points and lines. Other than with e.g. meshes, it seems more reasonable here to add these entities one by one. That's because one has to be a bit more careful with adding too many, because that would make the constraint solver quite slow. Also, ideally every constraint would have a unique purpose, so creating many constraints which are all very similar might not be the ideal workflow. It would be more efficient to create e.g. repeating patterns outside of the scheme and to just use the scheme to compute the dimensions of the pattern.

![image](https://hackmd.io/_uploads/SJ41NyrEa.png)

Existing entities and constraints can be updated as well.

![image](https://hackmd.io/_uploads/BJUDS1HNa.png)
