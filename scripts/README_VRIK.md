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

The component uses Godot's built-in `SkeletonIK3D` nodes for arm tracking:
- **SkeletonIK3D** is created for each arm chain (upper arm → forearm → hand)
- **IK targets** follow your VR controllers with rotation and position offsets applied
- **Magnet property** pulls elbows downward for natural arm poses
- Shoulders remain fixed while arms bend naturally to reach targets

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
- **Smooth Tracking:** Enable to reduce jitter from VR tracking noise (adds slight lag, disabled by default for 1:1 response)
- **Smoothing Speed:** Higher = snappier tracking, only used if smooth_tracking is enabled (default: 15.0)

### Body Settings
- **Use Locomotion Integration:** Enable to parent skeleton to VRLocomotion physics capsule (enables rotation with snap turns)
- **Body Height Offset:** Vertical position adjustment (start with 0, adjust as needed)
- **Body Forward Offset:** Forward/backward position adjustment relative to skeleton facing (positive = forward, negative = backward)
- **Align Skeleton With HMD:** Rotate skeleton to face HMD direction
- **Neck Offset:** Offset from HMD to neck position

### Hand Orientation
- **Hand Rotation X/Y/Z:** Euler angles to rotate hands (in degrees)
- **Hand Rotation Order:** Order to apply rotations (XYZ, XZY, YXZ, etc.)
- **Mirror Right Hand:** Automatically mirror left hand rotation for right hand
- **Hand Position Offset:** Offset from controller to hand in controller's local space (useful for moving grip point from wrist to palm)
- **Mirror Right Hand Position:** Automatically mirror X offset for right hand
- **Override Right Hand:** Use separate rotation and position settings for right hand

### Debug
- **Show Debug:** Print bone mapping and tracking info to console
- **Debug On Button Press:** Only print debug when right thumbstick is clicked
- **Show Controller Axes:** Visualize controller coordinate axes

### Bone Mappings
The component auto-detects bone names for Mixamo skeletons. You can manually override these:
- **Head Bone Name:** Head bone (default: auto-detected)
- **Neck Bone Name:** Neck bone (default: auto-detected)
- **Spine Bone Name:** Spine bone (default: auto-detected)
- **Left/Right Shoulder Bone Name:** Shoulder bones
- **Left/Right Arm Bone Name:** Upper arm bones (IK chain starts here)
- **Left/Right Forearm Bone Name:** Forearm bones
- **Left/Right Hand Bone Name:** Hand bones (IK chain ends here)
- **Hips Bone Name:** Hip/pelvis bone

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
