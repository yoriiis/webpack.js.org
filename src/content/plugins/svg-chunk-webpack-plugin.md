---
title: SvgChunkWebpackPlugin
contributors:
  - yoriiis
---

> Generate SVG sprites according to entrypoint dependencies. Each page only imports its own svgs, wrapped as a sprite and optimized by svgo.

The SvgChunkWebpackPlugin creates optimized SVG sprites, according to Webpack's entrypoints. Each sprite contains only the SVG dependencies listed on its entrypoints to improved code splitting, even on SVG files.

The plugin includes the popular [svgo](https://github.com/svg/svgo) package with the optimized settings, to generates clean and optimized SVG sprites.

Code splitting is the key to deliver files without any content that is unused by the pages. It already exists for CSS, Javascript and now for SVG files with this plugin.

## When to use this plugin

On multiple page application, each pages must includes only its necessary dependencies. In other words, it must include only the SVG files imported by its entrypoint and all its dependencies.

With reusable components, SVGs are often duplicated on all the project. Now, you can create a global SVG library and every Javascript files can easily import any SVG from this library. Entrypoint dependencies are automatically updated, thanks to the Webpack compilation.

When you work with SVGs exported by design softwares, like Sketch or Illustrator, their source code is never optimized and often contains comments, CSS classes which can create conflicts between them. The plugin automatically cleans all SVGs before creating the sprite.

## Zero config

The plugin works without configuration with already the optimized settings. For advanced usage, see the section [using configuration](#using-a-configuration).

## Installation

The plugin is available as a package with the name of `svg-chunk-webpack-plugin` on [npm](https://www.npmjs.com/package/svg-chunk-webpack-plugin) and [Github](https://github.com/yoriiis/svg-chunk-webpack-plugin).

```bash
npm install svg-chunk-webpack-plugin --save-dev
```

```bash
yarn add svg-chunk-webpack-plugin  --dev
```

## Environment

SvgChunkWebpackPlugin was built for Node.js `>=8.11.2` and Webpack `>=4.x`.

## Example

For example implementation, please see the [Github documentation](https://github.com/yoriiis/svg-chunk-webpack-plugin/blob/master/README.md#example).

## Basic usage

The plugin will generate one SVG sprite for each entrypoints. Sprites are built in the output path directory with all the other assets. Each sprite filename is composed with its entrypoint name (in the example below, that would be `home.svg`).

First, let's add the loader and the plugin to the Webpack configuration.

T> The loader and the plugin need to works together.

__webpack.config.js__

```js
var SvgChunkWebpackPlugin = require('svg-chunk-webpack-plugin');
var path = require('path');

module.exports = {
  entry: {
    home: 'home.js'
  },
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'main.js'
  },
  module: {
    rules: [
      {
        test: /\.svg$/,
        use: [
          {
            loader: SvgChunkWebpackPlugin.loader
          }
        ]
      }
    ]
  },
  plugins: [new SvgChunkWebpackPlugin()]
};
```

T> For more flexibility and better performance, inline SVG files are better. Fewer HTTP requests, CSS properties to change the style, no flickering during the page load.

Then, include the sprite in the wanted pages (_we use Twig in the following example_).

__home.html.twig__

```twig
{{ include 'home.svg' }}
```

Finally, use the SVG with the `<use>` tag, like the following example. Replace `<SVG_FILENAME>` by the SVG filename, without the extension.

__home.html.twig__

```html
<svg>
  <use xlink:href="#<SVG_FILENAME>"></use>
</svg>
```

---

## Using a configuration

You can pass configuration options to SvgChunkWebpackPlugin to overrides default settings. Allowed values are as follows:

### `filename`

`String`

Tells the plugin whether to personalize the default sprite filename. The placeholder `[name]` is automatically replaced by entrypoints names.

```js
new SvgChunkWebpackPlugin({
  filename: '[name].svg'
});
```

The `filename` parameter is compatible with Webpack caching placeholders, see the section [caching](#caching).

### `svgoConfig`

`Object`

Tells the plugin whether to personalize the default settings for svgo. Update the settings according to your needs from the plugins available on the [svgo](https://github.com/svg/svgo) documentation.


```js
new SvgChunkWebpackPlugin({
  svgoConfig: {
    plugins: [
      {
        inlineStyles: {
          onlyMatchedOnce: false
        }
      }
    ]
  }
});
```

T> The `onlyMatchedOnce` property allows to replace all occurences of CSS classes in HTML attributes, not only selectors that match once.

### `svgstoreConfig`

`Object`

SVG sprites are built using the svgstore package. Tells the plugin whether to personalize the default settings for [svgstore](https://github.com/svgstore/svgstore#options).

```js
new SvgChunkWebpackPlugin({
  svgstoreConfig: {
    cleanDefs: false,
    cleanSymbols: false,
    inline: true,
    svgAttrs: {
      'aria-hidden': true,
      style: 'position: absolute; width: 0; height: 0; overflow: hidden;'
    }
  }
});
```

T> Sprites already contain minimal inline styles to hide the sprite on the page to keep full support with all SVG features.

W> To avoid LinearGradient conflicts, avoid the `display: none` property which break SVG defs.

### `generateSpritesManifest`

`Boolean = false`

Tells the plugin whether to generate the `sprites-manifest.json`. The JSON file contains the list of all SVG included by entrypoints. It becomes very easy to known which SVG are included in which sprite.

```js
new SvgChunkWebpackPlugin({
  generateSpritesManifest: false
});
```

T> The file is only generated when the Webpack mode is set to `development`, even if the option is enabled.

### `generateSpritesPreview`

`Boolean = false`

Tells the plugin whether to generate the `sprites-preview.html`. The HTML preview contains a display list of all SVG included by entrypoints with the SVG overviews and the names.

```js
new SvgChunkWebpackPlugin({
  generateSpritesManifest: false
});
```

T> The file is only generated when the Webpack mode is set to `development`, even if the option is enabled.

---

## Caching

With [Webpack caching](https://webpack.js.org/guides/caching), several placeholders are available depending on your needs. With SVG inlined in the page, this option is not useful.

T> The `[contenthash]` placeholder is the best option because it depends on the sprite content.

W> Cache placeholders are expensive in build performance, use it only in production mode.

### `[hash]`

The `[hash]` placeholder will add a unique hash generated for every build. When the compilation build is updated, `[hash]` will change as well. See the [`stats.hash`](https://webpack.js.org/configuration/stats/#statshash) on the webpack documentation.

```js
new SvgChunkWebpackPlugin({
  filename: '[name].[hash].svg'
});
```

### `[chunkhash]`

The `[chunkhash]` placeholder will add a unique hash based on the content of the entrypoint. When the entrypoint's content changes, `[chunkhash]` will change as well. All the entrypoint dependencies will have the same hash.

```js
new SvgChunkWebpackPlugin({
  filename: '[name].[chunkhash].svg'
});
```

### `[contenthash]`

The `[contenthash]` placeholder will add a unique hash based on the content of the sprite. When the sprite's content changes, `[contenthash]` will change as well.

```js
new SvgChunkWebpackPlugin({
  filename: '[name].[contenthash].svg'
});
```
