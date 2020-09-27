+++
title = "Js13k 2020 post-mortem 1/3"
date = 2020-09-21
tags = []
+++

# Js13k 2020 post-mortem 1/3

A quick reminder of what the js13k is:

> [Js13kGames](https://js13kgames.com/) is a JavaScript coding competition for HTML5 Game Developers. The fun part of the compo is the file size limit set to 13 kilobytes. The competition started at 13:00 CEST, 13th August and ended at 13:00 CEST, 13th September 2020. Theme for this year was 404.

This year, unlike 2019 where I worked on my own, I brought some friends to the competition! There were 4 of us working on the game and we managed to submit 404 BC Pinball:

[![404 BC Pinball](https://js13kgames.com/games/404-bc-pinball/__big.jpg)](https://js13kgames.com/entries/404-bc-pinball)

In this post-mortem, I will highlight a few technical challenges that we had to overcome during the jam:
+ **Physics Engine**
+ **SVGs**
+ **Compression (this one *hurt* a lot)**

Last year I didn't have to worry about the size of the game because I was working on my own and exclusively wrote JavaScript. I didn't have much experience in game development or JavaScript, so I couldn't write that many lines in a month of competition.

This year, our game 404 BC Pinball is a whole different beast. Every elements of the pinball are rasterised SVGs, the physics engine is an implementation of the Separating Axis Theorem and the musics are made with ZzFx.
This is a lot more content to try to fit in 13kB.

Interested how we did it? Let's dig into the code!

# Physics engine

Disclaimer: I had no experience whatsoever on physics engine and decided to build one from scratch. I could have recycled an engine from past year's entries, but where's the fun in that?

## Resources

I mostly used this tutorial, which is well written:

[Dyn4j's tutorial](http://www.dyn4j.org/2010/01/sat/)

## Pinball physics

Pinball physics are quite easy to understand (as long as you've seen one!). It mostly consists in:
+ 1 ball, falling under gravity
+ A few static elements, which can collide/can't collide with the ball 
+ 2 flippers
+ 1 plunger (launches the ball)

## Collision detection

I quickly figured out that only having box colliders would be incredibly boring and just wouldn't work for the ball. So I decided to implement the separating axis theorem: it works very well and there's many different examples on the web in different programming languages.

I won't explain all the details of the theorem as there's already plenty of good resources on the subject. I'll show you the final result in JavaScript. Only works for convex shapes!

Here's the shortest implementation I came by:

```typescript
export class Shape {
    vertices: Vector[] = [];

    // Vector only holds 2 properties, x and y.
    constructor(vertices: Vector[]) {
        this.vertices = vertices;
    }

    // Projects a shape (each vertex) to a vector, returning a segment (min-max).
    project(vector: Vector) {
        return this.vertices.reduce(
            (minMax, vertice) => {
                const value = vertice.dot(vector);
                if (value < minMax.min)
                    minMax.min = value;
                if (value > minMax.max)
                    minMax.max = value;
                return minMax;
            },
            { min: this.vertices[0].dot(vector), max: this.vertices[0].dot(vector) }
        );
    }
}

export class CollisionDetector {
    // Returns the normal vector of each edges
    getSeparatingAxes(shape: Shape) {
        let axes = [];
        for (let i = 0; i < shape.vertices.length; i++) {
            const v1 = shape.vertices[i];
            const v2 = shape.vertices[(i + 1) % (shape.vertices.length)];
            const edge = v2.subtract(v1).normalize();
            axes[i] = edge.perp();
        }
        return axes;
    }

    // Get the relative overlap of 2 segments. Returns null if no overlap.
    getOverlap(projection1: { min: number, max: number }, projection2: { min: number, max: number }) {
        // No overlap
        if (projection1.max < projection2.min || projection2.max < projection1.min) return null;
        const min = Math.max(projection1.min, projection2.min);
        const max = Math.min(projection1.max, projection2.max);
        // Sign == 1 if projection1 is left, -1 otherwise. Check for 0 if mins are equal!
        let sign = Math.sign(projection1.min - projection2.min);
        if (sign == 0) sign = Math.sign(projection1.max - projection2.max);
        return sign * (max - min);
    }

    // Checking collision is quite simple:
    // Check minimum overlap of both projected shapes, against all the separating axes.
    // If one axe returns no overlap, shapes do not overlap.
    satCollide(shape1: Shape, shape2: Shape) {
        // There's room for optimization here, as we might check equivalent axes.
        // For a rectangle, checking 2 axes is enough (the other 2 are equivalent).
        const axes = this.getSeparatingAxes(shape1).concat(this.getSeparatingAxes(shape2));
        let minVector = null;
        let minOverlap = null;
        for (let axe of axes) {
            const overlap = this.getOverlap(shape1.project(axe), shape2.project(axe));
            if (overlap != null) {
                if (minOverlap == null || Math.abs(overlap) < Math.abs(minOverlap)) {
                    minOverlap = overlap;
                    minVector = axe;
                }
            } else {
                return null;
            }
        }
        // Return the minimum translation vector. Applying this vector to one shape will stop theme from colliding.
        return minVector.multiply(minOverlap);
    }
}
```

I didn't have much trouble to get this working for a variety of different shapes (triangle, rectangles, general polygons).

**But it cannot handle circles.**

Well, the good news is that I can approximate circles as a 24 corners polygon. And it works. It's certainly not perfect but I found it to be pretty good for an approximation. Plus, there's no code to add!

## Collision response

Oof - here's the part I struggled with. Handling collision between two different objects can be done in multiple ways. Each and every one of them has some good and bad properties, there's no silver bullets here. I'll walk you through mine which worked fine for 404 BC Pinball.

I used an impulse-based resolution and tried to implement it using this [tutorial](https://gamedevelopment.tutsplus.com/tutorials/how-to-create-a-custom-2d-physics-engine-the-basics-and-impulse-resolution--gamedev-6331). I believe it to be correct but the complexity of the solution is overwhelming to me. The physics behind aren't hard to understand, but the resulting code seemed too complex for what I wanted to do *(heh, I was going to throw all the code away in a month, so why bother)*.

Instead of applying an impulse only, I translate one shape away using the minimum translation vector from collision detector and apply an impulse. No sinking object, no positional correction. Only one translation and two forces to apply. Pretty easy, no?

Well, it's only 9 LOC:

```typescript
export class CollisionResolver {
    // Resolve collision on two bodies (= shapes with speed, position and mass).
    resolve(body1: Body, body2: Body, minimumTranslationVector: Vector) {
        const normal = minimumTranslationVector.normalize();
        // Bounciness is a property of the body, tweak it to your liking.
        const reaction1 = normal.dot(body1.speed.multiply(body1.bounciness * body2.bounciness));
        const reaction2 = normal.dot(body2.speed.multiply(body1.bounciness * body2.bounciness));
        const reaction = reaction1 - reaction2;
        if (reaction < 0) {
            // Apply impulses to bodies.
            body1.applyImpulse(normal.multiply(-reaction));
            body2.applyImpulse(normal.multiply(reaction));
            // Immediately resolve overlap by translating body1 away from body2.
            body1.translate(minimumTranslationVector);
        }
    }
}
```

That's most of the game's physics.

**But bodies have no momentum!**

That certainly is something I should have implemented properly. However, there's only **one** object which needs it, the flipper. Technically there are 2 flippers but the problem is exactly the same. I decided to not implement anything in the physics engine itself.

### Flippers physics

I won't get into too much details in this part because the code is atrocious and not really important. The idea behind it is! You can read [flipper.ts](https://github.com/Harmades/js13k-2020/blob/master/src/world/flipper.ts) if you want to.

The idea is to change the speed of the flipper (even though it's not moving) to give the ball a kick if it collides with the rotating flipper. The flipper speed depends on the maximum angular speed (the speed at which the flipper rotates) and the distance to the ball. This way, it simulates a **lever**. It required some tweaking on the different parameters, but if you're falling short on time like me, be sure to give it a try.

# Conclusion

When I look back to this implementation, compared to last year's physics engine, I think it turned out pretty well! I was able to use some shortcuts here and there to save some time and complexity while maintaining some consistency.
Here are some points that could be improved:
+ Broad phase collision detector
+ Handle real circle-polygon collision
+ Implement momentum for a general-purpose physics engine

That's the end of part 1. ~~Be sure to check out part 2 on SVGs when it's ready!~~ [It is!]({{< ref "/posts/js13k-2020-post-mortem-2" >}})