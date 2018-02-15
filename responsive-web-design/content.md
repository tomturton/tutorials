## What is Responsive Web Design?

Responsive Web Design is an approach for designing website and web-apps so they scale well on all devices.

Designing one site for phones, tablets and desktop browsers is all very well, but of course devices come in all shapes and sizes and so it’s difficult to make assumptions about any particular device type, or plan for the future.

RWD teaches us not to be limited by specific platforms.
It is less restricting to design for different capabilities instead.
These capabilities include *viewport size* and *orientation*, *connectivity*, and *input methods*.

The term was coined by [Ethan Marcotte](http://ethanmarcotte.com/) in 2010.


## What it isn’t
RWD practices mean that the content will perfectly fit any window size.
Therefore layouts need to be fluid (flexible), shrinking and growing with the window if it changes size.

In contrast, *Adaptive Web Design* makes use of predetermined fixed (static) layouts for any set of screen sizes.

While AWD is an acceptable method of designing for different window sizes, RWD has the advantage of being able to use all available space.
RWD interfaces are not designed for any particular window size, so there isn’t any ideal point where the viewport is just the right size to best display the content.


## RWD Fundamentals
### Fluid layouts
The first rule of RWD is to create content that always perfectly fits the display.
To do this we need fluid layouts.
Whereas traditionally page content has been designed in columns of absolute widths (usually pixels), fluid layouts primarily use percentages to calculate the width of columns and other layout features.

This does not mean all parts of a page layout need to scale relatively.
It is often useful to have a fixed width panel column as part of a fluid layout.
On smaller displays though, it may be necessary to shrink any fixed width elements or even move or remove them altogether.

One thing to consider when designing a fluid layout, is how paragraph text is affected.
Usually it is a good idea to have a comfortable measure of about 45 to 75 characters.
In fixed width panels this won’t be a problem, but for elastic columns you might want to fix the width of the paragraph or dynamically increase the font-size as the page gets wider.

Viewport units are a great way of defining widths for fluid layouts.
One `vw` unit is the equivalent of 1% of the overall viewport width.
Browser support is fairly good, starting from IE9, but some useful viewport units (such as `vmax`) [aren’t as widely available yet](http://caniuse.com/viewport-units). Also IE9 does not support sizing text with viewport units.


### Flexible images
Often images need to fit neatly across a column or the entire page.
Inline HTML images can be resized dynamically with CSS by setting the `max-width` property to `100%` and `height` to `auto`.

However, this approach usually relies on the browser requesting a high-resolution image, which is then scaled.
This is inefficient and inconsiderate when serving images to smaller displays, especially when connectivity is limited or bandwidth is restricted or costly.

The relatively new `srcset` and `sizes` attributes of the `<img>` element let you specify which images to serve based on viewport size and other conditions. The `<picture>` element is also useful for art-direction. For older browers, use the [Picture Fill](https://github.com/scottjehl/picturefill) polyfill.

It might be wise to use SVG images wherever possible. SVG is the web’s native vector format, so it’s naturally responsive and will always be better quality than raster images. Of course, being a vector format, SVG is only useful for graphics, not photos.

Meanwhile, CSS background images can be resized with the CSS3 `background-size` property, and when combined with *media queries* (see below) they can be served responsibly.



### Breakpoints
A site can be made responsive with just fluid grids and flexible images, but it can often help to style page content differently on different-sized displays.
We can do this without JavaScript on all modern browsers (IE9+) using *CSS3 Media Queries*.

Media queries expand the use of the CSS `@media` rule to include viewport dimensions, device orientation and even supported colour modes.

This media query, for example, assigns rules to devices that have a viewport width of at least 480 pixels:


~~~~ css
@media only screen and (min-width: 480px) {
  body {
    background-color: red;
    font-size: 1.2rem;
  }

  .logo {
    display: inline-block;
  }
}
~~~~

You can see how powerful this feature can be. You can style your webpage completely differently for one screen size than another. It is useful for formatting navigation menus differently, separating content into columns, altering fonts, showing more detailed information and a hundred other things.

#### Some notes on breakpoints
- The `only` keyword in the example above is not required, but forces browsers that don’t support media queries to ignore that media block. If they didn’t do this, then the rules inside every media query block will be applied.
- It is logical to write styles for smaller devices at the beginning of your stylesheet, outside of a media query block. Afterwards, start to add media queries in order of viewport size. This means by default, devices with smaller viewports aren’t downloading large assets which they won’t be using.
- I find using the CSS preprocessor [Sass](http://sass-lang.com/) makes using media queries very easy. Instead of separating layout code into large viewport-specific blocks, Sass lets you use media queries within a selector block.
- It is tempting to predefine your breakpoint widths and use them throughout your CSS. I would argue that instead you should find the perfect breakpoint for each component you are styling.
- If you’re not using Sass, it might be helpful to split up media queries into separate CSS files. Be sure to glue them all back together as part of your build process for production.
- For browsers that don’t support media queries, try the [respond.js](https://github.com/scottjehl/Respond) polyfill.


## Mobile First
It is worthwhile to develop an interface for the least accessible devices first and gradually build on top of that for bigger and better platforms. This concept is known as *mobile first*.

There are several reasons for this:

- From a *designer’s* perspective, it means being able to focus on the important parts of a functional interface. More can be added later for more-capable devices, if necessary.
- From a *developer’s* perspective, it is normally far easier to build upon CSS incrementally, than simplify it incrementally. 
- From a *user’s* perspective, it means that smaller devices only need a minimal amount of styles and assets such as images, so they can benefit from fast load times. This is particularly important when neither wi-fi nor high-speed cellular data are available, or the user has a tight bandwidth allowance.


## The viewport meta tag
Some devices pretend to have a larger resolution than they actually do, thus undoing your hard work in optimising the site for that device. To avoid this, the following meta tag should be added to the head of your HTML:

~~~~ html
<meta name="viewport" content="width=device-width, initial-scale=1">
~~~~


You could also disable zooming (particularly useful for web-apps), by adding `user-scalable=no` to the content attribute list.

The meta tag hack was introduced by Apple for Safari on iPhone and was quickly adopted by other mobile browser vendors. However, a W3C standard aims to provide a proper CSS solution:

~~~~ css
@viewport {
  user-zoom: fixed;
  width: device-width;
  zoom: 1.0;
}
~~~~


Currently this method is only available in IE10 and Opera, with prefixing.


~~~~ css
@-ms-viewport {
  user-zoom: fixed;
  width: device-width;
  zoom: 1.0;
}

@-o-viewport {
  user-zoom: fixed;
  width: device-width;
  zoom: 1.0;
}
~~~~

- More about the [viewport meta tag](http://www.html5rocks.com/en/mobile/mobifying/#toc-meta) - HTML5 Rocks
- More about the upcoming [viewport CSS standard](http://blog.teamtreehouse.com/thinking-ahead-css-device-adaptation-with-viewport) - Treehouse blog


## Testing
Testing your responsive site with any accuracy is an extremely difficult thing to do. Resizing your browser window is good enough while you are developing and doing the same in other browsers is also worthwhile occasionally. However, your desktop is only one device and ideally you should be testing your site on many more.

Online services such as [BrowserStack](http://www.browserstack.com/) allow real-time testing on many emulated devices, usually for a cost. Using emulation isn’t always 100% reliable, but these sort of services are usually the only feasible solution.

Unfortunately, the only proper way to test your site is to get hold of a good collection of web-enabled devices. This is usually difficult, but if you’re lucky enough to have an [Open Device Lab](http://opendevicelab.com/) nearby, you should use it!


## Tips and Tricks
- Use the `box-sizing: border-box` CSS rule to make it easier to define exact column widths. Paul Irish has a [good article on box-sizing](http://www.paulirish.com/2012/box-sizing-border-box-ftw/).
- Think about your site navigation. It probably shouldn’t take up the whole screen on smaller devices. Consider placing at bottom of page, with anchor at the top, or as a toggle-box at the top of the page.
- Always be thinking about how to save bandwidth. Users of mobiles and tablets using roaming data will thank you.


## TL;DR
- RWD is a set of guidelines on how to design for viewport sizes, not specific devices.
- The three pillars of RWD are *fluid layouts*, *flexible images* and *breakpoints*.
- Sites should be designed and developed *mobile first*.
- Add the viewport meta tag to force devices to use their native resolutions.


## Worth a look
- Ethan Marcotte’s original article on [Responsive Web Design](http://alistapart.com/article/responsive-web-design).
- An interesting trick for [fluid grids using text-align: justify](http://www.barrelny.com/blog/text-align-justify-and-rwd/).
- Current status of the [srcset attribute](http://caniuse.com/#feat=srcset) and the [picture element](http://caniuse.com/#feat=picture).



*[SVG]: Scalable Vector Graphics
*[W3C]: World-Wide Web Consortium
