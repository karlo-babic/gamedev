# 1. Init
- [Manual: Unity's interface](https://docs.unity3d.com/2021.3/Documentation/Manual/UsingTheEditor.html)

## New project
- Open Unity Hub.
- If Unity editor is not detected: in the "Installs" tab, click on "Locate", navigate to `C:\Program Files\Unity 2021.3.18f1\Editor`, and select "Unity.exe".
- In Unity Hub, click on "New project".
- Choose the correct Unity version, select "3D", set project name, and click on "Create project".
- The Unity editor should open in a few minutes.

## Scene initial setup
- By default, the scene contains the Main Camera and Directional Light.
- Add GameObjects: Capsule and Plane (right-click in the Hierarchy window -> 3D Object).
    - The Capsule will serve as the player, and the Plane will serve as the floor (rename it to "Floor").
![](https://i.imgur.com/L9BSsrD.png)

## Player initial setup
- Add an empty GameObject, rename it to "Player", and set the Capsule as its child.
- Set the Capsule's position to 0, 0, 0 (relative to its parent, Player), and reposition "Player" back to the center of the plane (above it).
- Gravity and Collision:
    - Select the Player GameObject and in the Inspector window, click on "Add Component".
        - Search for the "Rigidbody" component (not the 2D version) and add it.
        - Both the Player and the floor need colliders to prevent the player from passing through the floor. They should already have them (Capsule has a collider by default, as does the Plane).
    - Click on the play button to start the game.
        - The player should fall to the floor and collide with it.
    - To prevent the capsule from falling over upon collision, select the Player GameObject, go to the Inspector, expand Constraints, and select Freeze Rotation for the X and Z axes.

## Player initial controller (movement)
- In the Project window, right-click on the Assets folder and create a new folder named "Scripts".
- In the Scripts folder, create a new C# script named "PlayerController".
- Drag and drop the script into the Inspector window for the Player (adding the Player Controller component to that GameObject).
- Double click on the script to open it in the editor. Paste the following code:

```c#
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    // Speed at which the player moves
    public float moveSpeed = 5f;

    // Update is called once per frame
    void Update()
    {
        // Get input from the player
        float horizontalInput = Input.GetAxis("Horizontal");
        float verticalInput = Input.GetAxis("Vertical");

        // Calculate the movement direction based on input
        Vector3 moveDirection = new Vector3(horizontalInput, 0f, verticalInput).normalized;

        // Move the player based on the input direction
        transform.Translate(moveDirection * moveSpeed * Time.deltaTime);
    }
}
```

- Save the script and wait for Unity to reload it.
- The moveSpeed variable will be visible in the inspector as a property (in the Player Controller component in Player GameObject) because it is set as public.
- You should now be able to move the player using the WASD and/or arrow keys.

---

<div align="center"><b>
  <a href="Software.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
</b></div>
