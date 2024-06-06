# 13. User Interface
- In this section, we will create the main menu UI.

## Main menu
- Create a new scene "MainMenu" in the Scenes folder.
    - Open it by double-clicking.
    - Remove the Directional Light from the scene.
    - Set the scene view to 2D.
    - Create a UI > Canvas.
        - Double-click on it to zoom to it.
- Go to File > Build Settings and add the main menu scene to the "Scenes In Build" list. Set the main menu scene to index 0 (so it loads first when the game runs).
- Create UI > Panel as a child of the Canvas, rename it to "Background".
    - Set its Image component "Source Image" property to a suitable background image.
    - Scale and position it using the Rect Transform component to fit the screen.
    - Set the main camera Clear Flags property to "Solid Color" and set the Background color.

### Main menu: Buttons
- Create a new Panel "Main Menu" as a child of the Canvas. Position it at the center of the screen (using the Rect Transform component).
    - Create UI > Button as a child of that Panel (for buttons: Play, Options, and Exit).
        - Position them at the center of the panel and set their texts accordingly.
- Create a new C# script (below) and attach it to the "Main Menu" Panel.

```c#
using UnityEngine;
using UnityEngine.SceneManagement;

public class MainMenu : MonoBehaviour
{
    public void PlayGame()
    {
        SceneManager.LoadScene(1);  // Loads the scene with index 1
    }

    public void QuitGame()
    {
        //Application.Quit();  // Doesn't work when the game is running in the Unity editor
        UnityEditor.EditorApplication.isPlaying = false;  // Stops running the game if it is running in the Unity editor
    }
}
```

- Select all three buttons and in the Button component add one "On Click" method.
    - Attach the Main Menu Panel GameObject to the Object field (from which we can call the methods).
- Select only the Play button and set in the On Click method the method we want to run (MainMenu.PlayGame). Do the same for the Exit button.

### Main menu: Options menu
- Duplicate the entire MainMenu Panel GameObject (together with its children buttons).
    - Rename it to "Options Menu".
- Delete the Play and Options buttons. Rename the Exit button to "Back".
- In the Options button (under the Main Menu Panel), set two On Click methods:
    - GameObject.SetActive (false) for the MainMenu GameObject.
    - GameObject.SetActive (true) for the OptionsMenu GameObject.
- In the Back button (under the Options Menu Panel), set two On Click methods:
    - GameObject.SetActive (false) for the OptionsMenu GameObject.
    - GameObject.SetActive (true) for the MainMenu GameObject.
- Disable the Options Menu panel (so it is initially disabled when the game runs).

<div align="center"><b>
  <a href="12-Navigation.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
  <a href="14-Wrapping-up.html" style="font-size:64px; text-decoration:none"> > </a>
</b></div>
