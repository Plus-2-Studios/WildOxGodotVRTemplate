# VR IK Component

A VR Inverse Kinematics component that makes skeletal meshes follow your HMD and hand controllers using Godot's built-in SkeletonIK3D system.

## Quick Setup

1. **Add the component to your VRPlayer:**
   - Open `scenes/players/VRPlayer.tscn`
   - Add a child Node to the XROrigin3D and name it "VRIKComponent"
   - Attach `scripts/VRIKComponent.gd`

2. **Assign your skeletal mesh:**
   - In the Inspector, set `Skeletal Mesh Scene` to your skeletal mesh (e.g., `T-Pose.fbx`)
   - The component will automatically instantiate and find the Skeleton3D

3. **The component will auto-detect bone names:**
   - Works automatically with Mixamo skeletons (`mixamorig:LeftHand`, etc.)
   - If bones aren't detected, check the console for suggestions
   - Manually adjust bone names in the Inspector if needed

4. **Adjust hand rotation:**
   - Set `Hand Rotation X/Y/Z` to match your controller orientation
   - Enable `Mirror Right Hand` to automatically mirror left hand rotation
   - Common starting point: X=-90, Y=0, Z=0

That's it! Your skeletal mesh will follow your VR movements with proper arm IK.

## How It Works

The component uses Godot's built-in `SkeletonIK3D` nodes for arm and leg tracking:

### Arm Tracking
- **SkeletonIK3D** is created for each arm chain (upper arm → forearm → hand)
- **IK targets** follow your VR controllers with rotation and position offsets applied
- **Magnet property** pulls elbows downward for natural arm poses
- Shoulders remain fixed while arms bend naturally to reach targets

### Leg Tracking (Crouching/Squatting)
- **SkeletonIK3D** is created for each leg chain (thigh → calf → foot)
- **HMD height tracking** measures how much you crouch down
- **Hip bone adjustment** - your character's hips lower when you crouch
- **Foot targets** stay planted on the ground
- **Leg IK** naturally bends knees to bridge the distance between lowered hips and grounded feet
- This creates realistic crouching where the whole upper body lowers while feet stay planted

### Integration with VRLocomotion
- Enable **Use Locomotion Integration** to parent the skeleton to the VRLocomotion physics capsule
- When enabled, the skeleton automatically rotates with snap turns and moves with the player
- The skeleton position is calculated relative to the capsule, not the XROrigin
- If disabled or no physics body is found, the skeleton parents to XROrigin (standalone mode)
- This allows VRIKComponent to work both with and without VRLocomotion

## Settings

### Skeletal Mesh
- **Skeletal Mesh Scene:** The skeletal mesh scene to instantiate (e.g., T-Pose.fbx)
- **Skeleton:** (Optional) Manually assign a Skeleton3D if you don't use Skeletal Mesh Scene

### Tracking
- **Track Head:** Enable head tracking from HMD
- **Track Hands:** Enable hand/arm tracking from controllers
- **Track Legs:** Enable leg IK for crouching/squatting based on HMD height changes
- **Smooth Tracking:** Enable to reduce jitter from VR tracking noise (adds slight lag, disabled by default for 1:1 response)
- **Smoothing Speed:** Higher = snappier tracking, only used if smooth_tracking is enabled (default: 15.0)

### Body Settings
- **Use Locomotion Integration:** Enable to parent skeleton to VRLocomotion physics capsule (enables rotation with snap turns)
- **Body Height Offset:** Vertical position adjustment (start with 0, adjust as needed)
- **Body Forward Offset:** Forward/backward position adjustment relative to skeleton facing (positive = forward, negative = backward)
- **Align Skeleton With HMD:** Rotate skeleton to face HMD direction
- **HMD Rotation Deadzone:** Degrees of HMD rotation before body starts rotating (default: 45°, prevents body from rotating with every small head turn)
- **Body Rotation Smoothing:** How smoothly the body rotates to follow HMD (higher = faster, default: 5.0)
- **Foot Height Offset:** Offset feet above/below ground level (positive = higher, negative = lower)
- **Foot Spacing:** Distance between feet in meters (default: 0.3m)

### Hand Orientation
- **Hand Rotation X/Y/Z:** Euler angles to rotate hands (in degrees)
- **Hand Rotation Order:** Order to apply rotations (XYZ, XZY, YXZ, etc.)
- **Mirror Right Hand:** Automatically mirror left hand rotation for right hand
- **Hand Position Offset:** Offset from controller to hand in controller's local space (useful for moving grip point from wrist to palm)
- **Mirror Right Hand Position:** Automatically mirror X offset for right hand
- **Override Right Hand:** Use separate rotation and position settings for right hand

### Debug
- **Show Debug Logs:** Enable detailed debug logging to console (bone mappings, IK initialization, etc.)
- **Debug On Button Press:** Only print debug info when right thumbstick is clicked
- **Show Controller Axes:** Visualize controller coordinate axes (for debugging hand orientation)

### Bone Mappings
The component auto-detects bone names for Mixamo skeletons. You can manually override these:
- **Head Bone Name:** Head bone (default: auto-detected)
- **Neck Bone Name:** Neck bone (default: auto-detected)
- **Spine Bone Name:** Spine bone (default: auto-detected)
- **Left/Right Shoulder Bone Name:** Shoulder bones
- **Left/Right Arm Bone Name:** Upper arm bones (arm IK chain starts here)
- **Left/Right Forearm Bone Name:** Forearm bones
- **Left/Right Hand Bone Name:** Hand bones (arm IK chain ends here)
- **Hips Bone Name:** Hip/pelvis bone
- **Left/Right UpLeg Bone Name:** Thigh bones (leg IK chain starts here)
- **Left/Right Leg Bone Name:** Calf/shin bones
- **Left/Right Foot Bone Name:** Foot bones (leg IK chain ends here)

## Tips

### Hand Position
- Use **Hand Position Offset** to move the IK target from controller origin to desired grip point
- Common adjustments: Move forward (Z+) to shift from wrist to palm
- Position offset is in **controller's local space**: X=right, Y=up, Z=forward
- Enable **Mirror Right Hand Position** to automatically flip X offset for symmetry

### Hand Rotation
- Start with **X=-90, Y=0, Z=0** for most skeletons
- If hands are rotated wrong, try different rotation orders (XYZ vs YXZ)
- Enable **Mirror Right Hand** to automatically flip rotation for symmetry

### Arm Tracking
- The IK system uses the bone chain: **Upper Arm → Forearm → Hand**
- Shoulders stay fixed while arms bend naturally
- If elbows point wrong, the SkeletonIK3D `magnet` property controls elbow direction (currently set to pull downward)

### Leg Tracking & Crouching
- The system calibrates your initial HMD height at startup
- When you crouch (HMD goes lower), the hip bone lowers by the same amount
- Foot targets stay planted on the ground
- Leg IK bends the knees to reach between lowered hips and grounded feet
- Adjust **Foot Height Offset** if feet sink into or float above the ground
- Adjust **Foot Spacing** to change stance width

### Performance
- Smooth tracking adds slight latency but looks better
- Increase smoothing speed if tracking feels sluggish
- Disable debug output in production for better performance

## Common Bone Names

**Mixamo skeletons:** Prefix with `mixamorig:` (e.g., `mixamorig:Head`, `mixamorig:LeftHand`, `mixamorig:LeftArm`)

**Standard skeletons:** Usually just `Head`, `LeftHand`, `RightHand`, `LeftArm`, etc.

## Troubleshooting

**Hands not aligned with controllers:**
- Adjust hand rotation X/Y/Z values
- Try different rotation orders

**Arms bending weirdly:**
- Check that bone mappings are correct
- Verify that Upper Arm (not Shoulder) is set as the IK root

**Skeleton not following HMD:**
- Enable "Align Skeleton With HMD"
- Adjust "Body Height Offset" if floating/underground

**Performance issues:**
- Disable "Show Debug"
- Reduce "Smoothing Speed" slightly
