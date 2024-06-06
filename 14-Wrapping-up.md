# 14. Wrapping up
- In this section, we will cover weapon holding and audio.

## Weapon
- Here are two main methods to make the player character hold a weapon.

### Basic method
- If your character's arms are already in a weapon-holding position, attach the weapon 3D model to the character's primary hand.
    - Locate the main hand in the character skeleton in the scene hierarchy: Hips > Spine > RightShoulder > RightArm > RightForeArm > RightHand.
    - Place the weapon 3D model as a child of the RightHand and position/rotate it accordingly.

### IK method
- Attach a weapon using a two-bone IK (Inverse Kinematics) constraint.
    - Tutorial: https://www.youtube.com/watch?v=fB0P0C_3sPU

## Audio
- Add the "Audio Source" component to the weapon (which we want to produce the shooting sound).
    - Attach the appropriate audio clip to the "AudioClip" field.
    - Disable the "Play On Awake" field.
- In the Shooting.cs script:
    - Declare `public AudioSource shootAudioSource;`
        - Set that variable from the inspector to the audio source on the weapon.
    - Add `shootAudioSource.Play();` inside the Shoot() method.

<div align="center"><b>
  <a href="13-User-interface.html" style="font-size:64px; text-decoration:none"> < </a>
  <a href="Contents.html" style="font-size:64px; text-decoration:none"> ^ </a>
</b></div>
