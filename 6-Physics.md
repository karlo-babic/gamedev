# 6. Physics: forces and materials
- This section focuses on applying forces to objects, utilizing physics materials, and enhancing the player controller with physics-based movement.
- Examples:
    - [Half-Life 2](https://www.youtube.com/watch?v=UKA7JkV51Jw)
    - [Human: Fall Flat](https://www.youtube.com/watch?v=-Edk59BqSEU)
    - [Blade and Sorcery](https://www.youtube.com/watch?v=HeECulIwm6E)

## Wall explosion (forces)
- In this section, we'll set up a scenario where a wall is partially broken, and an explosive barrel causes the broken pieces to be forced away, creating an opening.
- Replace a section of the wall with a partially broken version.
    - Attach a Rigidbody component to each smaller broken piece intended to be propelled by the explosion.
        - Initially, set "Is Kinematic" to true to disable physics (this will be changed later).
- Create a barrel (a cylinder) that will explode when moved in front of the wall into a trigger collider (implemented in the next steps).
    - Add a Rigidbody component to the barrel and freeze rotation along the X and Z axes to prevent it from tipping over.
- Place a trigger collider in front of the broken wall section to detect the presence of the barrel and trigger the explosion.
    - Implement a new C# script "BarrelTriggerController" and attach it to the trigger GameObject.
        - Tag the barrel GameObject as "ExplosiveBarrel" for identification within the script.
        - Organize all broken wall pieces under the trigger GameObject as children for easier access during explosion.

```c#
using UnityEngine;

public class BarrelTriggerController : MonoBehaviour
{
    public float explosionForce = 180f;

    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("ExplosiveBarrel"))
        {
            ExplodeBarrel(other.gameObject);
        }
    }

    private void ExplodeBarrel(GameObject barrel)
    {
        Debug.Log("boom");

        // Get all Rigidbody components in children (pieces of the wall)
        Rigidbody[] rigidbodies = GetComponentsInChildren<Rigidbody>();

        foreach (Rigidbody rb in rigidbodies)
        {
            // Enable physics simulation
            rb.isKinematic = false;

            // Calculate the direction away from the explosion point
            Vector3 explosionDirection = rb.transform.position - barrel.transform.position;

            // Apply force to each Rigidbody piece
            rb.AddForce(explosionDirection.normalized * explosionForce / (explosionDirection.magnitude*explosionDirection.magnitude), ForceMode.Impulse);
        }

        Destroy(barrel);
    }
}
```

![](https://i.imgur.com/4MiHKBw.png)

## Sand and ice (physics materials)
- In this section, we'll set up surfaces with different physics properties: ice, which will be slippery, and sand, which will slow down the player.
- Use ProBuilder to create two planes for the ice and sand surfaces, positioning them in small valleys.
    - Set both planes' layers to "Ground."
    - Adjust their colors to blueish and brownish using Vertex Colors in ProBuilder.

![](https://i.imgur.com/4rlZN7f.png)

- Create a new folder named "Physics" and create three Physics Materials within it: IcePhyMaterial, SandPhyMaterial, and PlayerPhyMaterial.
    - Assign each material to its corresponding collider (for the two surfaces and the player's collider).
    - Note that adjusting friction values in these materials won't affect the player when walking on ice or sand. This is because the player's movement is currently handled using `transform.Translate(...);` This method updates the player's position in each frame without considering the friction values set in the materials.
- Update the PlayerController script by replacing that line (`transform.Translate(...)`) with a function that updates the player's position using physics, allowing friction to affect the player: `rb.AddForce(moveDirection * moveSpeed);`
    - To access the player's Rigidbody component, add the following to the script as well:

```c#
    private Rigidbody rb;

    private void Start()
    {
        rb = GetComponent<Rigidbody>();
    }
```

- Test various friction values for the ice and sand materials until you're satisfied with the player's movement on those surfaces.
    - Slope handling for player movement will be addressed in the next section.

## Player controller fix (physics-based movement)
- Currently, when we move the player, we apply a force in the direction of movement. However, if we encounter a slope, the force is directed straight into the slope rather than slightly upward, which is why the player cannot currently climb steeper slopes. In real life, when walking up stairs, we push ourselves upward as well as forward.
- To address this, we need to detect if there is a slope in front of the player and, if so, apply a small force upwards. This can be achieved with a raycast in front of the player's feet.
- Begin by visualizing where that raycast will be by adding this method to the PlayerController:

```c#
    // Draw the raycast for visualization
    private void OnDrawGizmos()
    {
        Debug.DrawRay(transform.position - transform.up * 0.95f, transform.forward * 1f, Color.green);
    }
```

![](https://i.imgur.com/ApgGa4E.png)

- That looks good (but the raycast could be shorter), we can proceed to add a raycast with these values to the FixedUpdate method and apply an upward force to the player.
    - You will have to declare `public LayerMask groundLayerMask;`

```c#
    void FixedUpdate()
    {
        // Update player position
        rb.AddForce(moveDirection * moveSpeed);

        // Raycast forward to detect ground in front of the player's feet
        if (Physics.Raycast(transform.position - transform.up * 0.95f, transform.forward, 1f, groundLayerMask))
        {
            // Add upward force to help the player ascend
            rb.AddForce(Vector3.up * moveSpeed*0.4f);
        }

        // Rotate the player to face the crosshair
        Vector3 lookDirection = crosshair.position - transform.position;
        lookDirection.y = 0f; // Ensure the player doesn't tilt up or down
        if (lookDirection != Vector3.zero)
        {
            Quaternion targetRotation = Quaternion.LookRotation(lookDirection);
            transform.rotation = Quaternion.Lerp(transform.rotation, targetRotation, Time.fixedDeltaTime * rotateSpeed);
        }
    }
```

- If ascending stairs proves difficult, disable the stairs' collider and add a plane (slope) while disabling its renderer. Climbing the slope should be easier for the player.
    - Additionally, create a plane collider for the side of the stairs to prevent the player from going beneath them.

## Jumping
- Your task here is to implement jumping.
- Use `Input.GetKeyDown(KeyCode.Space)` to detect if the player is pressing space.
- Check with a raycast if the player is on ground (first test the raycast using `Debug.DrawRay`)
- If player pressed space, and the player character is on ground, jump (using `rb.AddForce`, with `ForceMode.Impulse`).

<div align="center"><b>
  <a href="5-PlayerInteraction.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
</b></div>
