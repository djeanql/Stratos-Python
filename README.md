# Stratos Python

A raylib (pyray) based 2D engine for building games with seamless altitude depth, from ground level to space. This engine implements a vertical perspective scaling system designed for top-down 2D games that need to simulate depth along the vertical axis (altitude/depth). It creates natural perspective scaling where objects appear larger when near the camera and smaller when far away, while maintaining consistent visibility of the player's focal object. This system is used by my work-in-progress 2D space exploration game, where players can fly long distances in space and then descend to the surface of a planet. As a player's ship descends, the camera decends with it to keep the ship the same scale, while everything below gets larger. This effectively adds a third dimension to a 2D game.

This is the initial python-based version of Stratos, utilising raylib's python bindings. However this comes at a performance cost, so once past the proof of concept stage this engine will be rewritten in C++ with standard raylib.

Here is a simple example of engine useage for a space game. The player controlls a ship which can descend to a planet below. The player is able to walk around in their ship by pressing y. This is incomplete code, purely for demonstrating key features of Stratos.

```py
import math
import stratos

MAX_HEIGHT = 50000  # m
MIN_SPEED = 5       # m/s
MAX_SPEED = 6000    # m/s
ASCENT_SPEED_FACTOR = 0.7
DESCENT_SPEED_FACTOR = 1.0

game = stratos.Game()
scene = stratos.Scene()
game.set_scene(scene)

planet = scene.spawn(
    "planet",
    shape="sphere",
    width=10000,
    colour="green",
    position=stratos.Pos(0, 0, 0),
)

tree = planet.spawn_child(
    "tree",
    sprite="tree.png",
    width=2,
    position=stratos.Pos(50, 60, 0),
    collision_shape=True,
)

ship = scene.spawn(
    "ship",
    sprite="ship.png",
    width=15,
    position=stratos.Pos(0, 0, 500),
    collision_mask="ship_collision.png",
)

player = ship.spawn_child(
    "player_ship_interior",
    sprite="player.png",
    width=1.5,
    hidden=True,
)

# camera is 400m above the ship at all times
camera = ship.spawn_child(
    "camera",
    camera=stratos.Camera(position=stratos.Pos(0, 0, 400)),
)

interior_walk_mode = False

def clamp(v, lo, hi):
    return lo if v < lo else hi if v > hi else v

# increase speed with height, smoothly
def speed_at_height(height_m: float) -> float:
    h = clamp(height_m, 0.0, float(MAX_HEIGHT))
    t = h / float(MAX_HEIGHT)               # 0..1
    t = math.log10(1.0 + 9.0 * t)           # still 0..1, but curved
    return MIN_SPEED + t * (MAX_SPEED - MIN_SPEED)

def update():
    global interior_walk_mode

    # Toggle interior mode
    if stratos.key_pressed("y"):
        interior_walk_mode = not interior_walk_mode

        if interior_walk_mode:
            player.hidden = False
            ship.sprite = "ship_interior.png"
            ship.collision_mask = "ship_interior_collision.png"
        else:
            player.hidden = True
            ship.sprite = "ship.png"
            ship.collision_mask = "ship_collision.png"

    dt = stratos.dt
    speed = speed_at_height(ship.position.z) * dt

    if stratos.key_down("w"):
        ship.position.y += speed
    if stratos.key_down("s"):
        ship.position.y -= speed

    if stratos.key_down("q"):  # down
        ship.position.z -= speed * DESCENT_SPEED_FACTOR
    if stratos.key_down("e"):  # up
        ship.position.z += speed * ASCENT_SPEED_FACTOR

    ship.position.z = clamp(ship.position.z, 0.0, float(MAX_HEIGHT))

scene.set_camera(camera)
scene.set_update(update)
game.run()


```
