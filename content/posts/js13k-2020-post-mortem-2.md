+++
title = "Js13k 2020 post-mortem 2/3"
date = 2020-09-27
tags = []
+++

# Js13k 2020 post-mortem 2/3

This article is the second part of a three part series of the js13k post-mortem. This time, I'll show you how we've integrated visual assets into the game! Thanks to the experience I had from last year's jam, I quickly turned into SVGs for all visual assets, for multiple reasons:

+ SVGs are lightweight
+ SVGs compress very well
+ SVGs can be displayed in the browser natively
+ There are plenty of great SVG editors (Inkscape, Illustrator)
+ Bonus: we used SVGs draw all elements' colliders!

They do require a bit of work if you're going for performance, but it's not a big deal.

# Rendering

Browsers can display SVGs natively, but moving SVG objects around can be cumbersome and quite lengthy (code-wise), so I quickly figured out it won't be an optimal solution for the jam.

> Note: I haven't tried it and it would definitely be better to benchmark this solution.

I knew an HTML canvas would work perfectly for rendering, as long as I could transform SVGs into a standard image. It turns out to be quite easy:

```typescript
loadSvg(svg: SVGElement, onload: () => void) {
    const b64 = `data:image/svg+xml;base64,${btoa(svg.outerHTML)}`;
    const image = new Image();
    image.onload = () => {
        this.atlas = image;
        onload();
    };
    image.src = b64;
}
```

A 7 LOC method to load SVGs into an Image element is great and enabled us to work with Inkscape for our Pinball.

Great! Now we're able to transform SVGs into Image elements. However, it is more efficient to load a single Image into memory, and selecting the parts you need at render time. This single Image is called an atlas.

**So how do I form the atlas from a single SVG containing all the elements ?**

Here's the good part: SVG elements have a *very* convenient method called ```getBBox()```, which, well, returns the bounding box of a ```SVGSVGElement``` (no typo here, they really named it like this). I use the ids of specific SVG elements I was interested in, select them using ```getElementById(id)``` and voilà, you've got the position, width and height of your SVG element. You can then use it to render the element:

```javascript
render(context: CanvasRenderingContext2D) {
    context.drawImage(
        Assets.atlas, // Assets.atlas is the rasterised image of every assets in the game
        this.spriteBounds.x, // Spritebounds is the rectangle returned by .getBBox()
        this.spriteBounds.y,
        this.spriteBounds.width,
        this.spriteBounds.height,
        this.body.pos.x, // Draw it where you like
        this.body.pos.y,
        this.spriteBounds.width,
        this.spriteBounds.height 
    );
}
```

Using this method, no need to hard-code every assets' position in the atlas.

> Warning: do NOT leave transform(...) attributes in you SVG, or getBBox returns some weird results. I didn't have enough time to figure it out, so I removed the transform attributes by ungrouping an element in Inkscape (CTRL + SHIFT + G), and re-grouping all parts together (CTRL + G).

# Bonus: collider shape in SVG!

**What?**

Yes, I did draw most colliders in Inkscape too, using a separate layer. Because some shapes can be complex (but always a convex polygons), drawing them on Inkscape and using them in JavaScript is convenient.

There's one problem: I had to translate SVG path data to an array of points, and there aren't any APIs for that.

Here's an example of an SVG path (polygon with 4 edges):

```xml
<path id="$gutter.collider" d="m1027 10v153h218v-49z"/>
```

and the corresponding collider:

![Gutter collider](/collider.png)

With my friend David Sebaoun, we wrote a small parser for path data which translates the "d" attribute in a list of point (works on optimized SVG path!):

[Wonderful piece of code](https://github.com/Harmades/js13k-2020/blob/fd194095b1c27436d59f9bbc1695fd9ab5045702/src/math/shape.ts#L62)

It's been quite hard to write, and David wrote the complex part so I won't get into too much details. Hopefully you can re-use the code as-is or find some inspiration ;).

# Conclusion

I think using SVGs for js13k is a good choice because you can use the editor of your choice and style integrate your assets into the game without too much hassle. Florent Perez, who did most of the assets in Inkscape, has been able to stay away from JavaScript and focus on the game's visuals while we took care of the rendering. For this specific reason, using SVGs has been a successful choice.