# Canvas2D Settings

## The problem

The first problem is that there is significant redundant spec text between `CanvasRenderingContext2D` and `OffscreenCanvasRenderingContext2D`.

The second, more serious, problem is that, as with all things that are duplicate, they diverge unintentionally. And `CanvasRenderingContext2D` and `OffscreenCanvasRenderingContext2D` have indeed diverged.

### The redundant spec text

There is redundant spec content between 
onscreen [output bitmap](https://html.spec.whatwg.org/multipage/canvas.html#output-bitmap) and
offscreen [bitmap](https://html.spec.whatwg.org/multipage/canvas.html#offscreencontext2d-bitmap).

Same for
onscreen [alpha](https://html.spec.whatwg.org/multipage/canvas.html#concept-canvas-alpha) and
offscreen [alpha](https://html.spec.whatwg.org/multipage/canvas.html#offscreencontext2d-alpha).

And also
onscreen [color space](https://html.spec.whatwg.org/multipage/canvas.html#concept-canvas-color-space) and
offscreen [color space](https://html.spec.whatwg.org/multipage/canvas.html#offscreencontext2d-color-space).

### Divergence in the spec.

All of the accidental divergence that we will address are features added to `CanvasRenderingContext2D` but unintentionally not added to `OffscreenCanvasRenderingContext2D`.

#### `willReadFrequently`

`CanvasRenderingContext2D` has a `willReadFrequently` property, but `OffscreenCanvasRenderingContext2D` does not.
This was added [in this issue](https://github.com/whatwg/html/issues/5614).
The authors of those changes have confirmed that the changes were intended to apply to both `CanvasRenderingContext2D` and `OffscreenCanvasRenderingContext2D` and that `OffscreenCanvasRenderingContext2D` was an intended use case.
The property is implemented for `OffscreenCanvasRenderingContext2D` in Chromium (oops).

#### `desynchronized`

`CanvasRenderingContext2D` has a `desynchronized` property, but `OffscreenCanvasRenderingContext2D` does not.

This was added for `CanvasRenderingContext2D` in [this issue](https://github.com/whatwg/html/issues/4087).
This was also added for WebGL (referencing the same issue), where it is supported for both offscreen and onscreen.

The issue was originally proposed for `OffscreenCanvasRenderingContext2D` in [this issue](https://github.com/whatwg/html/issues/2659), but the issue languished forever.

#### `getContextAttributes`

`CanvasRenderingContext2D` has a `getContextAttrributes` method, but `OffscreenCanvasRenderingContext2D` does not.

This function has an odd history where the function was accidentally added, then when it was discovered and planned to be removed, a Blink "Intent to Remove" caused people to realize that maybe we need it.

It was committed [here](https://github.com/whatwg/html/commit/206873adc7d6862545b56097db874d175e81d15a).
It appears that not including this in `OffscreenCanvasRenderingContext2D` was an oversight.


## The fix

The proposed fix is to add a new interface
```idl
  interface mixin CanvasBitmap {
    CanvasRenderingContext2DSettings getContextAttributes();
  }
```

Include this in both `CanvasRenderingContext2D` and `OffscreenCanvasRenderingContext2D`.
```
  CanvasRenderingContext2D includes CanvasBitmap;
  OffscreenCanvasRenderingContext2D includes CanvasBitmap;
```

Then add a section for "the canvas settings" (similar to the existing ["the canvas state"](https://html.spec.whatwg.org/multipage/canvas.html#the-canvas-state)) which includes the text from `CanvasRenderingContext2D`'s
* [output bitmap](https://html.spec.whatwg.org/multipage/canvas.html#output-bitmap)
* origin-clean
* [alpha](https://html.spec.whatwg.org/multipage/canvas.html#concept-canvas-alpha)
* [desynchronized](https://html.spec.whatwg.org/multipage/canvas.html#concept-canvas-desynchronized)
* [will read frequently](https://html.spec.whatwg.org/multipage/canvas.html#concept-canvas-will-read-frequently)
* [color space](https://html.spec.whatwg.org/multipage/canvas.html#concept-canvas-color-space)
* [`getContextAttributes`](https://html.spec.whatwg.org/multipage/canvas.html#dom-context-2d-canvas-getcontextattributes)

This will implicitly add `willReadFrequently`, `desynchronized`, and `getContextAttributes` to `OffscreenCanvasRenderingContext2D`, which were supposed to be there anyway, and will prevent this sort of thing from happening again.

## Issues

The name `CanvasBitmap` does not impact future compatibility.

I originally had it be `CanvasSettings`, but that made the spec less clear.
All of the various settings have something to do with setting up the bitmap or displaying it.

