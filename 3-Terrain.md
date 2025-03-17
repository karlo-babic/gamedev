# 3. Terrain creation: Terrain Tools plugin
- [Unity Packages: Terrain Tools](https://docs.unity3d.com/Packages/com.unity.terrain-tools@4.0/manual/getting-started-with-terrain-tools.html)
    - [Tutorial](https://www.youtube.com/watch?v=smnLYvF40s4)
- This section delves into terrain creation using the Terrain Tools plugin.
- Examples of terrain building:
    - Traditional game maps (Sykrim, GTA, Guild Wars): [comparison](https://www.youtube.com/watch?v=LwXV0oLEfCM)
    - Unconventional game world: [Outer Wilds](https://www.youtube.com/watch?v=d6LGnVCL1_A)
    - Terrain generation:
        - [Minecraft's terrain generation](https://www.youtube.com/watch?v=CSa5O6knuwI)
        - [Landmass Generation tutorial series](https://www.youtube.com/watch?v=wbpMiKiSKm8&list=PLFt_AvWsXl0eBW2EiBtl_sxmDtSgZBxB3)
    - Experimental:
        - [Hyperbolica](https://www.youtube.com/watch?v=VYfWfrk5P7w)
        - ["Tiny voxels"](https://www.youtube.com/watch?v=CnBIq9KRpcI)
        - ["4D Minecraft"](https://www.youtube.com/watch?v=u8LMyWcKL_c)

## Terrain Tools plugin
- To install the plugin, navigate to the top menu bar and select Window > Package Manager.
    - In the Package Manager window, select Packages: Unity Registry, and search "terrain".
    - Install Terrain Tools.

## Terrain
- To create new terrain:
    - Navigate to the top menu bar: Window > Terrain > Terrain Toolbox.
    - In the "Create New Terrain" tab, click on the "Create" button.
- Adjust the positioning of the Player and Main Camera GameObjects so they are above the terrain. Then, create a cube to serve as a floor for the Player.
- While the Terrain GameObject is selected in the Inspector, you'll find the "Terrain" component with five tabs: "Create Neighbor Terrains", "Paint Terrain", "Paint Trees", "Paint Details", and "Terrain Settings".
- In the "Terrain Settings" tab:
    - Set Terrain Width and Length to 200 in the "Mesh Resolution" section.
- If additional terrain sections are required:
    - Access the "Create Neighbor Terrains" tab.
    - Click on neighboring regions around the current terrain in the scene.
- Ensure each terrain section is set to the "Ground" layer to enable raycast for crosshair positioning to interact with the terrain.
- In the "Paint Terrain" tab:
    - Choose "Raise or Lower Terrain" from the drop-down menu.
    - Select a brush and adjust Brush Strength and Size as needed.
    - Sculpt the terrain to create hills or other features.
- To add high-frequency details, we can use the Sculpt/Noise option (instead of the "Raise or Lower Terrain" option in the "Paint Terrain" tab).
    - In the inspector, set up the Noise Height Tool Settings. Try:
        - Noise Type: Billow.
        - Zoom in and out the Noise Field Preview (to change the scale of the noise).
        - Test different values in Domain Settings.

## Terrain Heightmap
- Utilizing a texture as a heightmap allows us to determine the elevation of terrain points based on the values within the texture:
    - Begin by acquiring a suitable texture:
        - We can generate a noise texture via: Window > Terrain > Edit Noise.
            - Export the generated noise as a texture using the option located at the bottom of the window.
- Next, proceed to create a new terrain:
    - Access the Terrain Toolbox from the top menu bar: Window > Terrain > Terrain Toolbox.
        - Activate the "Import Heightmap" feature.
            - Select the desired texture to be used as the heightmap.

## Terrain Texturing
- Aim to create a terrain that resembles a moon or barren planet, with sandy ground and rocky areas.
- Get two textures: one for the sandy ground and the other for rocky regions from [Poly Haven](https://polyhaven.com/textures).
- Import the textures into the Unity project under Assets/Textures.
- In the Inspector for the terrain, go to the Paint Terrain tab and choose "Paint Texture" from the drop-down menu.
    - Under the Terrain component, create a new layer called "Sand" and assign the sandy texture.
        - Open the created layer in the Inspector (double click on "Sand" in the Layer Palette list, or click on the created file for that layer in Assets folder):
            - Set size for X and Y to a bigger number (try 20).
            - Set the normal map (to the texture with "nor" in its filename).
    - Repeat the process for rocky areas, creating a new layer and assigning the rocky texture.
    - If the Layer Palette list is buggy, remove both layers and add the created layers (from Assets folder) as needed.
    - Select the rocky layer and paint the terrain for rocky areas.
- You can use Brush Mask to automatically paint flatter terrain with sand texture:
    - Create a new Brush Mask (in the Terrain component > Brush Mask section > "+" Slope).
    - Select the Sand layer and paint over the rocky areas (use large Brush Size).

The final state should look something like this:
![](https://i.imgur.com/EMv9TkJ.png)

---

<div align="center"><b>
  <a href="2-Camera.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
  <a href="4-ProBuilder.html" style="font-size:64px; text-decoration:none"> > </a>
</b></div>
