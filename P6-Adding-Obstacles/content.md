---
title: Adding obstacles
slug: adding-obstacles
---

You are going to be creating obstacles for the player to navigate and
implementing a mechanic to randomize their position to challenge the player.

# Creating the obstacle

> [action]
> Create a new _SpriteKit Scene File_ by selecting
> `File > New > File > SpriteKit Scene`:
>
> ![Selecting the SKS File](../Tutorial-Images/xcode_add_sks.png)
>
> Save the file as `Obstacle.sks`
>
> ![Saving the SKS File](../Tutorial-Images/xcode_add_sks_obstacle.png)

You will be constructing the obstacle with two carrots, one at the top and one
at the bottom and a special invisible _Color Sprite_ between the carrots, which
you will later use to score the player.

> [action]
> Modify the size and anchor point of this new obstacle scene
> properties as shown:
> ![Modify the obstacle scene size](../Tutorial-Images/xcode_obstacle_scene_size.png)
>
> Set the _Anchor Point_ to `(0.5, 0.5)` and _Size_ to `(51, 820)`.
>
> Drag `carrot_top.png` and `carrot_bottom.png` from your _Media Library_, and
> snap the carrots in place so that they are flush against the top and bottom
> parts of the scene respectively.
>
> Next add a _Color Sprite_ from the object library (That trusty `+` button
> again!) and position it in between the carrot's as shown: Set the _Name_ to
> `goal`
>
> ![Add the goal sprite](../Tutorial-Images/xcode_obstacle_goal_dims.png)
>
> If you need to double check, look at the _Position_ and _Size_ of the _Color
> Sprite_ in the screenshot to make sure you have the right dimensions.
>
> You want to change the color values of the sprite to be transparent as the
> player does not need to see it, it acts as an invisible sensor or trigger.
> This will be discussed in more detail in the next chapter.
>
> Your final obstacle should look something like this:
>
> ![Finished obstacle](../Tutorial-Images/xcode_obstacle_sks.png)

<!-- -->

> [info]
>
> So how did I know that the scene size should be exactly `51` pixels
> wide and `820` tall? Well, sometimes you need to play in the visual editor and
> then adjust, I saw the carrot's had a width of `51` so I used that value. The
> default height of `820` (iPhone 5) was no coincidence as the assets were
> designed to fill an iPhone 5 screen height, leaving a suitable `140` pixel gap
> for the player to pass through.

## Adding an obstacle

You can test if you've set up the obstacle correctly by adding it to the
_GameScene_.

> [action]
> Open `GameScene.sks` and drag `Obstacle.sks` into the scene, if you
> don't see anything, do a quick _Save_ to refresh SpriteKit.
>
> I'm sure you noticed the obstacle is sitting behind the crystals, you will
> want it to sit in front of the crystals yet also sit behind the ground.

Can you fix this? Remember you've ran into similar _Z-Position_ issues when you
initially setup the scene.

> [solution]
> Tweak the _scrollLayer_ which contains the ground nodes, set
> _Z-Position_ to `2` and set the z-Position of the _Obstacle_ node to `1`.

<!--  -->

> [action]
> Open `GameScene.sks`. Select the obstacle node and give it the name:
> "obstacle"
>
> Make sure you've set its position to `(383.769, 296.5)` as well!

Now this hopefully look a lot like this.

![Checking the obstacles](../Tutorial-Images/xcode_add_obstacle_gamescene.png)

# Dynamic obstacle generation

Time to learn about dynamic obstacle generation or DOG for short :] You added a
copy of `Obstacle.sks` to your scene. This is a sprite node. Your game will copy
this node to create a relentless stream of obstacles for the player to avoid. To
do this you will need a reference to the source obstacle.

In the image above you may have noticed that the obstacle was placed outside the
visible area of the game scene. This obstacle will always sit outside of view
and act as a source for obstacles that will move across the screen.

> [action] In `GameScene.swift`, add a variable at the top of your GameScene
> class to hold a reference to the source obstacle:
>
```
var obstacleSource: SKNode!
```
>
> Now set the value of the `obstacleSource` to the "obstacle" node in
> `GameScene.sks` using `child(withName:)`. In `didMove(to view:)`, add the
> following at the end of the method:
>
```
/* Set reference to obstacle Source node */
obstacleSource = self.childNode(withName: "obstacle")
```

## Obstacle layer

It will be useful to create a layer to hold all of the obstacles. You can do
this with a node in `GameScene`. By attaching all of the obstacles to a parent
node, they will draw at the z position of that node, and allow you to move all
of the obstacles by moving the node.

> [action] Open `GameScene.sks` and drag an _Empty_ node into the scene from the
> Object Library (you know how to get to this now! If you forgot, check back
> earlier in this tutorial).
>
> Set the position to `(0, 0)`, _Z-Position_ to `1` and _Name_ to
> `obstacleLayer`
>
> ![Creating the obstacle layer node](../Tutorial-Images/xcode_add_obstacle_layer.png)

<!-- -->

> [action]
>
> Make sure to set `obstacleLayer` to be the parent of the `obstacle` object!
>
> ![obstacle parent and child](../Tutorial-Images/xcode_obstacle_parent.png)

<!-- -->

> [info]
>
> It can sometimes be tricky to select nodes in the editor when they are
> on top of each other, an awesome tip that not everyone is aware of is to use
> the scene hierarchy selector:
>
> ![Selecting nodes](../Tutorial-Images/xcode_node_selection_tip.png)
>
> You can thank me later :]

## Connecting the obstacle layer

You'll need to code connect the obstacle layer object.

> [action]
>
> Open `GameScene.swift` and add the following property to the
> `GameScene` class:
>
```
var obstacleLayer: SKNode!
```
>
> Add the following code after the `scrollLayer` in the `didMove(to view:)` method. Create a connection:
>
```
/* Set reference to obstacle layer node */
obstacleLayer = self.childNode(withName: "obstacleLayer")
```

## Spawn Timer

You will add a timer property to help manage the rate of obstacle generation.
Each time period we will generate a new obstacle. The time you choose is
important to game play, too slow it's boring, too fast it's too hard.

> [action] In `GameScene.swift`, add the following code after the _sinceTouch_
> property declaration at the top of the `GameScene` class:
>
```
var spawnTimer: CFTimeInterval = 0
```

# Obstacle scrolling

You are going to create another conveyor belt solution for the newly added
`ObstacleLayer`.

> [action] In `GameScene.swift`, add the following method to the `GameScene` class:
>
```
func updateObstacles() {
   /* Update Obstacles */
>
   obstacleLayer.position.x -= scrollSpeed * CGFloat(fixedDelta)
>
   /* Loop through obstacle layer nodes */
   for obstacle in obstacleLayer.children as! [SKReferenceNode] {
>
       /* Get obstacle node position, convert node position to scene space */
       let obstaclePosition = obstacleLayer.convert(obstacle.position, to: self)
>
       /* Check if obstacle has left the scene */
       if obstaclePosition.x <= -26 {
       // 26 is one half the width of an obstacle
>
           /* Remove obstacle node from obstacle layer */
           obstacle.removeFromParent()
       }
>
   }
>
 }
```

This code should look familiar it's similar to the code used in `scrollWorld()`.
This time when an obstacle has left the scene, there is no longer any need for
it, so it will be removed by calling the `removeFromParent()` method.

Can you modify your game code to call the `updateObstacles()` method every
frame?

> [solution]
>
> As with the `scrollWorld()` method, in `GameScene.swift,` add the following to the `update(...)` method, after the `scrollWorld()` method.
>
```
/* Process obstacles */
updateObstacles()
```

Running the game now will not do much. The `obstacleSource` node is not attached
to the obstacleLayer. So it doesn't move. You are using this node as the source
from which you will create an endless supply of copies.

# Spawning endless randomized obstacles

The next task will be to continuously spawn obstacles by copying the source
obstacle you created in `GameScene.sks`.

To make the game interesting you will want each new obstacle appear at a
different height.

> [action]
> In `GameScene.swift`, add this code to the end of your
> `updateObstacles` method:
>
```
/* Time to add a new obstacle? */
if spawnTimer >= 1.5 {
>
    /* Create a new obstacle by copying the source obstacle */
    let newObstacle = obstacleSource.copy() as! SKNode
    obstacleLayer.addChild(newObstacle)
>
    /* Generate new obstacle position, start just outside screen and with a random y value */
    let randomPosition =  CGPoint(x: 347, y: CGFloat.random(in: 234...382))
>
    /* Convert new node position back to obstacle layer space */
    newObstacle.position = self.convert(randomPosition, to: obstacleLayer)
>
    // Reset spawn timer
    spawnTimer = 0
}
```

The code above creates a new obstacle instance every `1.5` seconds from
`Obstacle.sks`. So far `spawnTimer` has not been incremented, you will do that
in the next step.

We placed this new obstacle off the right side of the screen and set the **Y
Position** randomly between the **min** value of `234` and a **max** value of
`382` to mix things up a little.

When deciding these sorts of gameplay values, it's handy to go back to
`GameScene.sks` and check the **Y Position** of our obstacle. Move it up and
down and take note of the Y Position, then pick a range that looks good to you.

Once you are finished move the source obstacle outside of the scene on the right
side. This way it will not be visible. Players will only see the copies as they
are created.

> [info]
>
> You know that feeling when you play a great game and the core mechanic
> feels just right? This is rarely a coincidence, expect to adjust these values
> and tinker with mechanics, get friends to play and ask for feedback. Keep
> adjusting the values until the game play feels right and your play testers are
> having fun!

Although you've added a _spawnTimer_ property, it's not been setup to track
time. Can you think of how to do this?

> [solution]
>
> Add the following line to the end of your `update(...)` method in  `GameScene.swift`:
>
```
spawnTimer+=fixedDelta
```

Now run your game. You should see obstacles being generated every `1.5` seconds
and with varying vertical positions! You're getting closer to completing _Hoppy
Bunny_!

# Summary

The gaming is really coming together now, you've learnt to:

- Building a multi-object obstacle
- Dynamically generating obstacles in the scene
- Randomizing obstacle properties
- Removing objects from a scene when no longer required

In the next chapter you're going to implement physics based collision detection.
