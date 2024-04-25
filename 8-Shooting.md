# 8. Shooting
- In this section, we will implement shooting with [decals](https://www.techarthub.com/how-to-make-a-bullet-hole-in-substance-designer/) (for bullet holes) and the effect of shooting on the barrel and on characters (such as an enemy or the player).
- Examples:
    - [The Slightly Odd Way Bullets Work In FPS Games](https://www.youtube.com/watch?v=EpkvNUoxlxM)
    - [How Bullets Work In Games](https://www.youtube.com/watch?v=OaCqpHmHTXE)

## Raycast
- Add a mouse click listener in the Update method in the PlayerController script, declare `private Shooting shooting;` and initialize it in the Start method `shooting = GetComponent<Shooting>();`.

```c#
        // Check for left mouse button click
        if (Input.GetButtonDown("Fire1"))
        {
            // Call the Shoot() function from the Shooting script
            shooting.Shoot();
        }
```

- Create a new script "Shooting".
    - The Shoot method in this script casts a ray from the player in the looking direction (bullet path).

```c#
using UnityEngine;
using System.Collections.Generic;

public class Shooting : MonoBehaviour
{
    public Transform crosshair;
    public float shootRange = 100f;

    public void Shoot()
    {
        // Calculate the direction from the player to the crosshair
        Vector3 shootDirection = crosshair.position - transform.position;

        // Cast a ray in the shoot direction
        RaycastHit hit;
        if (Physics.Raycast(transform.position, shootDirection, out hit, shootRange))
        {
            // Handle hit event
            Debug.Log("Hit " + hit.collider.gameObject.name);
        }
    }

    // Draw the raycast for visualization
    private void OnDrawGizmos()
    {
        Debug.DrawRay(transform.position, (crosshair.position - transform.position) * shootRange, Color.red);
    }
}
```

## Decals (bullet holes in walls)
- To add bullet holes where the raycast hits, we can use decals. Decals are graphical elements used to simulate the appearance of bullet holes, scratches, or other damage on surfaces.
- Find a suitable texture for bullet hole decals, import them, and create a bullet hole material.
    - Change the material's Rendering Mode so that transparent parts do not render.
- Create a plane in the scene and apply the bullet hole material to it.
    - Resize it, disable its collider, and ensure it doesn't cast shadows.
- Move the plane to the Prefabs folder and remove it from the scene (we will instantiate it as bullet hole decals).
- In the Shooting.cs script, instantiate the decal using `InstantiateDecal(hit.point, hit.normal, hit.transform);` when the raycast hits an object.
- The InstantiateDecal method positions the bullet hole at the point where the raycast hits and rotates it to align with the surface:

```c#
    void InstantiateDecal(Vector3 position, Vector3 normal, Transform hitTransform)
    {
        // Instantiate the decal prefab
        GameObject decal = Instantiate(decalPrefab, position, Quaternion.identity);

        // Determine the rotation to align with the surface
        Quaternion rotation = Quaternion.FromToRotation(Vector3.up, normal);

        // Apply the rotation to the decal
        decal.transform.rotation = rotation;

        // Adjust the decal's position slightly to prevent z-fighting
        decal.transform.position += normal * 0.01f;

        // Parent the decal to the hit object to keep it in the correct position
        decal.transform.parent = hitTransform;
    }
```

- Currently, the decals are instantiated, but they are never destroyed.
- We can handle decal destruction more efficiently by attaching a script to each decal, which destroys it after a set lifespan:

```c#
using UnityEngine;

public class DecalDestroyer : MonoBehaviour
{
    public float lifespan = 5f; // Lifespan of the decal in seconds

    private float creationTime; // Time when the decal was instantiated

    void Start()
    {
        // Record the time when the decal was instantiated
        creationTime = Time.time;
    }

    void Update()
    {
        // Check if the decal has exceeded its lifespan
        if (Time.time - creationTime >= lifespan)
        {
            // Destroy the decal
            Destroy(gameObject);
        }
    }
}
```

- However, this approach is inefficient as each decal checks if it has exceeded its lifespan in every frame.
- A better method is to maintain a list of all decals and destroy the oldest ones if their count exceeds a certain threshold. This check only needs to occur when a new decal is instantiated. Modify the Shooting script accordingly:

```c#
using UnityEngine;
using System.Collections.Generic;

public class Shooting : MonoBehaviour
{
    public Transform crosshair;
    public float shootRange = 100f;
    public GameObject decalPrefab; // Prefab for the bullet hole decal
    public int maxDecals = 10; // Maximum number of decals allowed

    private List<GameObject> instantiatedDecals = new List<GameObject>();

    public void Shoot()
    {
        // Calculate the direction from the player to the crosshair
        Vector3 shootDirection = crosshair.position - transform.position;

        // Cast a ray in the shoot direction
        RaycastHit hit;
        if (Physics.Raycast(transform.position, shootDirection, out hit, shootRange))
        {
            // Handle hit event
            Debug.Log("Hit " + hit.collider.gameObject.name);
            InstantiateDecal(hit.point, hit.normal, hit.transform);
        }
    }

    void InstantiateDecal(Vector3 position, Vector3 normal, Transform hitTransform)
    {
        // Instantiate the decal prefab
        GameObject decal = Instantiate(decalPrefab, position, Quaternion.identity);

        // Determine the rotation to align with the surface
        Quaternion rotation = Quaternion.FromToRotation(Vector3.up, normal);

        // Apply the rotation to the decal
        decal.transform.rotation = rotation;

        // Adjust the decal's position slightly to prevent z-fighting
        decal.transform.position += normal * 0.01f;

        // Parent the decal to the hit object to keep it in the correct position
        decal.transform.parent = hitTransform;

        // Add the decal to the list of instantiated decals
        instantiatedDecals.Add(decal);

        // Ensure that only the newest decals are kept
        RemoveOldDecals();
    }

    void RemoveOldDecals()
    {
        // If the number of decals exceeds the maximum limit
        if (instantiatedDecals.Count > maxDecals)
        {
            // Calculate the number of excess decals
            int excessDecals = instantiatedDecals.Count - maxDecals;

            // Remove the oldest excess decals from the list and destroy them
            for (int i = 0; i < excessDecals; i++)
            {
                GameObject decal = instantiatedDecals[0]; // Get the oldest decal
                instantiatedDecals.RemoveAt(0); // Remove it from the list
                Destroy(decal); // Destroy the GameObject
            }
        }
    }

    // Draw the raycast for visualization
    private void OnDrawGizmos()
    {
        Debug.DrawRay(transform.position, (crosshair.position - transform.position) * shootRange, Color.red);
    }
}
```

## Check if shot
- To determine if an object has been shot (such as an enemy or an explosive barrel), attach the following script to that object (attach it to the explosive barrel):

```c#
using UnityEngine;

public class ShotController : MonoBehaviour
{
    public void Shot()
    {
        Debug.Log("I have been shot!");
    }
}
```

- Additionally, call that method from the Shoot method (in Shooting.cs) when the raycast hits an object:

```c#
            // Check if the hit object has a ShotController component
            ShotController shotController = hit.collider.gameObject.GetComponent<ShotController>();
            if (shotController != null)
            {
                // Call the Shot method of the ShotController
                shotController.Shot();
            }
```

## Barrel explosion
- In the ShotController script, we can determine if the object that was shot is an enemy, player, or an exploding barrel, and take the appropriate action:

```c#
using UnityEngine;

public class ShotController : MonoBehaviour
{
    public void Shot()
    {
        if (gameObject.CompareTag("ExplosiveBarrel"))
        {
            Explode();
        }
        else if (gameObject.CompareTag("Character"))
        {
            TakeDamage();
        }
        else
        {
            Debug.LogWarning("Object does not have a recognized tag for shot behavior.");
        }
    }

    void Explode()
    {
        Debug.Log("Barrel exploded!");
    }

    void TakeDamage()
    {
        Debug.Log("Took damage!");
    }
}
```

- Repurpose the trigger collider we used next to the broken wall for barrel detection. We want to check if the barrel is inside that trigger. When the barrel explodes, it should blast the broken wall away.
    - Modify the BarrelTriggerController script (attached to the trigger) to contain only the code below. Also, rename it to "WallExplosionTriggerController".
    - Set the trigger collider's layer to "Ignore Raycast" to prevent shooting the invisible collider (you can do the same for all trigger colliders):

```c#
using UnityEngine;

public class WallExplosionTriggerController : MonoBehaviour
{
    public float explosionForce = 180f;

    public void WallExplosion(GameObject barrel)
    {
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
    }
}
```

- Attach the script below to the barrel. We will use the variable `explosiveAreaTrigger` to check if the barrel is inside the trigger and to access that trigger to call the `WallExplosion` method.
    - Set the trigger collider game object's Tag to "ExplosionArea".

```c#
using UnityEngine;

public class BarrelState : MonoBehaviour
{
    public GameObject explosiveAreaTrigger;

    private void OnTriggerEnter(Collider other)
    {
        if (other.CompareTag("ExplosionArea"))
        {
            explosiveAreaTrigger = other.gameObject; // Store reference to the trigger GameObject
        }
    }

    private void OnTriggerExit(Collider other)
    {
        if (other.CompareTag("ExplosionArea"))
        {
            explosiveAreaTrigger = null; // Clear reference to the trigger GameObject
        }
    }
}
```

- Modify the Explode method in the ShotController script. Now, it calls the `WallExplosion` method in the trigger collider if the barrel is inside that trigger:

```c#
    void Explode()
    {
        Debug.Log("Barrel exploded!");
        // Check if the barrel is in the explosive area and trigger GameObject reference is not null
        BarrelState barrelState = GetComponent<BarrelState>();
        if (barrelState != null && barrelState.explosiveAreaTrigger != null)
        {
            // Call explosion method from the trigger GameObject
            barrelState.explosiveAreaTrigger.GetComponent<WallExplosionTriggerController>().WallExplosion(gameObject);
        }
        Destroy(gameObject);
    }
```

## Enemy, HP
- Create an enemy similar to the player: an empty parent "Enemy" with a Rigidbody component and a Capsule child.
- Set the enemy's collider (Capsule) to have the Tag "Character" (game objects that can take HP damage).
- Attach to the enemy's collider (Capsule) the script "ShotController" and a new script "HealthSystem":

```c#
using UnityEngine;

public class HealthSystem : MonoBehaviour
{
    public float HP = 100f;

    public void TakeDamage(float dmgValue)
    {
        HP = HP - dmgValue;
        if (HP <= 0)
        {
            Death();
        }
    }

    private void Death()
    {
        Debug.Log("Died");
        Destroy(gameObject);
    }
}
```

<div align="center"><b>
  <a href="7-Materials-light-particles.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
</b></div>
