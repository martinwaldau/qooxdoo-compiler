# Splash Screens
Splash screens are an optional feature of the application loader in `qooxdoo-compiler` that will provide an attractive feedback with a progress bar all the way through your application's startup.

By default, the loader does not provide any splash screen or progress bar; to add one, you need to replace the application's `.html` file with your own, and provide a global object called `QOOXDOO_SPLASH_SCREEN` that the boot loader can call during the boot process.

## Loading Speed
Splash screens will slow down the load fractionally, but only in the order of 100-200ms.  It's important to remember that loading speed is subjective, and as a developer the progress bar emphasises an unnecessary slow down of the load, whereas to a user a blank screen emphasises waiting for startup.  But if you measure it, the difference in time is negligable.

The new loader supports URL query parameters which allow you to customise the startup to fine tune or eliminate the boot loader so that you can time various different approaches.  If you have installed a boot loader, try appending `?qooxdoo:feedback-disable=true` to your application's URL and comparing boot times.  For more information on loader URLs, see [qooxdoo-compiler/docs/LoaderUrls.md](https://github.com/qooxdoo/qooxdoo-compiler/blob/master/docs/LoaderUrls.md)

## Example
For a working example, take a look at [qooxdoo-compiler/demos/boot/](https://github.com/qooxdoo/qooxdoo-compiler/tree/master/demos/boot/) - if you add this folder to your application's `source` folder and re-run `qx compile`, it will have a splash screen with loading progress bar.

## Chunking
Previously, the only option for loading scripts was either asynchronously (also called "parallel" loading) or not.  Asynchronous loading queues all scripts to be loaded by the browser (which could be hundreds thousands) and allows the browser to go ahead and load as fast as possible.  For splash screens this is not ideal because the browser is working so hard at loading scripts that it does not redraw the display with any updated progress information, so from the point of view of the user the progress bar will be stuck at a low number until the application is suddenly running.

To solve this, chunking (enabled by default *only if* there a `QOOXDOO_SPLASH_SCREEN` object) divides the list of scripts to load into "chunks" and adds a small 1ms timeout between each chunk to allow the browser to redraw the screen.  This slows the loading because of the 1ms delay and the time it takes to redraw, but on the other hand your user will get feedback all the way through the load.

By default the chunks are 20% of the total number of scripts, but you can override this with the `getSettings` method in the API.

## API

`QOOXDOO_SPLASH_SCREEN` is a plain Javascript object that looks like this:
```
  window.QOOXDOO_SPLASH_SCREEN = {
    loadBegin: function(callback) {
      /* ... snip ... */
      callback();
    },
    
    loadComplete: function(callback) {
      /* ... snip ... */
      callback();
    },
    
    scriptLoaded: function(options, callback) {
      /* ... snip ... */
      callback();
    },
    
    getSettings: function() {
      /* ... snip ... */
      return null;
    }
  };

```

The `options` parameter has a `command` property (e.g. `"script-loaded"`) that determines what the other properties of the object will be.

### `loadBegin(callback)`
This is fired immediately before scripts start to be loaded; `callback()` must be called at the end (script loading will not begin until it has been called) 

### `loadComplete(callback)`
This is fired when all scripts are loaded and the application is about to startup; `callback()` must be called at the end (the application will not start until it has been called) 

### `scriptLoaded(options, callback)`
This is fired every time a script completes loading, and has these properties in `options`:
- `numScripts` - total number of scripts to load
- `numScriptsLoaded` - number of scripts fully loaded so far
- `numScriptsLoading` - number of scripts in progress

### `getSettings()`
This is called once before any scripts are loaded to allow the loader to override default settings.  The result can be null, or an object which has these properties:
- `isLoadChunked` - (**optional**) whether to load scripts in "chunks"
- `loadChunkSize` - (**optional**) if loading with chunks, how big each chunk should be (default is 20% of the number of scripts)
