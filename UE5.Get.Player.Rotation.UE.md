## OpenAI ChatGPT v1.2025.343 UE.Get.Player.Rotation.UE5.md

**Q: In Unreal Engine Blueprints, I need to get the current rotation of the player (not the camera).**

**A:** Use the Pawn/Character’s Actor rotation, not the Controller/Camera rotation.

* If you have a reference to the player pawn:

  * `Get Player Pawn (0)` → `Get Actor Rotation`
* If you are inside the player Character Blueprint:

  * `Self` → `Get Actor Rotation`

Notes (common gotchas):

* `Get Control Rotation` = controller/camera aim rotation (often what the camera uses), not necessarily the pawn’s body.
* If you specifically want the character’s facing direction driven by movement (and not aim):

  * `Get Velocity` → `Get Rotation from XVector` (or `Make Rot from X`), but only valid when moving.
* If the pawn uses `Use Controller Rotation Yaw = false` and `Character Movement → Orient Rotation to Movement = true`, then `Get Actor Rotation` is the body yaw you want.
