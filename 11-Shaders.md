# 11. Shaders: shader graph
- This section will guide you through shaders, specifically shader graph in Unity.
- [Shader Graph documentation](https://docs.unity3d.com/Packages/com.unity.shadergraph%406.9/manual/index.html)
- Examples:
    - [Water shaders by Khena B](https://twitter.com/Khena_B/status/1774016235702792509)
    - [Fractal rendering: Mandelbrot](https://www.youtube.com/watch?v=lyj6jA75WEY)
    - Compute shaders: [Boids](https://www.youtube.com/watch?v=n0PcN0K8EVI), [Game of life](https://www.youtube.com/watch?v=xP5-iIeKXE8)

## Setup
- To begin, navigate to Package Manager -> Window > Package Manager: search for "Universal RP" (set Packages: Unity Registry) and install it.
- Create a new folder named "Shaders".
- Right click within the folder, then select Create > Shader Graph > BuiltIn > Lit Shader Graph.
- Create a new material within the same folder.
    - In the Inspector for the material, open the drop-down menu for the "Shader" field and select the shader graph you just created.
- Place a cube in the scene and apply the created material to it.
- Double click on the shader graph you created to open it.
    - Modify the Base Color (in the Fragment node), then click the Save Asset button (top left) and verify if the cube's color in the scene has changed.

## Texture
- Add a Texture2D into the shader graph (click the "+" button at the top left), and drag-and-drop it into the graph.
- Save the shader graph ("Save Asset" button), and in the Inspector, under Surface Inputs, set Texture2D to the desired texture.
- Click on an empty space within the graph and press space, then search for "Sample Texture 2D" node and add it to the graph.
- Connect these nodes as shown in the image below.

![](https://i.imgur.com/KN5kHYF.png)

- If you start the game, the cube should display the texture, although the texture may not be previewed in the shader graph. To preview it, click on the Texture2D node, and in the Graph Inspector (top right), set the default texture.

## Texture and color
- Adjust the color of the texture by multiplying it with another color. Utilize the "Color" and "Multiply" nodes for this purpose.
- For a pulsating effect, apply the sine of time to modify the intensity of the color's influence on the texture:

![](https://i.imgur.com/ZUCB5Xl.png)

## Water shader
- Begin by creating a plane for the water surface.
    - Turn off its collider and "Cast Shadows" property.
- Next, make a new material and shader graph.
    - Set the Shader property of the material to the newly created water shader graph.
- Open the Water Shader Graph.
    - Set Surface Type to Transparent in the Graph inspector.

### Depth
- This step involves rendering deeper water areas with a darker color.
- We'll achieve this by utilizing the depth buffer.
    - *Note: depth buffer in Unity manages the depth information of pixels in a scene, ensuring correct rendering order and visibility of objects based on their distance from the camera.*
- Example of depth buffer:
![](https://i.imgur.com/prUWVNx.png)

#### Depth: raw value
- **Scene Depth** node returns the depth of the current fragment (pixel) in the scene from the camera's perspective. It essentially provides information about how far away an object is from the camera.
- **Camera** node represents the camera in the scene. The **Far Plane** input of this node specifies the maximum distance from the camera at which objects will be rendered. It defines the far clipping plane of the camera frustum.
    - **Multiply** node multiplies the scene depth with the far plane value. We are scaling the depth values to normalize them to a more manageable range.
- **Screen Position** node provides the screen-space position of the current fragment (pixel) being rendered.
    - **Split** node extracts the A channel of the screen position (which represents the depth of the fragment in screen space).
- **Subtract** node calculates the difference between the scene depth and the screen depth to determine the depth gradient of the water.
    - Higher values indicate deeper water and appear brighter.
![](https://i.imgur.com/ttsGEbN.png)
![](https://i.imgur.com/6zrE199.png)

#### Depth: color
- **Lerp** node mixes two values, such as colors in this instance.
    - Create two Color variables in Shader Graph: ShallowWaterColor and DeepWaterColor.
    - Assign both values in the water material inspector (darker blue for deeper water and brighter blue for shallower water, set their alphas as well in the color picker).
![](https://i.imgur.com/BVU933q.png)
![](https://i.imgur.com/usgYPXS.png)

#### Depth: alpha
- We want shallow water to be more transparent than the deeper water.
    - You have to set a lower alpha value for the ShallowWaterColor.
- **Split** node extracts the alpha channel that is then connected to the Alpha field.
![](https://i.imgur.com/KA2QyzO.png)
![](https://i.imgur.com/D78OrSx.png)

#### Depth: parameters
- To control the look of water, we can add two variables (Depth and DepthStrength).
    - Calibrate those parametres from the inspector until satisfied with how water looks.
- - **Clamp** node ensures that the output stays within the range of 0 to 1.
![](https://i.imgur.com/ObhTXts.png)
![](https://i.imgur.com/Gf0iBfR.png)

### Foam
- Generate foam by adding noise to the water color.
- **Voronoi** node generates a type of pseudo-random noise useful for procedural generation.
- **Time** node is used as input for the Voronoi node, so it is animated.
- **Power** node returns the result of input A to the power of input B.
- **Add** node sums the generated noise with the output from the Lerp node from the previous shader graph.
- **Clamp** node limits output to the range between 0 and 1.
    - Its output is connected to the Base Color field.
![](https://i.imgur.com/uwxIyOF.png)
![](https://i.imgur.com/4wNyw4t.png)

### Ripples
- Currently the water looks very flat.
    - Let's add ripples using a normal map to control how light bounces off the water.
- **Gradient Noise** node makes a smooth noise often used for procedural generation.
    - We'll use two of these noises that move differently to create ripples.
- **Normal From Height** node takes height values and turns them into directions for how light should reflect off a surface.
    - Then, we'll connect its output into the Normal field.
![](https://i.imgur.com/00MSHcP.png)
![](https://i.imgur.com/88OhKwH.png)

### Waves
- In this section, we generate waves by displacing the vertices of the plane on which the water is rendered.
- **Gradient Noise** node produces smooth noise to augment the current positions of fragments, thereby simulating wave patterns.
- DisplacementStrength is a variable you can add to control height of the waves.
![](https://i.imgur.com/dukrv07.png)
- A fix so the positions of fragments are modified only in the Y direction.
![](https://i.imgur.com/8Tjt2Ot.png)

The entire shader graph (without the fix):
![](https://i.imgur.com/HXFoxeM.png)

## Etc
- Better bullet hole decals with the [Decal Renderer Feature](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/renderer-feature-decal.html), [Decal Shader Graph](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/decal-shader.html), [video explanation](https://www.youtube.com/watch?v=f7iO9ernEmM).
- Render player silhouette when behind other objects -> [youtube tutorial](https://www.youtube.com/watch?v=GAh225QNpm0).

<div align="center"><b>
  <a href="10-Procedural-generation.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
  <a href="12-Navigation.html" style="font-size:64px; text-decoration:none"> > </a>
</b></div>
