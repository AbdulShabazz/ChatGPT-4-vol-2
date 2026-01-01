## OpenAI ChatGPT v1.2025.350 Black.Circle.Fade.GIMP.md

**Q: In GIMP, how do I create a black circle that fades into transparency?**

**A:** Use a radial gradient on a circular selection, applied to the layer’s alpha channel.

Steps (GIMP 2.10+)

1. Create a transparent layer

* File → New (or open image).
* Layer → New Layer… → Fill with: Transparency.

2. Make a perfect circle selection

* Select → Ellipse Select Tool.
* Tool Options: enable Fixed: Aspect ratio = 1:1.
* Hold Shift while dragging (and Ctrl to draw from center, if desired).
* Position/size it as needed.

3. Fill with a black→transparent radial gradient

* Set Foreground color to black.
* Set Background color to transparent:

  * In the color picker, set Alpha (A) to 0, or
  * Use the “transparent” swatch if visible.
* Select the Blend Tool (Gradient).
* Tool Options:

  * Gradient: FG to BG (RGB) (or “FG to Transparent” if available)
  * Shape: Radial
* Click-drag from the center of the circle outward to the edge.

  * Shorter drag = faster fade.
  * Longer drag = smoother fade.

4. Remove the selection

* Select → None.

Common variants

* Hard black center, soft edge only:
  Do the gradient so black is at the center and transparent at the edge (as above). If you want a stronger center, duplicate the layer or run Colors → Levels/Curves.

* Only the ring/edge fades (hollow center):
  After step 2, Select → Border… (set thickness), then apply the same radial gradient.

* Non-destructive method (recommended):
  Instead of filling the layer directly, fill the circle with solid black first, then:

  * Layer → Mask → Add Layer Mask… → “White (full opacity)”
  * Apply the radial gradient to the mask using black→white (black = transparent, white = opaque).
