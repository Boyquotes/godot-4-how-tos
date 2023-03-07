## Gdscript

### Physics

The first major change in Godot Physics are the new names. The `KinematicBody` nodes became `CharacterBody` nodes, to better represent their main functionality: to provide an easy-to-use physics node with custom movement.

Physics API were also simplified for `CharacterBody`. What previously were three types of movement that required a set of arguments (`move_and_collide`, `move_and_slide` and `move_and_slide_with_snap`) became the method `move_and_slide` that requires no arguments. Instead, `CharacterBody` now have parameters to set so it can simulate the previous behavior, such as `floor_max_angle` and `floor_stop_on_slope`. `CharacterBody` nodes also have a `velocity` attribute that must be set before calling `move_and_slide` on a `_physics_process` frame.

The following code example make this difference clear:

```
# Godot 3.0
extends KinematicBody

const PLAYER_SPEED = 10.0


func _process(delta: float) -> void:
	var input_direction = Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
	var move_direction = Vector3(input_direction.x, 0.0, input_direction.y)
	move_and_slide(move_direction * PLAYER_SPEED, Vector3.UP)
```

```
# Godot 4.0
extends CharacterBody3D

const PLAYER_SPEED = 10.0


func _physics_process(delta):
	var input_direction = Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")
	var move_direction = Vector3(input_direction.x, 0.0, input_direction.y)
	velocity = move_direction * SPEED

	move_and_slide()

```


Another change in Godot 4.0 is the removal of "infinite inertia" on `CharacterBody` nodes. Previously in Godot 3.x, a `KinematicBody` node would be able to push `RigidBody` nodes around without changing any inspector setting. This happened because `KinematicBody` nodes doesn't have the concept of weight or force implicitly like `RigidBody` nodes does, so the Engine would assume those nodes had "inifinite inertia" and  would always be able to push other `RigidBody` nodes around. This specific behavior adds unpredictability to the physics engine, so it was removed and now `KinematicBody` nodes can't push other nodes, unless the user explicitly calls `apply_force` on `RigidBody` nodes or implement other types of pushing behaviors themselves.

An easy way to replicate the "infinite inertia" behavior in Godot 4.0 is to set the nodes in different collision layers. For example, if the Player character is in the `collision_layer` `1`, set the pushable `RigidBody` to be on `collision_mask` `1`, but **not** on `collision_layer` `1`. This will make the `RigidBody` scans for collisions in `collision_layer` `1`, but the Player object will ignore the collision, which will push the object around without affecting the Player.

### Tweens

The tweening system has undergone a major revamp, resulting in the removal of the old Tween node in favor of `SceneTreeTween`. This new API provides increased flexibility for users.

To create a tween, users can utilize the `create_tween()` method, which generates a tween object capable of receiving tweeners. The tween starts immediately upon its creation, eliminating the need to call `tween.start()`. It's essential to note that a tween without a tweener will result in errors.

Tweeners can't be manually generated; dedicated methods must be called to append them to the tween. By default, the tweens are executed sequentially. This default execution order can be changed and fine tuned with `set_parallel()` and `chain()`.

### Yield to Await

The yield keyword was replaced to await. The change are mainly syntactical, `yield(self, "signal_name")` and `await signal_name` will produce identical result.

#### Example

```
# Godot 3.0
signal finish_counting

func _ready():
	var goal = 3
	print("Let's count!")
	count_to(goal)
	yield(self, "finish_counting")
	print("I've counted to " + str(goal) + "!")

func count_to(goal : int):
	for i in goal:
		yield(get_tree().create_timer(1.0), "timeout")
		print(i + 1)
	emit_signal("finish_counting")
```

```
# Godot 4.0
signal finish_counting

func _ready():
	var goal = 3
	print("Let's count!")
	await count_to(goal)
	print("I've counted to " + str(goal) + "!")

func count_to(goal : int):
	for i in goal:
		await get_tree().create_timer(1.0).timeout
		print(i + 1)
	emit_signal("finish_counting")
```
