# 4. Map construction: ProBuilder plugin
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
- In the ProBuilder window, click on "New Shape", choose the Plane, and set dimensions to X=10, Y=0, Z=10. Hold shift and click on the floor in the scene to create that object. Rename the resulting GameObject to "Floor 10x10".
    - Move the Plane GameObject from the Hierarchy to a new "Prefabs" folder within the Assets folder.
        - This converts the GameObject into a prefab, enabling global modification of all instances in the scene.
    - Extend the floor by duplicating (Ctrl+d) and positioning more planes.
- Repeat the process to create walls (using the Cube shape) and doors (using the Door shape). Assemble rooms and an exit out of the base. Convert those GameObjects into Prefabs.
- Use the New Poly Shape feature in the ProBuilder window to create custom shapes within the base. Convert those GameObjects into Prefabs.
- Organize all created shapes under a new empty GameObject named "Spawn Base" in the hierarchy.
- To fix the invisible door sides:
    - Double-click on a door prefab to open it.
    - At the top of the scene view select Edge Selection tool.
    - Hold Ctrl key and click on two edges you want to connect with a face (edges on the top of the doors).
    - In the ProBuilder window click on "Bridge Edges".
- Apply colors to the shapes:
    - Double-click on the desired prefab to open it.
    - Select the object in the scene view (first switch back to Object Selection from Edge Selection).
    - In the ProBuilder window, select "Vertex Colors" and apply the chosen color.
    - Return to the scene by clicking the back button in the hierarchy window.
- To prepare for interior lighting and darken the space:
    - Create a Plane covering the entire base (representing the ceiling) using the New Shape feature in ProBuilder. Rename it to "CeilingForShadow".
    - One side of that plane is transparent, we want that side to look up: in the Inspector set the Plane's Y scale to -1.
    - In the Mesh Renderer component of the Plane set property Cast Shadows to "Two Sided".

The final state could look something like this:
![](https://i.imgur.com/wr6noLR.png)

## Upstairs
- To create an upper floor (upstairs), we will require stairs or a lift.
- Temporarily disable CeilingForShadow GameObject.
- Construct stairs shape using ProBuilder:
    - Set shadow casting to "Two Sided" to resolve light line issues in its shadow.
    - To enable player walking up the steps: lower the height of each step (or increase step count) or include an invisible ramp (collider) for smooth ascent.
- Organize all shapes from the ground floor (0th floor) under a newly created empty GameObject named "Floor 0."
- Duplicate floor tiles from the ground floor and reposition them for the 1st floor:
    - Remove tiles covering the stairs.
    - Fill in missing floor sections around the stairs.
- Set the layer of floor tiles on the 1st floor to "Ground" for aim target raycasting.
- Organize all shapes from the 1st floor under a newly created empty GameObject named "Floor 1."

### Upstairs visibility
- There are two primary methods to determine if the player is on the upper floor:
    - Based on the player's Y position.
    - Checking if the player is within a collider placed on the 1st floor.
- Create a new C# script and attach it to the Spawn Base GameObject.
- Below is the script using the player's Y position to identify if they are on the 1st floor:
    - Set the Floor 1 and Player fields for the script component in the Base Spawn GameObject.

```c#
using UnityEngine;

public class FloorVisibility : MonoBehaviour
{
    public GameObject player; // Reference to the player GameObject
    public GameObject floor1; // Reference to the parent GameObject representing the upper floor

    void Update()
    {
        if (IsPlayerOnFloor1())
            floor1.SetActive(true);
        else
            floor1.SetActive(false);
    }

    bool IsPlayerOnFloor1()
    {
        // Check if the player's Y position is greater than the upper floor's Y position
        return player.transform.position.y > 10;
    }
}
```

- The second approach involves using colliders to detect the player's presence on the 1st floor or stairs.
- Create a cube and adjust its position and scale to cover the entire 1st floor area:
    - Disable its Mesh Renderer component and enable the "Is Trigger" field (we do not want the player to collide with the cube, only trigger it).
- Repeat the process for a cube positioned on the stairs.
- Group these cubes under a parent empty GameObject named "Floor1Trigger."
- The script below checks if the player's position is within one of these colliders:

```c#
using UnityEngine;

public class FloorVisibility : MonoBehaviour
{
    public GameObject player; // Reference to the player GameObject
    public GameObject floor1; // Reference to the parent GameObject representing the upper floor
    public GameObject floor1Trigger; // Reference to the parent GameObject containing all the triggers for the upper floor

    void Update()
    {
        if (IsPlayerOnFloor1())
            floor1.SetActive(true);
        else
            floor1.SetActive(false);
    }

    bool IsPlayerOnFloor1()
    {
        // Get all colliders under the floor1Trigger GameObject
        Collider[] colliders = floor1Trigger.GetComponentsInChildren<Collider>();

        // Check if player is inside of any of the upper floor colliders
        foreach (Collider collider in colliders)
        {
            if (collider.bounds.Contains(player.transform.position))
                return true;
        }
        return false;
    }
}
```

- Optionally, instead of deactivating objects on the 1st floor, we might want to disable their rendering:

```c#
using UnityEngine;

public class FloorVisibility : MonoBehaviour
{
    public GameObject player; // Reference to the player GameObject
    public GameObject floor1; // Reference to the parent GameObject representing the upper floor
    public GameObject floor1Trigger; // Reference to the parent GameObject containing all the triggers for the upper floor

    void Update()
    {
        if (IsPlayerOnFloor1())
            SetFloorVisibility(floor1, true);
        else
            SetFloorVisibility(floor1, false);
    }

    bool IsPlayerOnFloor1()
    {
        // Get all colliders under the floor1Trigger GameObject
        Collider[] colliders = floor1Trigger.GetComponentsInChildren<Collider>();

        // Check if player is inside of any of the upper floor colliders
        foreach (Collider collider in colliders)
        {
            if (collider.bounds.Contains(player.transform.position))
                return true;
        }
        return false;
    }

    void SetFloorVisibility(GameObject floor, bool visible)
    {
        // Get all renderers in the floor object
        Renderer[] renderers = floor.GetComponentsInChildren<Renderer>();

        // Set visibility for each renderer
        foreach (Renderer renderer in renderers)
        {
            renderer.enabled = visible;
        }
    }
}
```

- Currently, we hide the 1st floor visually, but it's still there when the player is on the ground floor.
    - However, there's a problem: the crosshair raycast hits the 1st floor tiles even when the player is downstairs.
    - To fix this, we can change the 1st floor's layer depending on where the player is.
    - First, group all 1st floor tiles (floor planes) under a new parent object named "Floor1Tiles."
    - The script below changes the layer of each tile depending on the player's position (in the new method `SetFloorLayer`):

```c#
using UnityEngine;

public class FloorVisibility : MonoBehaviour
{
    public GameObject player; // Reference to the player GameObject
    public GameObject floor1; // Reference to the parent GameObject representing the upper floor
    public GameObject floor1Trigger; // Reference to the parent GameObject containing all the triggers for the upper floor
    public GameObject floor1Tiles; // Reference to the parent GameObject representing the floor tiles

    void Update()
    {
        if (IsPlayerOnFloor1())
        {
            SetFloorVisibility(floor1, true);
            SetFloorLayer(floor1Tiles, "Ground");
        }
        else
        {
            SetFloorVisibility(floor1, false);
            SetFloorLayer(floor1Tiles, "Default");
        }
    }

    bool IsPlayerOnFloor1()
    {
        // Get all colliders under the floor1Trigger GameObject
        Collider[] colliders = floor1Trigger.GetComponentsInChildren<Collider>();

        // Check if any of the player's colliders are overlapping with any of the upper floor colliders
        foreach (Collider collider in colliders)
        {
            if (collider.bounds.Contains(player.transform.position))
                return true;
        }
        return false;
    }

    void SetFloorVisibility(GameObject floor, bool visible)
    {
        // Get all renderers in the floor object
        Renderer[] renderers = floor.GetComponentsInChildren<Renderer>();

        // Set visibility for each renderer
        foreach (Renderer renderer in renderers)
        {
            renderer.enabled = visible;
        }
    }

    // Function to change the layer of all child objects under a specified parent GameObject
    public void SetFloorLayer(GameObject floorTiles, string layerName)
    {
        // Find the layer index based on the layer name
        int layerIndex = LayerMask.NameToLayer(layerName);

        // Iterate through all child objects of the specified parent GameObject
        foreach (Transform child in floorTiles.transform)
        {
            // Change the layer of each child object to the specified layer
            child.gameObject.layer = layerIndex;
        }
    }
}
```

- Rename "CeilingForShadow" GameObject to "CeilingForShadow0," duplicate it, and rename the copy to "CeilingForShadow1." Reposition the copy appropriately for the 1st-floor ceiling and place each ceiling in the corresponding parent GameObject (Floor 0 or Floor 1).

---

<div align="center"><b>
  <a href="3-Terrain.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
  <a href="5-PlayerInteraction.html" style="font-size:64px; text-decoration:none"> > </a>
</b></div>
