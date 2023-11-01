# Replacement-based Procedural Modelling [Proposal]

This document proposes a new workflow for procedural modelling which makes extensive use of a new `Replace Faces` node. The proposal still has some open ends that have to be solved going forward.

The general idea is that many objects can be modelled by starting with a primitive shape and then repeatedly replacing individual faces with new geometry that is embedded into the mesh.

![image.png](https://hackmd.io/_uploads/HkbzS51Q6.png)

This is quite different from traditional modelling where one rarely thinks about operations as replacing some geometry with new geometry. The problem with traditional modelling operators in a procedural context is that they usually have complex inputs that are hard to generate procedurally. For example, specifying where exactly an edge loop should be inserted on a mesh is easy in the 3d view by just hovering over a specific edge. Procedurally generating that input is hard. It's even harder for e.g. the knife operator.

Repeatedly replacing parts of a mesh more naturally fits into geometry node. It also makes it possible to build independent details of a mesh separately and to stitch them together afterwards.

![image.png](https://hackmd.io/_uploads/HkPGo9J76.png)

A reason for why a replacement-based workflow fits better into geometry may be that it better separates the "what" from the "where". For example, in the image above, one first builds the loop cut on a single plane and then inserts it on all selected faces. In traditional modelling, one would build the loop cuts directly on the target mesh. The separation makes it easier to think about both things independently. 

## Replace Faces

At the core of this workflow is a **Replace Faces** node. A couple additional nodes may be needed under som circumstances as well (e.g. Dissolve Edges).

The Replace Faces node removes selected faces and inserts other geometry at the same place. If the inserted geometry is a mesh, it can optionally be linked with the rest of the mesh. For example, the resulting mesh in the image above is a single manifold mesh island. The replacement geometry can also be point clouds and curves though.

The inserted geometry is deformed to the face. This deformation is not always well defined, especially for ngons because there are many possibilities. Maybe multiple deformation methods are needed for more complex cases, or we need the ability to customize this somehow.

![image.png](https://hackmd.io/_uploads/SyK_tpyQ6.png)

This image shows an initial design for the new node. It's likely that it has to be modified a bit as we learn more about different use cases. The general idea is that for every selected face, one instance is picked from the `Instances` input which is then inserted.

The `Root Corner` determines an "anchor" in each face. It determines the orientation of the inserted geometry. The image above shows the root corner on the selected faces.

### Deformation

The instances passed into the node are expected to be normalized. There are different ways this normalized space can be defined. One possibility is to define a standard triangle, quad and ngon with arbitrary vertex amount as the output of the `Mesh Circle` node.

If the replacement for a face is exactly the corresponding standard ngon (with the same number of vertices), the mesh is not changed at all. If the replacement is something else, the output geometry changes.

Points that are not exactly on the XY-plane will be moved on the direction of the face (corner) normal.

![image.png](https://hackmd.io/_uploads/B1Moia1QT.png)

### Merging

The most difficult aspect of this node to get right is merging. The general idea is that all points that are exactly on an edge of the standard ngon or above/below that may be merged with points on neighboring replacement geometry if the points are close. Points lying on the edge may also be merged with faces that are not replaced.

Some examples of different situations:
* ![](https://hackmd.io/_uploads/S1t7HmOWp.png)
* ![](https://hackmd.io/_uploads/BkwLHX_-a.png)
* ![](https://hackmd.io/_uploads/Hy1oSX_W6.png)
* ![](https://hackmd.io/_uploads/rJ2xLQu-p.png)
* ![](https://hackmd.io/_uploads/BJbPUmOZa.png)


### Further Thoughts

* Sometimes it may be useful to skip certain corners when deciding which standard ngon to use. In the example below, the face on the left could be treated as quad under some circumstances.
    ![image.png](https://hackmd.io/_uploads/r1ld_0kQp.png)
* Sometimes there may be some points that can be merged by the node but should not.
* Often it may be useful to remove edges and faces that are created between two replacement geometries. For example, replacing every face in a quad mesh with a cube could then behave as a solidify node.
* It's not entirely clear whether materials should be propagated from the face or the replacement geometry. Both seem reasonable depending on the goal.
* A node to dissolve geometry (similar to e.g. the Dissolve Edges operator) could be combined with the Replace Faces node in powerful ways. Together they can be used to replace patches of faces on a mesh with new topology.
* There are different ways to propagate attributes from the face to the replacement geometry. For example, when the replacement geometry contains curves, the attributes should be propagated from the face to the points or curves domain.