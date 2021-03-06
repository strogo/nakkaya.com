---
title: Line Segment/Circle - Collision Detection
tags: clojure
---

Here is another piece of code I didn't want to throw away, it uses
[vector projection](http://en.wikipedia.org/wiki/Vector_projection) to
determine if a circle collides with a line
segment. The idea was to figure out if one NPC can pass the ball to
another in a way that no opposing team members can intercept it. We
represent each opposing team player as a circle, the size of the circle
will grow or shrink depending on the velocity of the player then we
check each circle for collision with the line segment if non collides
we assume it is a safe pass, following images shows a safe pass where
non of the three opposing players can intercept the pass,

![line segment collision](/images/post/line-segment-collision.png)


     (defn closest-point-on-line [a b c]
       (let [ac (subtract c a)
             ab (subtract b a)
             proj-mag (dot-product ac (normalize ab))]
         (cond (< proj-mag 0) a
               (> proj-mag (magnitude ab)) b
               :default (add (project ac ab) a))))

We begin by creating two new vectors, one from the start of the line to
end of the line (AB) and one from start of the line to the center of the
circle (AC), then we calculate the magnitude (length) of the projection
of AC onto AB, if it is smaller than 0 then the closest point on this
line to 
the circle is the point A (start of the line segment), if it is bigger
than the magnitude of the AB vector then the closest point is B, else we
return the projection of AC onto AB plus A (which converts it back into
world coordinates) that gives us the closest point on the line to the
circle.

     (defn path-clean [a b c r]
       (let [closest (closest-point-on-line a b c)
             distance (magnitude (subtract c closest))]
         (if (<= distance r)
           false true)))

Now that we know the location of the closest point on the line to the
circle, collision detection is as simple as calculating the length
between closest point and the circle, if it is smaller than the radius
of the circle we have a collision else we have a clean path.


 - [collision-detection.clj](/code/clojure/collision-detection.clj)
