# 10. Procedural generation
- In this section, we will cover a straightforward form of procedural generation: randomly placing items on both flat and uneven terrain.
- Examples of proceural generation:
    - [Procedural Content Generation Wiki](http://pcg.wikidot.com/)
    - [reddit/proceduralgeneration](https://www.reddit.com/r/proceduralgeneration/)
    - Games: Minecraft, No Man's Sky, Deep Rock Galactic, Valheim, Dwarf Fortress, Don't Starve
    - [Procedural Landmass Generation tutorial playlist](https://www.youtube.com/watch?v=wbpMiKiSKm8&list=PLFt_AvWsXl0eBW2EiBtl_sxmDtSgZBxB3)
    - [Graph based dungeon generation](https://www.youtube.com/watch?v=NpS5v_Tg4Bw)
    - [Cellular Automata](https://www.youtube.com/watch?v=slTEz6555Ts)
    - [Wave Function Collapse algorithm](https://www.youtube.com/watch?v=2SuvO4Gi7uY)
    - [Creature generation](https://www.youtube.com/watch?v=a87tB__3KEs)
    - [Procedural material tutorial playlist (Blender)](https://www.youtube.com/watch?v=PpFNJI631RE&list=PLsGl9GczcgBs6TtApKKK-L_0Nm6fovNPk)

## Spawning Items
- Obtain several smaller 3D assets you wish to spawn (e.g., rocks).
- Import them into your Unity project.
- Place them in the scene and unpack them (right-click > Prefab > Unpack).
    - Extract the 3D models from these assets if they contain multiple GameObjects.
    - Resize them if necessary.
- Create prefabs for these 3D models (within a new folder named "Resources/Rocks").
    - *Note: All assets that are in a folder named "Resources" anywhere in the Assets folder can be accessed via the Resources.Load functions* ([docs](https://docs.unity3d.com/ScriptReference/Resources.html)).

### Inside (on flat floor)
- To randomly spawn items inside a room with a flat floor, we will generate N random positions within that floor area.
- Define boundaries for item spawning within the room using a cube.
    - Create a cube and adjust its position/rescale to cover the desired spawn area.
        - Disable its renderer and collider.
    - Attach the below script to the cube.
    - Set the Folder Name property in the cube's inspector to "Rocks" (the name of the folder containing item prefabs).

```c#
using UnityEngine;
using System.Collections.Generic;

public class SpawnItemsRoom : MonoBehaviour
{
    // Name of the folder containing item prefabs
    public string folderName;
    public int numberOfItemsToSpawn;

    private List<GameObject> itemPrefabs = new List<GameObject>();

    private void Start()
    {
        LoadItemPrefabs();
        SpawnItems();
    }

    private void LoadItemPrefabs()
    {
        GameObject[] prefabs = Resources.LoadAll<GameObject>(folderName);
        itemPrefabs.AddRange(prefabs);
    }

    private void SpawnItems()
    {
        Bounds spawnBounds = GetComponent<Renderer>().bounds;

        for (int i = 0; i < numberOfItemsToSpawn; i++)
        {
            // Generate a random position within the room area
            Vector3 randomPosition = new Vector3(
                Random.Range(spawnBounds.min.x, spawnBounds.max.x),
                spawnBounds.min.y + 0.3f,
                Random.Range(spawnBounds.min.z, spawnBounds.max.z)
            );

            // Instantiate a random rock prefab
            int randomIndex = Random.Range(0, itemPrefabs.Count);
            GameObject selectedPrefab = itemPrefabs[randomIndex];
            Quaternion randomRotation = Quaternion.Euler(0f, Random.Range(0f, 360f), 0f);
            GameObject newRock = Instantiate(selectedPrefab, randomPosition, randomRotation);
            newRock.transform.parent = transform; // Set the current GameObject as the rock's parent
        }
    }
}
```

### Outside (on uneven terrain)
- Utilize a cube for boundaries (as before), positioning it outside over the terrain.
    - Disable its renderer and collider.
- This time, we will employ raycasts with random origins (on the top face of the cube), projecting downwards onto the terrain.
    - Attach the provided script to the cube.

```c#
using UnityEngine;
using System.Collections.Generic;

public class SpawnItemsTerrain : MonoBehaviour
{
    // Name of the folder containing item prefabs
    public string folderName;
    public int numberOfItemsToSpawn;

    private float raycastMaxDistance;
    private List<GameObject> itemPrefabs = new List<GameObject>();

    private void Start()
    {
        // Calculate the maximum distance for raycasting (cube height)
        raycastMaxDistance = GetComponent<Renderer>().bounds.max.y - GetComponent<Renderer>().bounds.min.y;
        LoadItemPrefabs();
        SpawnItems();
    }

    private void LoadItemPrefabs()
    {
        GameObject[] prefabs = Resources.LoadAll<GameObject>(folderName);
        itemPrefabs.AddRange(prefabs);
    }

    private void SpawnItems()
    {
        for (int i = 0; i < numberOfItemsToSpawn; i++)
        {
            // Get random position within the spawn area cube
            Vector3 randomPosition = GetRandomPosition();

            // Cast a ray downwards from the random position
            RaycastHit hit;
            if (Physics.Raycast(randomPosition, Vector3.down, out hit, raycastMaxDistance))
            {
                // Instantiate a random rock prefab at the hit point
                int randomIndex = Random.Range(0, itemPrefabs.Count);
                GameObject selectedPrefab = itemPrefabs[randomIndex];
                Quaternion randomRotation = Quaternion.Euler(0f, Random.Range(0f, 360f), 0f);
                GameObject newRock = Instantiate(selectedPrefab, hit.point, randomRotation);
                newRock.transform.parent = transform; // Set the current GameObject as the rock's parent
            }
        }
    }

    private Vector3 GetRandomPosition()
    {
        Bounds spawnBounds = GetComponent<Renderer>().bounds;

        float randomX = Random.Range(spawnBounds.min.x, spawnBounds.max.x);
        float randomZ = Random.Range(spawnBounds.min.z, spawnBounds.max.z);

        return new Vector3(randomX, spawnBounds.max.y + 2f, randomZ);
    }
}
```

<div align="center"><b>
  <a href="9-Character-animation.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
  <a href="11-Shaders.html" style="font-size:64px; text-decoration:none"> > </a>
</b></div>
