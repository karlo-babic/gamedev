# 9. Character animation
- In this section, we will cover character animation.
- Examples:
    - [Player animation tutorial playlist](https://www.youtube.com/playlist?list=PLwyUzJb_FNeTQwyGujWRLqnfKpV-cj-eO)
    - [Procedural (spider) animation tutorial](https://www.youtube.com/watch?v=swYBGqXtHEY)
    - Procedural animation in [Rain World](https://www.youtube.com/watch?v=sVntwsrjNe4) and [Synthetic Selection](https://www.youtube.com/watch?v=iNXzOuc9UWo)

## Preparation
- Obtain animation assets from [Mixamo](https://www.mixamo.com/): search for animations in the Animations tab and 3D models in the Characters tab.
- Import the model and animations into a new folder named "Models/Player".
    - Extract textures and materials from the character model.
    - Place the model as a child of the Player GameObject in the scene.
    - Disable the Capsule's renderer, leaving it solely for the player's collider.
- Add the Animator component to the player model in the inspector.
- Create an "Animations" folder and an Animator Controller named "PlayerController" within it. Assign this controller to the Animator component.
- Expand animation assets and copy "Animation Clip" out of each one (click on it and press ctrl-d), and rename them appropriately (e.g., Idle, Walking, Running, Fire, etc.), then move them to the Animations folder.

## Animator
- Open the Animator by double-clicking the PlayerController.
    - Drag the Idle animation clip into the Animator, setting it as the Default state (connected to the Entry state).
- Enable the "Loop Time" field for the Idle animation clip to allow looping.
- Repeat these steps for the Walking and Running animation clips.
    - Enable the "Loop Pose" field to keep the player model stationary during these animations.

### Bool parameters: isWalking, isRunning (and fire trigger)
- Include Walking and Fire animations in the Animator.
- Establish a transition from the Idle state to the Walking state.
    - In the Animator window, navigate to the "Parameters" tab and add a new bool parameter: "isWalking".
    - Assign a condition to the transition from Idle to Walking: "isWalking = true" (to switch from Idle to Walking when the player is moving).
    - Access the "isWalking" variable from the PlayerController script:
        - Declare `public Animator animator;`
        - Set this field in the Player's inspector.
        - In the Update method, set `animator.SetBool("isWalking", true);` if `moveDirection.magnitude > 0`, otherwise set it to false.
- Disable the "Has Exit Time" field for each transition.
- Establish a transition from Walking to Idle (activated when the player is not walking).
- Create a transition from "Any State" to the Fire state (and add a new trigger parameter: "fire"). Use this parameter as the condition for the transition.
    - This transition allows the state machine to switch to the Fire state regardless of the current animation state.
    - In the transition settings, set the transition offset to 0.25 to synchronize the firing animation with the trigger activation.
    - In script, set this trigger when the player shoots: `animator.SetTrigger("fire");`.
- Add a transition from Fire to Exit (looping the state machine back to the Entry state).

![](https://i.imgur.com/3yA7dcC.png)

- Add the Running animation into the animator.
- Include a new bool parameter: "isRunning". Adjust it in the PlayerController script by assigning true if `rb.velocity.magnitude > 5`.
- Establish transitions from Walking to Running, from Running to Walking, and from Running to Idle, utilizing appropriate parameters as conditions.

![](https://i.imgur.com/kbuAYu6.png)

### 1D Blend trees and float parameter speed
- Right-click in the animator window > Create State > From New Blend Tree.
    - A new float parameter might appear, rename it to "speed".
        - Set its value from the PlayerController script: `animator.SetFloat("speed", rb.velocity.magnitude);`.
- Remove Walking and Running states from the animator.
    - Establish a new transition from Idle state to the Blend Tree and one transition back to the Idle state.
    - The transition from Idle to Blend Tree should have one condition: speed > 0.1
    - The transition from Blend Tree Idle should have one condition: speed < 0.1

![](https://i.imgur.com/dfQiE7A.png)

- Double click on the Blend Tree to open it.
    - Click on the Blend Tree node.
    - In the inspector, add two motions: Walking and Running.
    - Set the threshold on the Running motion to be 6.5 (you first have to disable "Automate Threshold").
        - This value represents approximately the maximum speed the player moves at, although it may vary depending on your implementation.
- Now, the player character will be animated accordingly based on the speed at which they move. However, this setup currently only accounts for forward motion.

### 2D Blend Tree and float parameters velocityX and velocityY
- Start by creating a new Blend Tree and setting it as the default state (right-click on it).
    - Ensure that the Entry has a transition leading to the new Blend Tree. Remove the Idle state and any previous Blend Trees.
    - Double-click the new Blend Tree to open it.
- Click on the Blend Tree node and set the Blend Type to "2D Freeform Directional" in the inspector.
- Create two float parameters: velocityX and velocityY.
- Add all desired motions to animate (such as Idle, Walking, Running, Strafe left, Strafe right, etc.).

![](https://i.imgur.com/YP1ByOb.png)

- You can move the points around.
    - Blue points indicate favored velocities for animations, while the red point represents the current velocity in both x and y directions.
- Set the x and y velocity in the script. The velocity should be relative to the look direction.
    - You will have to have acess to the `lookDirection` variable. You can declare it outside the FixedUpdate method.

```c#
Quaternion targetRotation = Quaternion.LookRotation(lookDirection);
Vector3 relativeVelocity = Quaternion.Inverse(targetRotation) * rb.velocity/6.5f;
animator.SetFloat("velocityX", relativeVelocity.x);
animator.SetFloat("velocityY", relativeVelocity.z);
```

- Now player movement should be animated properly.
    - But shooting animation stops all other animations (solvable using animation layers), and we are missing jump and death animations.

<div align="center"><b>
  <a href="8-Shooting.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
</b></div>
