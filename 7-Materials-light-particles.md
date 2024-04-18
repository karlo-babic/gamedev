# 7. Materials, light, and particles
- In this section, we'll delve into materials, lighting, environment setup, and particle effects.
- Examples:
    - [Going From Prototype to Finished Product: Iron Rebellion](https://www.reddit.com/r/virtualreality/comments/1c4qz5t/going_from_prototype_to_finished_product_iron/)
    - [Light game design](https://www.youtube.com/watch?v=Y01C2n_HWVM)
    - [DOS raytracer](https://youtu.be/N8elxpSu9pw&t=672)
    - [Particles tutorial](https://www.youtube.com/watch?v=FEA1wTMJAR0)

## Asset importing
- Let's start by importing a 3D model of a barrel, applying appropriate textures to its material, and placing it in the scene.
    - Download a 3D model of a barrel from a website like [TurboSquid](https://www.turbosquid.com/).
    - Create a new folder named "Models" in your "Assets" folder, and within it, create another folder named "Exploding Barrel."
    - Import the downloaded 3D model into the "Exploding Barrel" folder.
    - Once imported, select the model, navigate to the "Materials" tab in the inspector, and click on "Extract Textures" and "Extract Materials." Ensure the destination folder for both is set to "Exploding Barrel."
    - Now, you'll find new textures in the "Exploding Barrel" folder. Apply these textures to the barrel's material, adjusting properties like Smoothness, Normal Map, Height Map, and Tiling as needed.
    - Replace the previous barrel in the scene with the newly imported one, ensuring it has necessary components like Rigidbody, Collider, and the correct tag.

## Materials
- Next, let's add two additional materials: one for the base's interior floors and another for the ice surface outside. Additionally, we'll create a material that uses a reflection probe.
    - Download suitable materials for the base floor and the ice surface from websites like [Poly Haven](https://polyhaven.com/textures) or [TextureCan](https://www.texturecan.com/).
    - Import these materials into your project's "Textures" folder, organizing them into separate "Ice" and "Floor" folders.
    - Create new materials in Unity and assign the appropriate textures to each.
    - Apply these materials to the respective floor and ice surface objects in your scene, adjusting properties like Smoothness, Normal Map, Height Map, and Tiling as desired.

### Reflection probe
- Let's set up a [reflection probe](https://docs.unity3d.com/Manual/class-ReflectionProbe.html):
    - Create a sphere GameObject and assign it a material with high Metallic and Smoothness values.
    - Add a Reflection Probe GameObject to the scene by going to GameObject > Light > Reflection Probe.
    - Make the Reflection Probe a child of the sphere and position it at the center (0,0,0) of the sphere.
    - Configure the Reflection Probe settings, setting Type to "Realtime" and Refresh Mode to "Every Frame."
    - Add a Rigidbody component to the sphere GameObject to ensure physics interaction.
    - To improve reflections further, change the skybox:
        - Download a suitable skybox image from websites like [Sketchfab](https://sketchfab.com/) (login required) and import it into a "Skybox" folder in your project.
        - Create a new material named "Space Skybox" and set its Shader to "Skybox/Panoramic," assigning the imported skybox image as the Spherical HDR.
        - Set this "Space Skybox" material as the skybox:
            - Open Window > Rendering > Lighting and in the tab "Environment" set the Skybox Material to the one you just created.
    - You will maybe have to run the game for the reflection to update to the new skybox.

## Lights
- Before we add lights, we have to make it so it is darker inside the base.
    - Activate "CeilingForShadow0" if it is not active currently (it should be located in the Floor 0 in the scene hierarchy).
    - Open Window > Rendering > Lighting and in the tab "Environment":
        - Set Environment Lighting: Intensity Multiplier to 0.
        - Set Environment Reflections: Intensity Multiplier to a lower value.
        - Add fog.
- Create a new GameObject > Light > Spotlight.
    - Set it on the ceilling looking down and test various values (such as: Range, Spot Angle, Color, Intensity, ...).
    - Set multiple such light sources around the base.
    - Attach one to the player (player's torchlight).
- We can make one of those lights flicker:
    - Attach the following script to a light source.

- Adjust the scene's lighting to create a darker ambiance within the base:
    - Activate the "CeilingForShadow0" GameObject if it's not already active (located within the Floor 0 hierarchy).
    - In the Unity editor, navigate to Window > Rendering > Lighting > Environment.
    - Set Environment Lighting: Intensity Multiplier to 0 and reduce Environment Reflections: Intensity Multiplier.
    - Add fog to enhance the atmosphere outside the base.
- Create spotlights (GameObject > Light > Spotlight) on the ceiling to illuminate the interior, experimenting with settings like Range, Spot Angle, Color, and Intensity.
- Scatter multiple spotlights around the base.
- Attach one spotlight to the player GameObject to simulate a torchlight effect.
- Implement a flickering effect for one of the spotlights using a script:

```c#
using System.Collections;
using UnityEngine;

public class SpotlightFlicker : MonoBehaviour
{
    public float flickerInterval = 0.1f; // Time between flickers
    public float flickerChance = 0.5f; // Chance of flicker occurring

    private Light spotlight; // Reference to the spotlight component

    void Start()
    {
        // Get the Light component attached to this GameObject
        spotlight = GetComponent<Light>();

        // Start the flickering coroutine
        StartCoroutine(Flicker());
    }

    IEnumerator Flicker()
    {
        while (true)
        {
            // Randomly decide whether to flicker
            if (Random.value < flickerChance)
            {
                // Toggle the spotlight on or off
                spotlight.enabled = !spotlight.enabled;
            }

            // Wait for the flicker interval
            yield return new WaitForSeconds(flickerInterval);
        }
    }
}
```

## Particles
- Here we will add particles to the malfunctioning/flickering light.
- Create a new GameObject > Effects > Particle System.
- Place it as a child of the flickering light and position it at its location.
- Modify various values so the particles fall down randomly, like sparks.

- In this section, we'll enhance the malfunctioning/flickering light effect with particles.
    - Start by creating a new GameObject and selecting Effects > Particle System.
    - Make this particle system a child of the flickering light and position it accordingly.
    - Adjust the particle system settings to simulate falling particles randomly, resembling sparks. Experiment with parameters like emission rate, lifetime, speed, size, and color to achieve the desired effect.

![](https://i.imgur.com/0WmgJLQ.png)

<div align="center"><b>
  <a href="6-Physics.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
</b></div>
