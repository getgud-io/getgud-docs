# Understanding Angles: Quick Tutorial

In the GetGud C++ SDK, understanding angles and how to work with the Position action is crucial for accurately representing player movements and interactions within your game. This document aims to provide clarity on these concepts.

## Angles Representation

When working with the Position action, it's essential to understand how angles are represented in the SDK. The SDK follows a standardized approach for representing angles, which includes the concepts of yaw, pitch, and roll.

### Yaw

**Definition:** Yaw represents the horizontal rotation or left/right movement of the player's view.  
**Range:** In the context of the GetGud SDK, yaw typically ranges from -180 to 180 degrees.  
**Orientation:**
- Yaw of 0 corresponds to facing forward.
- Positive yaw values denote a clockwise rotation (turning right).
- Negative yaw values denote a counterclockwise rotation (turning left).

**Yaw Range: -180 to 180**  
In Counter-Strike, yaw ranges from -180 to 180 degrees, where:
- Yaw of 0 looks at the point directly to the east.
- Yaw of -180 looks at the point directly to the west.
- Yaw of 90 looks at the point directly to the south.
- Yaw of -90 looks at the point directly to the north.

When transitioning from 0 to -180, the player turns right, and when transitioning from 0 to 180, the player turns left. It's essential to align the game's orientation with the correct coordinates on the map.

### Pitch

**Definition:** Pitch represents the vertical rotation or up/down movement of the player's view.  
**Range:** In the GetGud SDK, pitch typically ranges from 89 to -89 degrees.  
**Orientation:**
- Pitch of 0 corresponds to looking straight ahead.
- Positive pitch values denote looking upward.
- Negative pitch values denote looking downward.

**Pitch Range: 89 to -89**  
In Counter-Strike, pitch ranges from 89 to -89 degrees, where:
- A pitch of 89 degrees corresponds to looking downward (most down point).
- A pitch of -89 degrees corresponds to looking upward (most up point).

### Center View

In Three.js, the center view is typically considered to be at 90 degrees, with below being 0 degrees and up being 180 degrees.

### Roll

**Definition:** Roll represents the rotation around the player's viewing axis.  
**Range:** Roll is less commonly used in the context of the GetGud SDK but may be relevant for specific game mechanics.  
**Orientation:**
- Roll typically ranges from -180 to 180 degrees.
- Positive values indicate a clockwise rotation.
- Negative values indicate a counterclockwise rotation.

## Usage

To send a Position action to the SDK, follow these steps:
1. Construct a `PositionActionData` object with relevant data.
2. Add the action to a deque of actions.
3. Send the deque of actions using the `SendActions` method.

```cpp
// Example of creating and sending a Position action
std::deque<GetGudSdk::BaseActionData*> actionsToSend {
 new GetGudSdk::PositionActionData(
 matchGuid, curTimeEpoch, "player-5",
 GetGudSdk::PositionF{20.32000f, 50.001421f, 0.30021f},
 GetGudSdk::RotationF{10, 20})
};

// Send the actions to the SDK
GetGudSdk::SendActions(actionsToSend);

Our code comments when we implemented angles:

Yaw
// counter strike yaw is from -180 to 180. 0 corresponds to center.
    // here is how angles work in cs
    // Ang 0 looks at point (+inf, 0)
    // Ang -180 looks at point (-inf, 0)
    // Ang 90 looks at point (0, +inf)
    // Ang - 90 looks at point (0, -inf)
    // for us it means we should align our north point with correct coordinate on map
    // when you go from 0 to -180 you turn right. when you go 0 to 180 you turn left.
    // when you are at -180 and you continue to turn right, we switch to -180 and it starts to decrease.
    // 0 right, 90 up, 180 left, 270 down, 360 right, 450 up
    // 0 right, -90 down, -180 left ..

Pitch
// counter strike pitch is from 89 to -89.
    // 89 corresponds to most down point, -89 corresponds to most up point
    // three js center view is 90 degrees, below is 0, up is 180
