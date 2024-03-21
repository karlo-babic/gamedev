# 2. Camera
- [Manual: Camera component](https://docs.unity3d.com/2021.3/Documentation/Manual/class-Camera.html)
- Here, we will cover multiple ways to implement camera movement:
    - Basic player camera (as Player child)
    - Scripted player camera (follow player)
    - Scripted mouse camera (follow in-between player and mouse)
    - Cinemachine (Unity plugin)
    - Minimap (render texture)
- Examples:
    - Basic first-person shooter camera ([Counter-Strike](https://www.youtube.com/watch?v=bTDJQYV5Q58))
    - Hyper-realistic first-person shooter camera ([Unrecord](https://www.youtube.com/watch?v=IK76q13Aqt0))
    - Third-person camera ([Tomb Raider](https://www.youtube.com/watch?v=vBCeSQSCrqY))
    - Top-down camera ([Age of Empires](https://www.youtube.com/watch?v=5TnynE3PuDE))
    - 2D/3D mix ([FEZ](https://www.youtube.com/watch?v=lrEsNI0aCPk))
    - Camera as a gameplay mechanic ([Viewfinder](https://www.youtube.com/watch?v=crteKROYkHM))

## Open the project
- If you download the project from the previous exercise...
    - In Unity Hub, click on "Add" and locate the project folder (unzipped).
    - When you load the project and the Unity editor opens, open the Scenes folder (in the Project window), and double-click the main scene to open it.

## Basic player camera
- Move the Main Camera upwards and rotate it slightly down.
- Set the Main Camera as a child of the Player.
![](https://i.imgur.com/8WwnwIl.png)

## Scripted player camera
- Keep the Main Camera in the root (not as a child of the Player).
- We will implement camera movement in a script.
    - Create a new script named "CameraMovement" and add it to the Main Camera GameObject.

### Scripted player camera: Basic follow
- We will first implement basic following (which will look the same as in the Basic camera setup):

```c#
using UnityEngine;

public class CameraMovement : MonoBehaviour
{
    // Reference to the player GameObject to follow
    public Transform player;

    // Offset between the camera and the player
    private Vector3 offset;

    // Called once when the script is enabled
    void Start()
    {
        // Calculate the initial offset between the camera and the player
        offset = transform.position - player.position;
    }

    // Called every frame after all Update functions have been called
    void LateUpdate()
    {
        // Update the camera position to directly follow the player
        transform.position = player.position + offset;
    }
}
```

- The Camera Movement component in the Main Camera GameObject now has a "Player" property (because of the line `public Transform player;`),
    - Drag and drop the Player GameObject into this field.
    - Now the player variable (in the script) references the Player's Transform component. We use it here to get the Player's position in the world.

### Scripted player camera: Smooth follow
- Here, we will modify the script so the camera doesn't follow the player instantly, but with a smooth delay:

```c#
using UnityEngine;

public class CameraMovement : MonoBehaviour
{
    // Reference to the player GameObject to follow
    public Transform player;

    // Speed at which the camera follows the player
    public float followSpeed = 4f;

    // Offset between the camera and the player
    private Vector3 offset;

    // Called once when the script is enabled
    void Start()
    {
        // Calculate the initial offset between the camera and the player
        offset = transform.position - player.position;
    }

    // Called every frame after all Update functions have been called
    void LateUpdate()
    {
        // Calculate the target position for the camera to move towards
        Vector3 targetPosition = player.position + offset;

        // Smoothly move the camera towards the target position using Lerp
        transform.position = Vector3.Lerp(transform.position, targetPosition, followSpeed * Time.deltaTime);
    }
}
```

## Scripted mouse camera
- It would make sense for the camera to follow a point between the player and the player's mouse cursor (crosshair). Then, the player will be able to look around by moving the mouse.

### Crosshair position
- First, we will have to implement crosshair placement (aim target).
    - We will translate from the cursor position on the screen to the crosshair position in the game world.
- Create a Sphere (a temporary "crosshair" - aim target visualization). Name it "Crosshair".
- Create a new script named "CrosshairPlacement" and place it into the Crosshair GameObject:

```c#
using UnityEngine;

public class CrosshairPlacement : MonoBehaviour
{
    void Update()
    {
        // Cast a ray from the camera through the mouse position
        Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        RaycastHit hit;

        if (Physics.Raycast(ray, out hit))
        {
            // If the ray hits something, place the crosshair at the hit point
            transform.position = hit.point;
        }
    }
}
```

- In this script, a ray is cast from the camera through the mouse cursor.
    - The position it hits is the position we want the crosshair to be at.
- To fix the problem with the Crosshair moving towards the camera when you stop moving the mouse, you can disable the Crosshair's Sphere Collider component (the ray was hitting that collider).

- We replace: `transform.position = hit.point;` with `transform.position = new Vector3(hit.point.x, hit.point.y + 1.3f, hit.point.z);` so the crosshair hovers above the ground.
- We want the crosshair to hover above the ground, not above other colliders (e.g., Player's collider). To do that, we will add a new Layer named "Ground", and we will set it so the ray will be able to hit only the colliders in that layer.
    - Select the Floor, in Inspector open the "Layer" dropdown and click on "Add Layer", type "Ground" in an empty field. Now Floor is in that layer.

```c#
using UnityEngine;

public class CrosshairPlacement : MonoBehaviour
{
    // Layer mask for ground objects
    public LayerMask groundLayerMask;

    void Start()
    {
        //Set Cursor to not be visible
        //Cursor.visible = false;
    }

    void Update()
    {
        // Create a ray that starts from the main camera and passes through the mouse position on the screen
        Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
        // RaycastHit provides details about where the ray hits an object
        RaycastHit hit;

        // Cast a ray into the scene and check for collisions
        if (Physics.Raycast(ray, out hit, Mathf.Infinity, groundLayerMask))
        {
            // If the ray hits something, place the crosshair at the hit point
            transform.position = new Vector3(hit.point.x, hit.point.y+1.3f, hit.point.z);
        }
    }
}
```

- In the Crosshair GameObject (Inspector), component Crosshair Placement, set the "Ground Layer Mask" property to "Ground" (used in the script: `public LayerMask groundLayerMask;`).

### Follow in-between player and crosshair
- In the CameraMovement script, we add a reference for the Crosshair (to get its position) and implement the "PositionBetween" method that calculates the position between the Player and Crosshair GameObjects. The camera should follow that in-between point:

```c#
using UnityEngine;

public class CameraMovement : MonoBehaviour
{
    // References to the Player and Crosshair
    public Transform player;
    public Transform crosshair;

    public float followSpeed = 4f;

    private Vector3 offset;

    void Start()
    {
        offset = transform.position - player.position;
    }

    void LateUpdate()
    {
        // Calculate the target position for the camera to move towards
        Vector3 targetPosition = this.PositionBetween(player.position, crosshair.position, 0.4f) + offset;

        transform.position = Vector3.Lerp(transform.position, targetPosition, followSpeed * Time.deltaTime);
    }

    // Calculate the position between two given positions (between player and crosshair)
    public Vector3 PositionBetween(Vector3 startPosition, Vector3 endPosition, float fraction)
    {
        return Vector3.Lerp(startPosition, endPosition, fraction);
    }
}
```

## Cinemachine plugin
- [Plugin website](https://unity.com/unity/features/editor/art-and-design/cinemachine)
- [Youtube tutorial](https://www.youtube.com/playlist?list=PLX2vGYjWbI0TQpl4JdfEDNO1xK_I34y8P)

## Minimap
- We can create a minimap by setting up a camera that overlooks the player from above.

### Camera setup
- Add a new camera to the scene hierarchy and name it "Minimap Camera".
- Make it a child of the Player.
- Position it directly above the player (x=0, y=100, z=0) and rotate it downwards (x=90).
- Set its projection to "Orthographic" and adjust its Size to 20.
- Create a render texture: Right-click in Assets > Create > Render Texture, and name it "MinimapTexture".
- Assign the "MinimapTexture" as the "Target Texture" property in the Minimap Camera GameObject.

### UI Canvas
- Create a UI Canvas in the scene hierarchy and name it "UI Canvas".
- Inside the canvas, add a Raw Image by navigating to UI > Raw Image, and name it "Minimap".
    - You can preview UI elements during gameplay or switch to the "Game" tab in the editor.
- Set the Texture property of the Minimap to "MinimapTexture".
- Adjust the position of the Minimap to the top-right corner of the screen using the Rect Transform component:
    - Choose the "top-right" option from the Anchor Presets.
    - Set Pos X and Pos Y to -50.

Final state:
![](https://i.imgur.com/CLJpOcD.png)
*The floor is larger and I added a few 3D objects (a cube and a sphere).*

---

<div align="center"><b>
  <a href="1-Init.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
  <a href="3-Terrain.html" style="font-size:64px; text-decoration:none"> < </a>
</b></div>
