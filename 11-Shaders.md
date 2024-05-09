# 11. Shaders: shader graph
- This section will guide you through shaders, specifically shader graph in Unity.
- [Shader Graph documentation](https://docs.unity3d.com/Packages/com.unity.shadergraph%406.9/manual/index.html)
- Examples:
    - [Water shaders by Khena B](https://twitter.com/Khena_B/status/1774016235702792509)
    - [Fractal rendering: Mandelbrot](https://www.youtube.com/watch?v=lyj6jA75WEY)
    - Compute shaders: [Boids](https://www.youtube.com/watch?v=n0PcN0K8EVI) [Game of life](https://www.youtube.com/watch?v=xP5-iIeKXE8)

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

To be continued...

<div align="center"><b>
  <a href="10-Procedural-generation.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
</b></div>
