# 4. ProBuilder
- [ProBuilder](https://unity.com/features/probuilder)
- [Unity Packages: ProBuilder](https://docs.unity3d.com/Packages/com.unity.probuilder@4.0/manual/index.html)
    - [Tutorials](https://www.youtube.com/@unity/search?query=Probuilder)
- This section focuses on map construction using the ProBuilder plugin.
- Examples of map building:
    - Games featuring maps built with ProBuilder: [SUPERHOT](https://www.youtube.com/watch?v=pzG7Wc6mbwE), [Tunic](https://www.youtube.com/watch?v=Q5XpgTO7YN0)
    - Map building for CS:GO: [video](https://www.youtube.com/watch?v=PygJXsD1XsE)

## ProBuilder plugin
- To begin, install the ProBuilder plugin:
    - Go to the top menu bar: Window > Package Manager.
    - In the Package Manager window, select Packages: Unity Registry, and search for "probuilder",
    - Install ProBuilder.
- Open the ProBuilder window by navigating to: Tools > ProBuilder > ProBuilder Window.

## Building the Spawn Base
- Constructing the spawn base (where the player spawns) involves creating floors, walls, and doors.
- For precise positioning of ProBuilder elements, enable Grid Snapping in the scene view by toggling the button with a grid and a magnet at the top left.
- In the ProBuilder window, click on "New Shape", choose the Plane, and set dimensions to X=10, Y=0.1, Z=10. Rename the resulting GameObject to "Floor 10x10".
    - Move the Plane GameObject from the Hierarchy to a new "Prefabs" folder within the Assets folder.
        - This converts the GameObject into a prefab, enabling global modification of all instances in the scene.
    - Extend the floor by duplicating and positioning more planes.
- Repeat the process to create walls (using the Cube shape) and doors (using the Door shape). Assemble rooms and an exit out of the base.
- Use the New Poly Shape feature in the ProBuilder window to create custom shapes within the base.
- Organize all created shapes under a new empty GameObject named "Spawn Base" in the hierarchy.
- To fix the invisible door sides:
    - Double-click on a door prefab to open it.
    - At the top of the scene view select Edge Selection tool.
    - Hold Ctrl key and click on two edges you want to connect with a face (edges on the top of the doors).
    - In the ProBuilder window click on "Bridge Edges".
- Apply colors to the shapes:
    - Double-click on the desired prefab to open it.
    - In the ProBuilder window, select "Vertex Colors" and apply the chosen color.
    - Return to the scene by clicking the back button in the hierarchy window.
- To prepare for interior lighting and darken the space:
    - Create a Plane covering the entire base (representing the ceiling) using the New Shape feature in ProBuilder.
    - One side of that plane is transparent, we want that side to look up: in the Inspector set the Plane's Y scale to -1.
    - In the Mesh Renderer component of the Plane set property Cast Shadows to "Two Sided".

The final state could look something like this:
![](https://i.imgur.com/wr6noLR.png)

---

<div align="center"><b>
  <a href="3-Terrain.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
</b></div>
