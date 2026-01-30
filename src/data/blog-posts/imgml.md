---
title: I Turned Images Into 10,000 Tiny HTML Elements Because Why Not
slug: imgml
publishDate: 29 Jan 2026
description: _Sometimes the best projects are the ones that solve problems nobody has.
---

<!-- @format -->

_Sometimes the best projects are the ones that solve problems nobody has._

---

Last weekend, I found myself staring at my screen, thinking about images. Not in a productive way. Not "how can I optimize image loading" or "what's the best format for web." No, I was thinking something far more unhinged:

**What if every pixel was just a tiny HTML element?**

And thus, IMGML was born.

## The Premise

Here's the concept: take an image, read it pixel by pixel, and generate an HTML file where each pixel is represented by a 1x1 `<hr>` element with a background color matching the original pixel.

That's it. That's the whole thing.

No `<img>` tags. No `<canvas>`. No SVG. Just thousands upon thousands of microscopic horizontal rules, arranged in a grid, pretending to be an image.

## Does It Work?

Technically? Yes.

Practically? That's a strong word.

Upload a 100x100 pixel image and you get 10,000 HTML elements. A 200x200 image? 40,000 elements. Your browser will render it. Your browser might also ask you why you've done this.

But here's the thing — it _does_ look like the image. Open the HTML file and there it is, your photo, reconstructed entirely from styled `<hr>` tags. It's beautiful in the way that a Rube Goldberg machine is beautiful. Unnecessarily complex, deeply impractical, and yet somehow satisfying.

## Why Though?

I get asked this a lot. "Why would you do this?" "What's the use case?" "Are you okay?"

The answers are: because I could, there isn't one, and debatable.

This is a "what if" project. It exists in that space between curiosity and questionable judgment. The space where you wonder "can I do this?" before asking "should I do this?"

Some might call it a waste of time. I call it a learning experience wrapped in absurdity.

## The Tech

For those curious, it's built with Laravel 12 and Livewire. Upload an image, the backend processes it pixel by pixel, and you download your freshly minted HTML monstrosity.

Here's what the output looks like — just a wall of `<hr>` tags, each with an inline background color:

```html
<r>
  <hr style=background:#cad5d9/>
  <hr style=background:#c5ced3/>
  <hr style=background:#cdd6db/>
  <hr style=background:#dde4ea/>
  <hr style=background:#e4ebf1/>
  <hr style=background:#e1e6ec/>
  <hr style=background:#dadfe5/>
  <hr style=background:#d7dadf/>
  ...
</r>
```
The `<r>` tag represents a row of pixels. Why `<r>` and `<hr>`? File size. When you're generating tens of thousands of elements, every character counts. `<hr>` is the shortest self-closing tag in HTML, and `<r>` is about as minimal as a row container can get. It's not semantic, but neither is this entire project.
## The Optimization Rabbit Hole
Once I committed to this absurd idea, I became obsessed with making each pixel as small as possible. Here's how that went:
**Attempt 1: RGB**
```html
<hr style="background:rgb(255, 255, 255)">
```
Works, but verbose. All those characters add up.
**Attempt 2: Hex**
```html
<hr style="background:#c5ced3"/>
```
Shorter. Hex codes are more compact than RGB.
**Attempt 3: Drop the self-closing slash**
```html
<hr style="background:#c5ced3">
```
It's optional in HTML5. Every byte counts.
**Attempt 4: Drop the quotes**
```html
<hr style=background:#c5ced3>
```
Also valid HTML. We're getting minimal.
**Attempt 5: The bgcolor attribute**
```html
<hr bgcolor="#c5ced3">
```
Deprecated, and for some reason only works on `<body>`. Dead end.
**Attempt 6: The color attribute**
```html
<hr color=#c5ced3>
```
This works. It's deprecated, it's not recommended, and it's perfect for this project.
**Attempt 7: Drop the hashtag**
```html
<hr color=c5ced3>
```
Turns out the `#` is optional on hex colors. One more character gone.
This is the shortest I could make each pixel. If you know a way to make it smaller, I'm genuinely curious.

The code is straightforward. PHP's image functions handle the pixel reading:

```php
// Resize to keep things manageable
$image = $this->resizeImage($path, 200, 200);
$resource = imagecreatefromjpeg($image);

$width = imagesx($resource);
$height = imagesy($resource);
$pixels = [];

// Loop through every pixel
for ($y = 0; $y < $height; $y++) {
    for ($x = 0; $x < $width; $x++) {
        // Get color index at this coordinate
        $color = imagecolorat($resource, $x, $y);

        // Convert to RGB array
        $rgb = imagecolorsforindex($resource, $color);

        // Format as hex (no hashtag needed)
        $hex = sprintf('%02x%02x%02x', $rgb['red'], $rgb['green'], $rgb['blue']);

        $pixels[] = $hex;
    }
}
```

Loop through each row, each column, grab the color, convert to hex, done. Livewire handles the upload/download flow. Nothing revolutionary — just reasonable tools applied to an unreasonable idea.

## What I Actually Learned

Despite the silliness, building IMGML reminded me of a few things:

1. **Constraints breed creativity.** Limiting myself to just HTML elements forced me to think differently about rendering.

2. **Performance matters everywhere.** Processing images pixel by pixel taught me to respect the work that actual image libraries do.

3. **Side projects don't need purpose.** Not everything has to ship, scale, or solve a real problem. Sometimes you just build something weird and smile.

## Should You Use This?

Absolutely not.

Well, maybe. If you need to:

- Confuse a web developer
- Create the world's least efficient image format
- Prove a point about HTML being turing complete (it's not, but this feels adjacent)
- Kill time on a Saturday

Then yes, IMGML is for you.

## Try It Yourself

The project is open source. Clone it, run it, upload an image, and watch your browser question its life choices as it renders your photo one `<hr>` at a time.

Will it change the world? No.

Will it make you briefly happy in a "wow, that's dumb" kind of way? Almost certainly.

And sometimes, that's enough.

Here's [my GitHub avatar in IMGML format](/assets/blog/imgml.html) if you want to see it in action.

> **Note:**
>
> ```html
> <hr color="c5ced3" />
> ```
>
> <small>_Does not work on Firefox. This example uses the following structure:_</small>
>
> ```html
> <hr style="background:#07040b" />
> ```

---

_IMGML is available on [GitHub](https://github.com/brunoabpinto/IMGML). Star it if you appreciate the absurdity._
