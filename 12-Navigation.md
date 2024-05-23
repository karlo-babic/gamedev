# 12. Navigation: NavMesh
- This section covers enemy [navigation](https://docs.unity3d.com/Packages/com.unity.ai.navigation@2.0/manual/NavigationOverview.html): NavMesh, NavMesh Agent, NavMesh Obstacle.

## Navigation mesh
- To define surfaces on which an agent can move, create a plane covering the entire floor in the base, and enable the Navigation Static property in the Navigation window > Object tab (with that GameObject selected).
- To define objects that agents can't go through, select all of them and in the Navigation window > Object tab enable the Navigation Static property and set Navigation Area to "Not Walkable" (so the surfaces on top of those objects are not defined as walkable areas).
- In the Navigation window > Bake tab, click on the Bake button. The navigation mesh should now be ready.

![](https://i.imgur.com/sqFQtPJ.png)

## NavMesh Agent
- Add the "Nav Mesh Agent" component to the GameObject that will represent the enemy. Remove the RigidBody component from the enemy GameObject if there is one.
- Create a new C# script "EnemyNavigation" (below), and set its public variable "Move Position Transform" to the player GameObject, so the enemy follows the player.
- In the enemy GameObject's Nav Mesh Agent component, set the Base Offset so the enemy is on the floor.

```c#
using UnityEngine;
using UnityEngine.AI;

public class EnemyNavigation : MonoBehaviour
{
    public Transform movePositionTransform;

    private NavMeshAgent navMeshAgent;

    void Awake()
    {
        navMeshAgent = GetComponent<NavMeshAgent>();
    }

    void Update()
    {
        navMeshAgent.destination = movePositionTransform.position;
    }
}
```

## NavMesh Obstacle
- To allow the agent to go through doors when they are open, use the NavMesh Obstacle component.
- Add the NavMesh Obstacle component to doors that can open.
    - By default, they are treated as a rigidbody, so the enemy will collide with the doors.
    - In the obstacle component, enable the "Carve" property (so the NavMesh updates when the doors change position, so the enemy doesn't try to collide with the closed doors).
- In the Navigation window > Object tab (with the door selected) disable the Navigation Static property (if it was enabled).
- In the Navigation window > Bake tab, Clear and then Bake the NavMesh again.
- You can add the obstacle component to barrels and broken wall pieces as well.
- You can disable the renderer and collider components in the plane GameObject you created for NavMesh (note: you can't bake the NavMesh while the plane's renderer is disabled).

<div align="center"><b>
  <a href="11-Shaders.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
  <a href="13-User-interface.html" style="font-size:64px; text-decoration:none"> < </a>
</b></div>
