# spritesmith [![Build status](https://travis-ci.org/Ensighten/spritesmith.png?branch=master)](https://travis-ci.org/Ensighten/spritesmith)

Convert images into [spritesheets][] and coordinate maps.

[spritesheets]: http://en.wikipedia.org/wiki/Sprite_%28computer_graphics%29#Sprites_by_CSS

`spritesmith` is also available as:

- [grunt plugin](https://github.com/Ensighten/grunt-spritesmith)
- [gulp plugin](https://github.com/twolfson/gulp.spritesmith)
- [CLI utility](https://github.com/bevacqua/spritesmith-cli)

A folder of icons processed by `spritesmith`:

![Fork icon][fork-icon] ![+][]
![GitHub icon][github-icon] ![+][]
![Twitter icon][twitter-icon] ![=][]

[fork-icon]: docs/fork.png
[github-icon]: docs/github.png
[twitter-icon]: docs/twitter.png
[+]: docs/plus.png
[=]: docs/equals.png

generates a spritesheet:

![spritesheet](docs/spritesheet.png)

and a coordinate map:

```js
{
  "/home/todd/github/spritesmith/docs/fork.png": {
    "x": 0,
    "y": 0,
    "width": 32,
    "height": 32
  },
  "/home/todd/github/spritesmith/docs/github.png": {
    "x": 32,
    "y": 0,
    "width": 32,
    "height": 32
  },
  // ...
}
```

## Getting started
`spritesmith` can be installed via npm: `npm install spritesmith`

```js
// Load in dependencies
var spritesmith = require('spritesmith');

// Generate our spritesheet
var sprites = ['fork.png', 'github.png', 'twitter.png'];
spritesmith({src: sprites}, function handleResult (err, result) {
  result.image; // Binary string representation of image
  result.coordinates; // Object mapping filename to {x, y, width, height} of image
  result.properties; // Object with metadata about spritesheet {width, height}
});
```

## Documentation
`spritesmith` exports a `spritesmith` function as its `module.exports`.

If you would like a faster build time or need to support an obscure image format, see `params.engine`.

If you would like to adjust how images are laid out, see `params.algorithm` and `params.algorithmOpts`.

// TODO: Add examples with algorithms and whatnot

### `spritesmith(params, callback)`
Utility that takes images and generates a spritesheet, coordinate map, and spritesheet info

- params `Object` Container for paramters
    - params.src `String[]` Array of filepaths for images to include in spritesheet
    - params.padding `Number` Padding to use between images
        - For example if `2` is provided, then there will be a `2px` gap to the right and bottom between each image
    - params.engine `String|Object` Optional engine override to use
        - By default we use [`pixelsmith`][], a node-based `spritesmith` engine
        - For more engine options, see the [Engines section](#engines)
    - params.engineOpts `Object` Options to pass through to engine for settings
        - For example `phantomjssmith` accepts `timeout` via `{engineOpts: {timeout: 10000}}`
        - See your engine's documentation for available options
    - params.exportOpts `Mixed` Options to pass through to engine for export
        - For example `gmsmith` supports `quality` via `{exportOpts: {quality: 75}}`
        - // TODO: Verify `gmsmith` and others list their available export options
        - See your engine's documentation for available options
    - params.algorithm `String` Optional algorithm to pack images with
        - By default we use `top-down` which packs images vertically from smallest (top) to largest (bottom)
        - For more algorithm options, see the [Algorithms section](#algorithms)
    - params.algorithmOpts `Object` Optional algorithm options to pass through to algorithm for layout
        - For example `top-down` supports ignoring sorting via `{algorithmOpts: {sort: false}}`
        - See your algorithm's documentation for available options
            - https://github.com/twolfson/layout#algorithms
- callback `Function` Error-first function that receives compiled spritesheet and map
    - `callback` should have signature `function (err, result)`
    - err `Error|null` If an error occurred, this will be it
    - result `Object` Container for result items
        - result.image `String` Binary string representation of image
        - result.coordinates `Object` Map from filepath to coordinate information between original sprite and spritesheet
            - `filepath` will be the same as provided in `params.src`
            - result.coordinates[filepath] `Object` Container for coordinate information
                - result.coordinates[filepath].x `Number` Horizontal position of top-left corner of original sprite on spritesheet
                - result.coordinates[filepath].y `Number` Vertical position of top-left corner of original sprite on spritesheet
                - result.coordinates[filepath].width `Number` Width of original sprite
                - result.coordinates[filepath].height `Number` Height of original sprite
        - result.properties `Object` Container for information about spritesheet
            - result.properties.width `Number` Width of the spritesheet
            - result.properties.height `Number` Height of the spritesheet

[`pixelsmith`]: https://github.com/twolfson/pixelsmith

### Algorithms
Images can be laid out in different fashions depending on the algorithm. We use [`layout`][] to provide you as many options as possible. At the time of writing, here are your options for `params.algorithm`:

[`layout`]: https://github.com/twolfson/layout

|         `top-down`        |          `left-right`         |         `diagonal`        |           `alt-diagonal`          |          `binary-tree`          |
|---------------------------|-------------------------------|---------------------------|-----------------------------------|---------------------------------|
| ![top-down][top-down-img] | ![left-right][left-right-img] | ![diagonal][diagonal-img] | ![alt-diagonal][alt-diagonal-img] | ![binary-tree][binary-tree-img] |

[top-down-img]: https://raw.githubusercontent.com/twolfson/layout/2.0.2/docs/top-down.png
[left-right-img]: https://raw.githubusercontent.com/twolfson/layout/2.0.2/docs/left-right.png
[diagonal-img]: https://raw.githubusercontent.com/twolfson/layout/2.0.2/docs/diagonal.png
[alt-diagonal-img]: https://raw.githubusercontent.com/twolfson/layout/2.0.2/docs/alt-diagonal.png
[binary-tree-img]: https://raw.githubusercontent.com/twolfson/layout/2.0.2/docs/binary-tree.png

More information can be found in the [`layout`][] documentation:

https://github.com/twolfson/layout

### Engines
An engine can greatly improve the speed of your build (e.g. `canvassmith`) or support obscure image formats (e.g. `gmsmith`).

All `spritesmith` engines adhere to a common specification and test suite:

https://github.com/twolfson/spritesmith-engine-test

Below is a list of known engines with their tradeoffs:

#### pixelsmith
[`pixelsmith`][] is a `node` based engine that runs on top of [`get-pixels`][] and [`save-pixels`][].

[`get-pixels`]: https://github.com/mikolalysenko/get-pixels
[`save-pixels`]: https://github.com/mikolalysenko/save-pixels

**Key differences:** Doesn't support uncommon image formats (e.g. `tiff`) and not as fast as a compiled library (e.g. `canvassmith`).

#### phantomjssmith
[`phantomjssmith`][] is a [phantomjs][] based engine. It was originally built to provide cross-platform compatibility but has since been succeeded by [`pixelsmith`][].

**Requirements:** [phantomjs][] must be installed on your machine and on your `PATH` environment variable. Visit [the phantomjs website][phantomjs] for installation instructions.

**Key differences:** `phantomjs` is cross-platform and supports all image formats.

[`phantomjssmith`]: https://github.com/twolfson/phantomjssmith
[phantomjs]: http://phantomjs.org/

#### canvassmith
[`canvassmith`][] is a [node-canvas][] based engine that runs on top of [Cairo][].

**Requirements:** [Cairo][] and [node-gyp][] must be installed on your machine.

Instructions on how to install [Cairo][] are provided in the [node-canvas wiki][].

[node-gyp][] should be installed via `npm`:

```bash
npm install -g node-gyp
```

**Key differences:** `canvas` has the best performance (useful for over 100 sprites). However, it is `UNIX` only.

[`canvassmith`]: https://github.com/twolfson/canvassmith
[node-canvas]: https://github.com/learnboost/node-canvas
[Cairo]: http://cairographics.org/
[node-canvas wiki]: (https://github.com/LearnBoost/node-canvas/wiki/_pages
[node-gyp]: https://github.com/TooTallNate/node-gyp/

#### gmsmith
[`gmsmith`][] is a [`gm`][] based engine that runs on top of either [Graphics Magick][] or [Image Magick][].

**Requirements:** Either [Graphics Magick][] or [Image Magick][] must be installed on your machine.

For the best results, install from the site rather than through a package manager (e.g. `apt-get`). This avoids potential transparency issues which have been reported.

[Image Magick][image-magick] is implicitly discovered. However, you can explicitly use it via `engineOpts`

```js
{
  engineOpts: {
    imagemagick: true
  }
}
```

**Key differences:** `gmsmith` allows for configuring image quality whereas others do not.

[`gmsmith`]: https://github.com/twolfson/gmsmith
[`gm`]: https://github.com/aheckmann/gm
[Graphics Magick]: http://www.graphicsmagick.org/
[Image Magick]: http://imagemagick.org/

## Contributing
In lieu of a formal styleguide, take care to maintain the existing coding style. Add unit tests for any new or changed functionality. Lint via `npm run lint` and test via `npm test`.

## Donating
Support this project and [others by twolfson][gratipay] via [gratipay][].

[![Support via Gratipay][gratipay-badge]][gratipay]

[gratipay-badge]: https://cdn.rawgit.com/gratipay/gratipay-badge/2.x.x/dist/gratipay.png
[gratipay]: https://www.gratipay.com/twolfson/

## License
Copyright (c) 2012-2014 Ensighten

Licensed under the MIT license.
