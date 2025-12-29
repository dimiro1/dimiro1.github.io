+++
title = 'How we made buttons before CSS3'
date = 2026-01-02
description = ''
tags = ['CSS', 'HTML']
draft = true
+++

It's 2007, and your designer just handed you a design with a "beautiful button", back then, most likely a PSD from Photoshop or a Fireworks file. Looking at the file, you notice that the button has:

- A two-tone gradient with a hard edge in the middle
- Rounded corners
- A subtle box shadow at the bottom

Today, you'd write a few lines of CSS and call it a day. But back then? CSS couldn't do gradients (~2010). Couldn't do rounded corners (~2010). Couldn't do box shadows (~2009).

I started web development around 2007-2008 and learned these techniques in the trenches. This was the way. Let me show you how we did it.

## The button

This is what the designer handed you:

{{< html-demo title="Final Button" image="button.png" >}}
<style>
  body { margin: 0; padding: 16px; display: flex; justify-content: center; background: #e0e0e0; }
</style>
<img height="60" src="{{image}}" />
{{< /html-demo >}}

## 1st approach: Image as background

The simplest way to create such a button is to export the image from Photoshop and use it as a background of an HTML button.

Here's the image we'll use as background:

{{< html-demo title="The Image" image="button-bg.png" >}}
<style>
  body { margin: 0; padding: 16px; display: flex; justify-content: center;  background: #e0e0e0; }
</style>
<img height="60" src="{{image}}" />
{{< /html-demo >}}

And here's the result:

{{< html-demo title="Image Button" image="button-bg.png" >}}
<style>
  body { margin: 0; padding: 16px; display: flex; justify-content: center; background: #e0e0e0; }
  .btn {
    width: 200px;
    height: 60px;
    background: url('{{image}}') no-repeat center;
    background-size: contain;
    border: none;
    cursor: pointer;
	font-family: Arial, sans-serif;
    font-size: 16px;
    font-weight: bold;
    color: #333;
  }
</style>
<button class="btn">Download Now</button>
{{< /html-demo >}}

```html
<style>
  .btn {
    width: 200px;
    height: 60px;
    background: url('button.png') no-repeat center;
    background-size: contain;
    border: none;
    cursor: pointer;
    font-family: Arial, sans-serif;
    font-size: 16px;
    font-weight: bold;
    color: #333;
  }
</style>
<button class="btn">Download Now</button>
```

This approach works, but has some drawbacks:
- The button size is fixed to the image dimensions
- You need a new image for every button variation
- Each image is a separate HTTP request - a big problem in 2007 when connections were slow and [browsers limited concurrent downloads to 2 per domain](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Connection_management_in_HTTP_1.x)

## 2nd approach: Slicing the image

The fixed-size problem led to a clever technique: slice the button into three pieces:

1. **Left side** - full height image with rounded left corners
2. **Right side** - full height image with rounded right corners (mirrored from left in Photoshop)
3. **Middle stripe** - 1px wide, repeats horizontally to fill any width

{{< html-demo title="The 3 slices" left="left.png" right="right.png" content="button-content.png" >}}
<style>
  body { margin: 0; padding: 16px; display: flex; flex-direction: column; align-items: center; gap: 8px; background: #e0e0e0; }
  .slices { display: flex; gap: 20px; }
  .slice {
    height: 60px;
    background-size: contain;
    background-repeat: no-repeat;
  }
  .slice-left {
    width: 12px;
    background-image: url('{{left}}');
    background-size: 12px 60px;
  }
  .slice-content {
    width: 1px;
    background-image: url('{{content}}');
    background-repeat: no-repeat;
    background-size: 1px 60px;
  }
  .slice-right {
    width: 12px;
    background-image: url('{{right}}');
    background-size: 12px 60px;
  }
  .labels { display: flex; gap: 20px; font: 12px Arial; color: #666; }
</style>
<div class="slices">
  <div class="slice slice-left"></div>
  <div class="slice slice-content"></div>
  <div class="slice slice-right"></div>
</div>
<div class="labels"><span>left</span><span>middle</span><span>right</span></div>
{{< /html-demo >}}

We'd use absolutely positioned spans for the edges:

{{< html-demo title="Sliced Button" left="left.png" right="right.png" content="button-content.png" >}}
<style>
  body { margin: 0; padding: 16px; display: flex; justify-content: center; background: #e0e0e0; }
  .btn {
    position: relative;
    display: inline-block;
    height: 60px;
    background: url('{{content}}') repeat-x left center;
    background-size: auto 60px;
    padding: 0 40px;
    border: none;
    cursor: pointer;
    font-family: Arial, sans-serif;
    font-size: 16px;
    font-weight: bold;
    color: #333;
    line-height: 60px;
  }
  .btn .left {
    position: absolute;
    top: 0;
    left: 0;
    width: 12px;
    height: 60px;
    background: #e0e0e0 url('{{left}}') no-repeat left top;
    background-size: 12px 60px;
  }
  .btn .right {
    position: absolute;
    top: 0;
    right: 0;
    width: 12px;
    height: 60px;
    background: #e0e0e0 url('{{right}}') no-repeat right top;
    background-size: 12px 60px;
  }
</style>
<button class="btn">
  <span class="left"></span>
  <span class="right"></span>
  Download Now
</button>
{{< /html-demo >}}

```html
<style>
  .btn {
    position: relative;
    height: 60px;
    background: url('middle.png') repeat-x;
    padding: 0 40px;
    border: none;
    line-height: 60px;
  }
  .btn .left {
    position: absolute;
    top: 0;
    left: 0;
    width: 12px;
    height: 60px;
    background: url('left.png') no-repeat;
  }
  .btn .right {
    position: absolute;
    top: 0;
    right: 0;
    width: 12px;
    height: 60px;
    background: url('right.png') no-repeat;
  }
</style>
<button class="btn">
  <span class="left"></span>
  <span class="right"></span>
  Download Now
</button>
```

Now the button can grow to fit any text! But we still have problems:
- Extra span elements for the edges
- Multiple HTTP requests for images
- Need new images for different colors

## 3rd approach: CSS Sprites

The slicing technique works, but each image is a separate HTTP request. In 2007, [browsers limited concurrent connections to just 2 per domain](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Connection_management_in_HTTP_1.x). Three images? Three round trips to the server. There were tricks to work around this limit - maybe a topic for another article.

The solution? Combine all images into a single sprite and use `background-position` to show just the piece you need:

{{< html-demo title="The Sprite" sprite="button-sprite.png" >}}
<style>
  body { margin: 0; padding: 16px; display: flex; justify-content: center; align-items: center; gap: 40px; background: #e0e0e0; }
  .full-sprite {
    width: 12px;
    height: 180px;
    background: url('{{sprite}}') no-repeat;
    background-size: 12px 180px;
    border: 1px dashed #999;
  }
  .breakdown { display: flex; flex-direction: column; align-items: center; gap: 8px; }
  .slices { display: flex; gap: 20px; }
  .sprite {
    width: 12px;
    height: 60px;
    background: url('{{sprite}}') no-repeat;
    background-size: 12px 180px;
  }
  .sprite-left { background-position: 0 0; }
  .sprite-middle { background-position: 0 -60px; }
  .sprite-right { background-position: 0 -120px; }
  .labels { display: flex; gap: 20px; font: 12px Arial; color: #666; }
  .arrow { font-size: 24px; color: #999; }
</style>
<div class="full-sprite"></div>
<div class="arrow">→</div>
<div class="breakdown">
  <div class="slices">
    <div class="sprite sprite-left"></div>
    <div class="sprite sprite-middle"></div>
    <div class="sprite sprite-right"></div>
  </div>
  <div class="labels"><span>left</span><span>middle</span><span>right</span></div>
</div>
{{< /html-demo >}}

Same image, different `background-position` values:

{{< html-demo title="Sprite Button" sprite="button-sprite.png" >}}
<style>
  body { margin: 0; padding: 16px; display: flex; justify-content: center; background: #e0e0e0; }
  .btn {
    position: relative;
    display: inline-block;
    height: 60px;
    background: url('{{sprite}}') repeat-x 0 -60px;
    background-size: 12px 180px;
    padding: 0 40px;
    border: none;
    cursor: pointer;
    font-family: Arial, sans-serif;
    font-size: 16px;
    font-weight: bold;
    color: #333;
    line-height: 60px;
  }
  .btn .left {
    position: absolute;
    top: 0;
    left: 0;
    width: 12px;
    height: 60px;
    background: #e0e0e0 url('{{sprite}}') no-repeat 0 0;
    background-size: 12px 180px;
  }
  .btn .right {
    position: absolute;
    top: 0;
    right: 0;
    width: 12px;
    height: 60px;
    background: #e0e0e0 url('{{sprite}}') no-repeat 0 -120px;
    background-size: 12px 180px;
  }
</style>
<button class="btn">
  <span class="left"></span>
  <span class="right"></span>
  Download Now
</button>
{{< /html-demo >}}

```html
<style>
  .btn {
    position: relative;
    height: 60px;
    /* Middle row of sprite, repeats horizontally */
    background: url('sprite.png') repeat-x 0 -60px;
    padding: 0 40px;
    border: none;
    line-height: 60px;
  }
  .btn .left {
    position: absolute;
    top: 0;
    left: 0;
    width: 12px;
    height: 60px;
    /* Top row of sprite */
    background: url('sprite.png') no-repeat 0 0;
  }
  .btn .right {
    position: absolute;
    top: 0;
    right: 0;
    width: 12px;
    height: 60px;
    /* Bottom row of sprite */
    background: url('sprite.png') no-repeat 0 -120px;
  }
</style>
<button class="btn">
  <span class="left"></span>
  <span class="right"></span>
  Download Now
</button>
```

One image, one HTTP request. The browser downloads the sprite once, caches it, and every button on the site loads instantly.

Back in the day, some sites had massive sprites with all their buttons, icons, and UI elements in a single file.

## Button states

What about hover and active states? Back to Photoshop. For each state, you'd create new images, slice them, add them to the sprite, and write more CSS rules. A simple hover effect could mean hours of work.

Over time, tools appeared to automate sprite creation - some even output JSON with all the positions.

## And today?

All that complexity? Gone. Modern CSS can do it in a few lines:

{{< html-demo title="CSS3 Button" >}}
<style>
  body { margin: 0; padding: 16px; display: flex; justify-content: center; background: #e0e0e0; }
  .btn {
    font-family: Arial, sans-serif;
    font-size: 16px;
    font-weight: bold;
    color: #333;
    padding: 18px 40px;
    border: none;
    border-radius: 12px;
    background: linear-gradient(to bottom, #FFD54F 0%, #FFD54F 50%, #F9A825 50%, #F9A825 100%);
    box-shadow: 0 3px 0 rgba(120, 80, 0, 0.35);
    cursor: pointer;
  }
  .btn:hover {
    background: linear-gradient(to bottom, #FFE082 0%, #FFE082 50%, #FFC107 50%, #FFC107 100%);
  }
  .btn:active {
    box-shadow: 0 2px 0 rgba(120, 80, 0, 0.35);
    transform: translateY(2px);
  }
</style>
<button class="btn">Download Now</button>
{{< /html-demo >}}

```html
<style>
.btn {
  padding: 18px 40px;
  border: none;
  border-radius: 12px;
  background: linear-gradient(to bottom,
    #FFD54F 0%, #FFD54F 50%,
    #F9A825 50%, #F9A825 100%);
  box-shadow: 0 3px 0 rgba(120, 80, 0, 0.35);
}
</style>

<button class="btn">Download Now</button>
```

No images. No extra markup. Just CSS.

---

It's incredible how far we've come, nowadays css can do animations, grids, custom properties, container queries and much more.

I really like the web as a platform, it keeps improving, you don't need anyone's permission to create something, no app store approval, no gatekeepers. Open your editor, open a browser, and start building.

Let me know what you think.
