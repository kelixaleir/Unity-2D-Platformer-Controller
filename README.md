# Unity 2D Platformer Controller
=======================

<!---%=description%-->

A customizable 2D platformer motor that interacts with Unity's physics engine to do mechanics such as double jumps, wall jumps, and corner grabs. Includes a player controlled prefab that can be dropped into any scene for immediate support.

<!---%=obtain%-->

####Obtain!####
[Releases](https://github.com/cjddmut/Unity-2D-Platformer-Controller/releases)

If you'd like the most up to date version (which is the most cool), then pull the repo or download it [here](https://github.com/cjddmut/Unity-2D-Platformer-Controller/archive/develop.zip) and copy the files in Assets to your project's Assets folder.

<!---%=docrest%-->

## Setup

For immediate player support, drop the Basic Player Controller prefab into the scene and update the field Environment Check Mask to the layer that contains your environment. For more complicated interaction, interface with PlatformerMotor2D's members and methods.

## Requirements of PlatformerMotor2D

### Rigidbody2D ###

A Rigidbody2D is required for the motor as it uses rigidbody2D.MovePosition() to move the GameObject. The Rigidbody2D should have Fixed Angle checked and Is Kinematic needs to be unchecked. Mass has no impact on the motor.

The motor sets the drag and gravity of the Rigidbody2D to 0 and handles deceleration and gravity on its own. If disabled then the motor will restore the original values of the Rigidbody2D. This is useful if you a player to lose control and allow Unity's physics to take over.

Since the motor uses MovePosition, extrapolate will not have any effect.

If the motor appears to punch through/into walls when dashing or falling then set Collision Detection to continuous.

### Collider2D ###

The motor requires that a Collider2D be present on the GameObject or that a Collider2D is specified to the motor through the colliderToUse property. The motor uses the bounds of the Collider2D to understand where to check for contact with surfaces.

## PlatformerMotor2D Inspector Properties

### General ###

**Static Environment Layer Mask** - This tells the motor what layer collisions to consider the environment (to determine if on the ground, wall, or corner).

**Moving Platform Layer Mask** - What layer contains moving platforms. The motor uses this knowledge to grab a rigidbody2D from the platforms.

**Environment Check Distance** - This is how far out the motor will check for the environment.  

### Movement ###

**Ground Speed** - Maximum ground speed.

**Time to Ground Speed** - The time, in seconds, it will take to reach ground speed. This is used to calculate acceleration. A value of 0 mean instantaneous movement.

**Ground Stop Distance** - If at full speed, how far will the motor skid to a stop.

**Allow Direction Change In Air** - If true, then the motor's x velocity can be changed while in air. If false, then the motor's x velocity cannot be changed when in the air.

**Horizontal Air Speed** - Maximum speed the motor will move horizontally while in the air.

**Time to Air Speed** - The time, in seconds, it will take to reach air speed. This is used to calculate acceleration.

**Air Stop Distance** - If at full air speed, how far will the motor 'skid' to a stop.

**Max Fall Speed** - Maximum fall speed (only y axis when negative).

**Max Fast Fall Speed** - Maximum fall speed when falling fast.

**Fast Fall Gravity Multiplier** - Gravity multiplier when falling fast. A value of 1 means no different, higher values mean faster fall acceleration.

### Jumping ###

**Base Jump Height** - The height, in Unity units, that the motor will jump to.

**Held Extra Jump Height** - If the motor is informed that the jump is held then this is the additional height the character will jump.

**Grace For Jump** - The time in seconds that a jump will be allowed after it become invalid. This would be like allowing a jump even though the player has technically walked off the edge. Setting this to a low value (such as 0.1) can feel like a better experience for the player. If undesired then just set to 0.

**Air Jumps Allowed** - This sets the number of air jumps the character is allowed to perform. Setting it to 0 will disable air jumping altogether.

**Allow Wall Jump** - If jumping off the wall is allowed.

**Wall Jump Multiplier** - The base jump speed is calculated from Base Jump and Extra Jump Height. The multiplier multiplies the result. Leave at 1 for no change.

### Wall Cling ###

**Allow Wall Cling** - If the motor should cling to the walls (sticking in place).

**Wall Cling Duration** - The time, in seconds, that the motor will stick to walls. A large value (say 1000000) is effectively infinite.

### Wall Slide ###

**Allow Wall Slide** - If the motor should consider any wall sliding calculations. Wall sliding is when the character would slow down while 'sliding' down the wall.

**Wall Slide Speed** - The speed that the character will slide down the wall.

### Corner Grabs ###

**Allow Corner Grab** - If corner grabbing is allowed. 

**Corner Grab Duration** - The time, in seconds, that the motor will stick to corners.

**Corner Jump Multiplier** - The multiplier on a corner jump from the calculated speed.

**Corner Distance Check** - A corner is considered grabbed if the upper corners of the collider do not intersect with the environment but the sides do. The value changes the consideration for box checks dimensions.

### General Wall Interactions ###

**Wall Interaction Threshold** - The input threshold for wall clings, corner grabs, and slides. Could be set to higher to prevent unwanted sticking to walls.

### Dashing ###

**Allow Dashing** - Is dashing allowed?

**Dash Distance** - The distance covered by the dash.

**Dash Duration** - The duration of the dash.

**Dash Cooldown** - How long, in seconds, before the motor will allow dash again?

**Dash Easing Function** - The easing function of the dash. For a dash that movement with a consistent speed pick linear.

**Gravity Delay After Dash** - The delay, in seconds, before gravity is reenabled after a dash. This can allow a short pause after an air dash before falling.

### PlatformerMotor2D Members ###

```csharp
float normalizedXMovement
```

Set the x movement direction. This is multiplied by the max speed. -1 is full left, 1 is full right. Higher numbers will result in faster acceleration.


```csharp
float timeScale
```

Set the time scale for the motor. This is independent of the global time scale. Negative values are not supported.


```csharp
Vector2 Velocity
```

The velocity of the motor. This should be queried instead of the rigidbody's velocity. Setting this during a dash doesn't have any meaning. NOTE: Setting rigidbody2D.velocity can have unexpected results. If you want to let Unity Physics take over then disable the motor first.


```csharp
MotorState motorState // Readonly

enum MotorState
{
    OnGround,
    Jumping,
    Falling,
    FallingFast,
    Sliding,
    OnCorner,
    Clinging,
    Dashing,
    Frozen
}
```

Call this to get state information about the motor. This will be information such as if the object is in the air or on the ground. This can be used to set the appropriate animations.

```csharp
CollidedSurface collidingAgainst // Readonly

[Flags]
public enum CollidedSurface
{
    None = 0x0,
    Ground = 0x1,
    LeftWall = 0x2,
    RightWall = 0x4,
    Ceiling = 0x8
}

```

State information on what the motor believes itself to be colliding against. These are flags so any number may be turned on.

```csharp
bool facingLeft // Readonly
```

Since the motor needs to know the facing of the object, this information is made available to anyone else who might need it.

```csharp
bool fallFast
```

Set this true to have the motor fall faster. Set to false to fall at normal speeds.

```csharp
float amountFallen // Readonly
```

Returns the amount of distance the motor has fallen. Includes fallen fast distance.

```csharp
float amountFastFallen // Readonly
```

Returns the amount of distance the motor has fallen fast.

```csharp
float amountJumpedFor // Readonly
```

Returns the amount the motor has jumped. This ceases to keep calculating after the motor starts to come down.

```csharp
Vector2 dashDirection // Readonly
```

Returns the direction of the current dash. If not dashing then returns Vector2.zero.

```csharp
float distanceDashed // Readonly
```

Returns the amount of distance dashed. If not dashing then returns 0.

```csharp
bool canDash // Readonly
```

If the motor is currently able to dash.

```csharp
bool jumpingHeld
```

If jumpingHeld is set to true then the motor will jump further. Set to false if jumping isn't 'held'.

```csharp
bool frozen
```

Setting frozen to true will put the motor in a 'frozen' state. All information will be saved and set once unfrozen (the motor also reduce gravity to 0).

Note: This isn't a way to turn off the motor. To turn off the motor, simply set the script to disabled.

```csharp
MovingPlatformMotor2D connectedPlatform

```

Returns the moving platform that the motor is coupled with. If null then no moving platform.

```csharp
Collider2D colliderToUse
```
Set this to use a specific collider for checks instead of grabbing the collider from gameObject.collider2D.

```csharp
Action onDash
Action onDashEnd
Action onJump
Action onAirJump
Action onWallJump
Action onCornerJump
Action onLanded
```

Attach to these delegates to receive notifications for dash, dash end, jumping, and landing events.

### PlatformerMotor2D Methods ###

```csharp
void Jump()
```

Call this to have the GameObject try to jump, once called it will be handled in the FixedUpdate tick. The y axis is considered jump.

```csharp
void Jump(float customHeight)
```

Jump that allows a custom height. The extraJumpHeight is still applicable.

```csharp
void ForceJump()
```

This will force a jump to occur even if the motor doesn't think a jump is valid. This function will not work if the motor is dashing.

```csharp
void ForceJump(float customHeight)
```

Force a jump with a custom height. The extraJumpHeight is still applicable.

```csharp
void EndJump()
```

Call to end a jump. Causes the motor to stop calculated held speed for a jump.

```csharp
void ResetAirJump()
```

Resets the state for the jump counter to 0. This doesn't do anything if air jumps is set to 0.

```csharp
void Dash()
```

Call this to have the GameObject try to dash, once called it will be handled in the FixedUpdate tick. This casues the object to dash along their facing (if left or right for side scrollers).

```csharp
void ForceDash()
```

Forces the motor to dash even if dash isn't available yet.

```csharp
void Dash(Vector2 dir)
```

Send a direction vector to dash allow dashing in a specific direction.

```csharp
void ForceDash(Vector2 dir)
```

forces the motor to dash along a specified direction.

```csharp
void EndDash()
```

Call to end dash immediately.

```csharp
void DisconnectFromPlatform()
```

Decouples the motor from the platform. This could be useful for a platform that throw the motor in the air. Call this when when the motor should disconnect then set the appropriate velocity.

## PlayerController2D

The PlayerController2D script is a simple script that connects player input to the motor. This is set up as an example and it is encourage to write your own script that interacts with the motor.

## Moving Platforms

Given the complexity of the Unity 2D Physics engine, moving platforms have a few special rules in order to work. Each moving platform is required to have a MovingPlatformMotor2D attached to it and it is **IMPORTANT** that the MovingPlatformMotor2D script and any other script that moves the platforms is executed before the PlatformerMotor2D script in the Script Execution Order Settings (Edit -> Project Settings -> Script Execution Settings). The platform should update its position in FixedUpdate() and can leverage the velocity/position from MovingPlatformMotor2D.

See the Moving Platform scene for examples.

### MovingPlatformMotor2D Members ###

```csharp
Vector2 velocity
```

Set this to drive the platform by speed. This velocity is used to determine if the player should fall off the platform if the platform is falling too fast. Updates the transform position in FixedUpdate.

```csharp
Vector2 position
```

Quick access to transform position typed to Vector2.

```csharp
Action<PlatformerMotor2D> onMotorContact
```

Invoked when a motor makes contact with a moving platform and is considered 'attached' to it.

### One Way Platforms ###

To acheive one way platforms with the motor. Have a environment piece with an edge collider at the top and a PlatformEffector2D component attached. Be sure to check Use One Way and to check Used By Effector on the edge collider.

The motor will ignore all "Used By Effector" collisions except for the ground giving the desired effect. Note at this point no wall interaction mechanics are supported for one way platforms.

See the Platformer scene for an example.

## FAQs

**What's with all the required layers?**
This is mostly an optimization. MovingPlatformMotor2D is required on moving platforms but querying on a regular environment that doesn't have one generates garbage. The more garbage generated then the more often the garbage collection will kick in. If the layers present a problem then it would be easy to change the check to tags instead in the motor.

**Can I use PlatformerMotor2D for controlling AI movements?**
Sure can. PlatformerMotor2D doesn't know anything about inputs, it just acts on information passed to it. An AI script can interface with the motor similarly how a player controller script could. A very simple example is included in the SimpleAI scene.

**I let go of the joystick and my GameObject isn't sliding the distance it is supposed to!**
If you're using the supplied PlayerController2D script or one of your own in which you use Input.GetAxis() then there's a built in deceleration in what Input.GetAxis() returns. This can definitely be impacting the distance the GameObject skids to a stop! To see a true skid to stop, set normalizedXMovement to zero.

**Something isn't working right!**
Well, this happens. Please open up an [issue](https://github.com/cjddmut/Unity-2D-Platformer-Controller/issues)!

## Games using the Unity 2D Platformer Controller

[Beep Boop](http://cjkimberlin.com/games/LD32/WebGL/index.html) - Help Beep Boop navigate past dangers using its unconventional weapon to send it flying. Created solo in 72 hours for Ludumdare 32.

If you are using the motor for your game, add it and set up a pull request or let me know at [@cjkimberlin](https://twitter.com/cjkimberlin)!
<!---%title=Unity 2D Platformer Controller%-->
<!---%download=https://github.com/cjddmut/Unity-2D-Platformer-Controller/releases/download/v0.2.0/PC2D_v0.2.0b.unitypackage%-->
<!---%github=https://github.com/cjddmut/Unity-2D-Platformer-Controller%-->
