# Integrate Position Coordinate and Angles into Getgud SDK

This tutorial explains how to integrate position and rotation data from Unity and Unreal Engine 5 with the Getgud SDK. It covers coordinate system differences, angle representations, and provides practical code examples for both engines.

## Coordinate Systems

### Getgud SDK
Getgud uses a left-handed coordinate system where the Z coordinate corresponds to up/down movement:
- X: Forward/Backward (positive is forward)
- Y: Right/Left (positive is moving left)
- Z: Up/Down (positive is up)

### Unreal Engine 5
UE5 uses a right-handed coordinate system where the Z coordinate corresponds to up/down movement:
- X: Forward/Backward (positive is forward)
- Y: Right/Left (positive is right)
- Z: Up/Down (positive is up)

### Unity
Unity uses a right-handed coordinate system where the Y coordinate corresponds to up/down movement:
- X: Right/Left (positive is right)
- Y: Up/Down (positive is up)
- Z: Forward/Backward (positive is forward)

## Angle Representation

### Getgud SDK
Getgud SDK uses Pitch and Yaw for rotation, with an inverted naming convention compared to standard systems:

- **Pitch** (horizontal rotation):
  - Range: -180 to 180 degrees
  - 0°: forward, 90°: right, -90°: left, 180°/-180°: backward

- **Yaw** (vertical rotation):
  - Range: 89 to -89 degrees
  - 0°: straight ahead, 89°: looking down, -89°: looking up

### Unreal Engine 5
UE5 uses a different convention for angles:

- **Yaw**: 0 to 360 degrees (clockwise)
- **Pitch**: -90 to 90 degrees
  - Positive pitch is looking up
  - Negative pitch is looking down

### Unity
Unity uses Euler angles, but for Getgud SDK integration, we calculate angles differently:

- **Pitch**: Calculated using `Mathf.Asin(forward.y) * Mathf.Rad2Deg`
- **Yaw**: Calculated using `Mathf.Atan2(forward.x, forward.z) * Mathf.Rad2Deg`

## Position and Rotation Integration

### Unity Integration

In Unity, use the following code to calculate angles, adjust coordinates, and send data to the Getgud SDK:

```csharp
// Calculate pitch, yaw, roll
float pitch = Mathf.Asin(forward.y) * Mathf.Rad2Deg;
float yaw = Mathf.Atan2(forward.x, forward.z) * Mathf.Rad2Deg;
float roll = Mathf.Atan2(right.y, up.y) * Mathf.Rad2Deg;

Debug.Log($"Getgud: Player {player.Username} moved {transform.position.ToString()} with pitch: {pitch}, yaw: {yaw}, roll: {roll}");

BaseActionData positionBaseActionData = new BaseActionData
{
    actionTimeEpoch = DateTimeOffset.UtcNow.ToUnixTimeMilliseconds(),
    matchGuid = NetworkManager.Singleton.MatchGuid,
    playerGuid = "example-player-guid"
};

// Necessary adjustments for GetgudSDK around angles
// Swap pitch and yaw as required by the SDK
float temp = pitch;
pitch = yaw;
yaw = temp;

// Getgud most upward yaw angle is -89, most down yaw angle is 89.
// We need to change sign in Unity
yaw = -yaw;

// Same with pitch. We need to reverse Unity pitch to match Getgud format
pitch = -pitch;

// Create PositionF for position action
PositionF currentPosition = new PositionF
{
    X = transform.position.z,   // Unity's forward maps to Getgud's forward
    Y = -transform.position.x,  // Unity's right maps to Getgud's right (with negation)
    Z = transform.position.y    // Unity's up maps to Getgud's up
};

// Create RotationF for position action
RotationF currentRotation = new RotationF { Yaw = yaw, Pitch = pitch, Roll = roll };

// Create SendPositionActionInfo
SendPositionActionInfo positionInfo = new SendPositionActionInfo
{
    baseData = positionBaseActionData,
    position = currentPosition,
    rotation = currentRotation
};

// Send the position action
int positionResult = Methods.SendPositionAction(positionInfo);
```

### Unreal Engine 5 Integration

In Unreal Engine 5, the position coordinates match Getgud SDK, but angles need adjustment:

```cpp
FVector position = GetActorLocation();
FRotator CameraRotation = Controller->GetControlRotation();

float pitch = 0;
if (CameraRotation.Pitch > 180.0f)
{
    pitch = 360.0f - CameraRotation.Pitch;
}
else
{
    pitch = -CameraRotation.Pitch;
}

float yaw = 360 - CameraRotation.Yaw;

// Necessary adjustments for GetgudSDK around angles
// Swap pitch and yaw as required by the SDK
GetgudSDK::BaseActionData* outAction = nullptr;
outAction = new GetgudSDK::PositionActionData(
    g_matchGuid, UnixTimestampMillis, g_playerGuid,
    GetgudSDK::PositionF{ 	
        (float)position.X / 100.0f,  // divide position to transform from cm to meters
        -(float)position.Y / 100.0f, 
        (float)position.Z / 100.0f 
    },
    GetgudSDK::RotationF{pitch, yaw, 0});

GetgudSDK::SendAction(outAction);

delete outAction;
```

Note that we are dividing position coordinates by 100. While not required for Getgud SDK functionality, this conversion to meters aids in debugging during integration and makes movement easier to visualize in the Getgud.io modeler, as the data is not initially normalized.

## Best Practices

1. Send position updates at 32 to 128 ticks per second for smooth gameplay representation.
2. Ensure consistent coordinate system and angle conversions throughout your integration.
3. Test thoroughly to verify that player movements and rotations are accurately represented in the Getgud SDK.
