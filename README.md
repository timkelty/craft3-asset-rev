![image](./media/logo.png)

# Craft 3 Asset Rev (Cache Busting)
[![Build Status](https://travis-ci.org/clubstudioltd/craft3-asset-rev.svg?branch=master)](https://travis-ci.org/clubstudioltd/craft3-asset-rev)
[![Total Downloads](https://poser.pugx.org/clubstudioltd/craft3-asset-rev/downloads)](https://packagist.org/packages/clubstudioltd/craft3-asset-rev)
[![Latest Stable Version](https://poser.pugx.org/clubstudioltd/craft3-asset-rev/v/stable)](https://packagist.org/packages/clubstudioltd/craft3-asset-rev)
[![License](https://poser.pugx.org/clubstudioltd/craft3-asset-rev/license)](https://packagist.org/packages/clubstudioltd/craft3-asset-rev)

A Twig extension for Craft 3 that helps you cache-bust your assets by appending a query string or swapping out asset file names with their revved version, as they are defined in a JSON manifest file.

Manifest files would most likely be generated by Grunt/Gulp modules, such as [grunt-filerev-assets](https://github.com/richardbolt/grunt-filerev-assets) or [gulp-rev](https://github.com/sindresorhus/gulp-rev).

## Why?
In order to speed up the load time of your pages, you can set a far-future expires header on your images, stylesheets and scripts. However, when you update those assets you'll need to update their file names to force the browser to download the updated version.

Using a manifest file is the recommended approach - you can read up on why using query strings isn't ideal [here](http://www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/).

## Installation

Install via composer:

```
composer require clubstudioltd/craft3-asset-rev
```

or, download the plugin and copy the contents of `src` into a folder called `assetrev` in your `plugins` directory.

Be sure to activate the plugin from the Craft plugins settings page. Once activated, you may want to specify a custom path to your asset manifest file within the plugin configuration.

## Configuration
The plugin comes with a `config.php` file that defines some sensible defaults.

If you want to set your own values you should create a `assetrev.php` file in your `config` directory. The contents of this file will get merged with the plugin defaults, so you only need to specify values for the settings you want to override.

### Manifest Path
`manifestPath` is where Craft should look for your manifest file. Non-absolute paths will be relative to the base path of your Craft installation (whatever `CRAFT_BASE_PATH` is set to).

### Assets Base Path
`assetsBasePath` is the the base path to your assets. Again, this is relative to your craft base directory, unless you supply an absolute directory path.

### Asset Url Prefix
`assetUrlPrefix` will be prepended to the output of `rev()`.

**Note:** You can use any environment variables that you may have set in your `.env` file using the `getenv()` function.

### Example assetrev.php Config File

```
<?php
return array(
    'manifestPath' => 'resources/assets/assets.json',
    'assetsBasePath' => '../public/build/',
    'assetUrlPrefix' => getenv('ASSET_URL_PREFIX'),
);
```

## Usage
Once activated and configured you can use the `rev()` function in your templates.

```
<link rel="stylesheet" href="{{ rev('css/main.css') }}">
```

In some cases (e.g. when building additional files that aren't available in the manifest file or are files that are served via proxy), you can prevent the extension from throwing an exception about a missing file mapping by setting the optional `$strict` parameter to `false`: 

```
<link rel="stylesheet" href="{{ rev('css/not-available-in-manifest.css', false) }}">
```

This will append a query string (see below) if the file does not exist in the manifest file: `css/not-available-in-manifest.css?1473534554`.

### Manifest Files

`css/main.css` will be replaced with the corresponding hashed filename as defined within your assets manifest .json file.

If the contents of your manifest file are...

```
{
    "css/main.css": "css/main.a9961d38.css",
    "js/main.js": "js/main.786087f5.js"
}
```

then `rev('css/main.css')` will expand to `css/main.a9961d38.css`.

### Query String Fallback

If the plugin can't find a valid manifest file it will fall back to appending a query string to your file, based on the time it was last modified. In this scenario `rev('css/main.css')` will expand to something like `css/main.css?1473534554`.
