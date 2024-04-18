# 5. Player interaction: interacting with the environment
- This section focuses on player movement and interactions. We will cover:
    - player rotation,
    - door opening (automatic and with a button).
- Interactions in games:
    - NPC interaction: [Red Dead Redemption 2](https://www.youtube.com/watch?v=Dw_oH5oiUSE)
    - Physics-based interaction (VR): [Tundra Fight School](https://www.youtube.com/watch?v=T_m6oEmPj8I), [Nimso Studios](https://www.youtube.com/watch?v=tMDmwD1cQPg)
    - World interaction: [Teardown](https://www.youtube.com/watch?v=ttwBelIlLv8), [Noita](https://www.youtube.com/watch?v=0cDkmQ0F0Jw)

## Player rotation
- The player character should face the direction of the crosshair (aim target).
- Start by creating a Cylinder and make it a child of the Player GameObject.
    - Adjust its size and position to resemble a gun pointing in the same direction as the player.
    - Disable its collider in the inspector.
- Update the PlayerController script so the player rotates toward the crosshair:

```c#
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    public float moveSpeed = 5f;
    public Transform crosshair; // Reference to the crosshair object (aim target)

    void Update()
    {
        float horizontalInput = Input.GetAxisRaw("Horizontal");
        float verticalInput = Input.GetAxisRaw("Vertical");

        Vector3 moveDirection = new Vector3(horizontalInput, 0f, verticalInput).normalized;

        transform.Translate(moveDirection * moveSpeed * Time.deltaTime);


        // Rotate the player to face the crosshair
        Vector3 lookDirection = crosshair.position - transform.position;
        lookDirection.y = 0f; // Ensure the player doesn't tilt up or down
        if (lookDirection != Vector3.zero)
        {
            Quaternion targetRotation = Quaternion.LookRotation(lookDirection);
            transform.rotation = targetRotation;
        }
    }
}

```

- Slow down player rotation by gradually moving towards the target rotation:
    - `transform.rotation = Quaternion.Lerp(transform.rotation, targetRotation, Time.deltaTime * rotateSpeed);`
    - Set the `rotateSpeed` variable as a public float and experiment with different values.
    - Interactable video series about quaternions (very useful when working with rotations in 3D space): [https://eater.net/quaternions](https://eater.net/quaternions)
- Current issues include:
    - Player movement is relative to its rotation, but we want it to be absolute (pressing forward always moves the player in the same direction - north).
    - The minimap camera rotates with the player, which is undesirable.
    - When the player collides with a wall, they are suddenly pushed back, resulting in a repetitive cycle.
- To fix player movement, adjust the PlayerController.cs script to translate the player's position relative to world space:
    - Modify the relevant line: `transform.Translate(moveDirection * moveSpeed * Time.deltaTime, Space.World);`
    - Using `Space.World` ensures movement is based on world axes rather than the player's local axes.
- Solve the minimap camera issue by placing the "Minimap Camera" GameObject at the root of the scene hierarchy and implementing movement logic in a new script (similar to the Main Camera).
    - Create a new C# script "MinimapCameraMovement" (below) and set public variables in the inspector.

```c#
using UnityEngine;

public class MinimapCameraMovement : MonoBehaviour
{
    public Transform player;
    public float followSpeed = 4f;
    public Vector3 offset = new Vector3(0, 100, 0);

    void LateUpdate()
    {
        transform.position = Vector3.Lerp(transform.position, player.position + offset, followSpeed * Time.deltaTime);
    }
}
```

- To correct the wall collision/movement behavior, transfer transforms to the FixedUpdate method:

```c#
using UnityEngine;

public class PlayerController : MonoBehaviour
{
    public float moveSpeed = 5f;
    public Transform crosshair; // Reference to the crosshair object (aim target)
    public float rotateSpeed = 8f;

    private Vector3 moveDirection;

    void Update()
    {
        // Get input from the player
        float horizontalInput = Input.GetAxisRaw("Horizontal");
        float verticalInput = Input.GetAxisRaw("Vertical");

        // Calculate movement direction
        moveDirection = new Vector3(horizontalInput, 0f, verticalInput).normalized;
    }

    void FixedUpdate()
    {
        // Update player position
        transform.Translate(moveDirection * moveSpeed * Time.fixedDeltaTime, Space.World);

        // Rotate the player to face the crosshair
        Vector3 lookDirection = crosshair.position - transform.position;
        lookDirection.y = 0f; // Ensure the player doesn't tilt up or down
        if (lookDirection != Vector3.zero)
        {
            Quaternion targetRotation = Quaternion.LookRotation(lookDirection);
            transform.rotation = Quaternion.Lerp(transform.rotation, targetRotation, Time.fixedDeltaTime * rotateSpeed);
        }
    }
}
```

## Doors
- In this section, we'll:
    - Create doors that open automatically when the player approaches.
    - Design doors that open when a button is pressed.
    - Exercise with an elevator scenario.

### Automatic sliding doors
- Place a Cube inside a door frame in the spawn base (which will represent the door).
- Add a Collider component to the door frame as a trigger (colliding with it opens the door).
    - Adjust the Collider size to fit the desired trigger area and enable "Is Trigger."
- Create a new C# script "AutomaticDoorController" (below) and configure its public variables in the inspector.

```c#
using UnityEngine;

public class AutomaticDoorController : MonoBehaviour
{
    public Collider playerCollider; // Reference to the object that contains player's collider
    public GameObject door; // Reference to the sliding door object
    private Vector3 initialPosition; // Initial position of the door
    private Vector3 openPosition; // Target position when the door is open
    private Vector3 closedPosition; // Target position when the door is closed
    public float openSpeed = 2f; // Speed at which the door opens
    public float closeSpeed = 1f; // Speed at which the door closes

    private bool isPlayerInside = false; // Flag to track if the player is inside the trigger collider

    void Start()
    {
        initialPosition = door.transform.position;
        openPosition = initialPosition + Vector3.forward * 2.4f;
        closedPosition = initialPosition;
    }

    void Update()
    {
        if (isPlayerInside)
        {
            // Move the door gradually towards the open position
            door.transform.position = Vector3.Lerp(door.transform.position, openPosition, openSpeed * Time.deltaTime);
        }
        else
        {
            // Move the door gradually towards the closed position
            door.transform.position = Vector3.Lerp(door.transform.position, closedPosition, closeSpeed * Time.deltaTime);
        }
    }

    void OnTriggerEnter(Collider other)
    {
        if (other == playerCollider)
        {
            isPlayerInside = true; // Player entered the trigger collider
        }
    }

    void OnTriggerExit(Collider other)
    {
        if (other == playerCollider)
        {
            isPlayerInside = false; // Player exited the trigger collider
        }
    }
}
```

### Button-controlled doors
- Your objective is to create a door that opens when the player presses a nearby button.
    - The button activates when the player is close and presses "e" on the keyboard.
    - The door gradually opens upwards.
- Use the ButtonController script for the button and the DoorController script for the door.

```c#
using UnityEngine;

public class ButtonController : MonoBehaviour
{
    public GameObject door;
    public GameObject player;
    public float interactionRange = 2f; // The range within which the player can interact with the button

    void Update()
    {
        // Check if the player is pressing the "e" key
        if (Input.GetKeyDown(KeyCode.E))
        {
            // Check if the player is close to the button game object
            if (IsPlayerNearButton())
            {
                // Call a method on the door object to open it
                door.GetComponent<DoorController>().OpenDoor();
            }
        }
    }

    bool IsPlayerNearButton()
    {
        // YOUR CODE HERE
    }
}
```

```c#
using UnityEngine;
using System.Collections;

public class DoorController : MonoBehaviour
{
    // YOUR CODE HERE
    public float delayBeforeClose = 5f; // Delay before starting the door-closing process
    private bool doorOpening = false; // Flag to track if the door is opening

    void Start()
    {
        // YOUR CODE HERE
    }

    void Update()
    {
        // YOUR CODE HERE
    }

    public void OpenDoor()
    {
        Debug.Log("opening");
        doorOpening = true;

        // Start the coroutine for delaying the door closing
        StartCoroutine(DelayedClose());
    }

    IEnumerator DelayedClose()
    {
        // Wait for the specified delay before starting the door-closing process
        yield return new WaitForSeconds(delayBeforeClose);

        // Trigger the door-closing event
        CloseDoor();
    }

    public void CloseDoor()
    {
        Debug.Log("closing");
        doorOpening = false;
    }
}

```

## Elevator
- Implement an elevator that ascends to the 1st floor when the player presses the button in the elevator.

---

<div align="center"><b>
  <a href="4-ProBuilder.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
  <a href="6-Physics.html" style="font-size:64px; text-decoration:none"> > </a>
</b></div>
