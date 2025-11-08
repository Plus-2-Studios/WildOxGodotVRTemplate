# VR IK Debug Mirror

A debug visualization tool that mirrors the player's skeleton tracking in real-time. Use this to see how the VR IK system tracks the HMD, solves arm IK, and handles leg IK for crouching.

## What It Does

The debug mirror creates a visible skeleton in your scene that copies all bone poses from the player's skeleton, including IK modifications. This lets you:

- **See your character from the outside** while playing in VR
- **Debug IK behavior** - watch how arms bend, legs crouch, etc.
- **Verify bone tracking** - ensure head, hands, and feet are moving correctly
- **Tune IK settings** - adjust hand rotations, foot spacing, etc. and see results immediately

## Quick Setup

1. **Add the scene to your world:**
   - Drag `scenes/debug/SkeletonDebugMirror.tscn` into your scene
   - Position it where you want to see the debug skeleton (e.g., in front of a mirror)

2. **Auto-detection (default):**
   - Leave `Auto Find Player` enabled
   - The mirror will automatically find your player's Skeleton3D at runtime
   - No configuration needed!

3. **Manual configuration (optional):**
   - Disable `Auto Find Player` if you want to specify the path manually
   - Set `Player Skeleton Path` to your player's Skeleton3D path
   - Example: `/root/Main/PhysicsBody/SkeletalMesh_IK/Skeleton3D`

4. **Run your VR scene:**
   - The debug skeleton will automatically mirror your movements
   - Move around, crouch, wave your hands - the debug skeleton copies everything!

## Settings

### Mirror Settings
- **Auto Find Player:** Automatically finds the player's Skeleton3D at runtime (default: true)
- **Player Skeleton Path:** Manual path to the player's Skeleton3D (only used if auto-find is disabled)
- **Update Rate:** How many times per second to update (60 = smooth, 30 = performance, 0 = every frame)

## How It Works

The debug mirror uses Godot's skeleton system to capture IK-modified bone poses:

1. **Auto-Detection:** Searches the scene tree for a Skeleton3D node, preferring one under XROrigin3D
2. **Signal Connection:** Connects to the skeleton's `skeleton_updated` signal, which fires after all IK processing
3. **Bone Copying:** Copies all bone poses including SkeletonIK3D modifications using `set_bone_global_pose_override()`
4. **Rate Limiting:** Updates at the configured rate (default 60 FPS) to balance smoothness and performance

This approach captures the final bone transforms after all SkeletonModifier3D processing (including SkeletonIK3D for arms and legs) has completed.

## Common Uses

### Testing Leg IK
- Place the mirror skeleton in front of you
- Crouch down and watch how the hips lower and knees bend
- Adjust `foot_height_offset` and `foot_spacing` in the player's VRIKComponent
- See changes immediately in the mirror

### Tuning Hand Rotation
- Place the mirror to the side
- Hold controllers in various poses
- Adjust hand rotation values in the player's VRIKComponent
- Watch the mirror's hands to verify alignment

### Verifying IK Chains
- Watch arm IK solve as you move controllers
- See leg IK respond to crouching
- Both base poses and IK overrides are mirrored accurately

## Troubleshooting

**Mirror skeleton doesn't move:**
- Check console for auto-detection messages
- If using manual path, verify `Player Skeleton Path` is correct
- Make sure the player's Skeleton3D has initialized
- Look for: "Found player skeleton at: /path/to/skeleton"

**Mirror skeleton disappears:**
- Check if the skeletal mesh is being instantiated
- Look for errors in the console about missing Skeleton3D

**Only head moves, not arms/legs:**
- This is a timing issue - make sure you're using the latest VRIKDebugMirror.gd
- The script should connect to `skeleton_updated` signal to capture IK modifications

**Performance issues:**
- Reduce `Update Rate` from 60 to 30 or lower
- Bone pose copying is very fast, but you can reduce frequency if needed

## Technical Details

The debug mirror uses Godot's skeleton modifier system:
- **skeleton_updated signal:** Fires after all SkeletonModifier3D processing completes
- **Deferred processing:** SkeletonIK3D applies modifications in deferred process, after `_process()`
- **Global pose overrides:** IK results are captured using `set_bone_global_pose_override()`
- **Bone chain walking:** For each active SkeletonIK3D, walks the bone chain from tip to root

This ensures that both direct bone modifications (like the head) and IK-driven modifications (like arms and legs) are accurately mirrored.

## Tips

- **Mirror positioning:** Place 2-3 meters in front of your spawn point for best view
- **Multiple mirrors:** You can add multiple debug mirrors at different angles
- **Disable in production:** Remove or disable debug mirrors before releasing your game
- **No configuration needed:** Auto-detection works in most cases, just drag and drop!
