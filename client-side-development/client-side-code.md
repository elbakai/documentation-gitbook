@ngdoc overview
@name Client-Side Code

@description
# Client-Side Code

SocketStream allows you to write and structure client-side Javascript in exactly the same way as server-side code, allowing you to easily share modules between both.


### How to use Modules

All files which aren't `libs` (see below) are treated as modules. You have exactly the same ability to export functions, `require()` other modules, and cache values within modules as you do when writing server-side code in Node.js.

Client-side code lives in `/client/code`. Create as many subdirectories as you wish. Reference your modules relatively, e.g. `require('../../image/processor')`, or absolutely `require('/path/to/my/module.js')`. It all work as you would expect, just bear in mind a module will never be executed unless it is explicitly `require()`'d.

Top tip: Type `require.modules` in the browser console to see a list of all modules you can `require()` in your app


### Special Exceptions

While we try to keep the experience between browser and server as similar as possible, there are a few special cases to be aware of:


#### 'libs' - Legacy (non Common JS) Libraries

Any file which lives in a directory called 'libs' will NOT be served as a module. Instead these files will be sent as-is without any modification. Typically you'll want to ensure jQuery and other libraries which use the `window` variable are always placed in a `/client/code` directory called 'libs'.

As load order is critically important for non Common JS libraries **either** name your files alphanumerically within the `libs` directory **or** list each file explicitly in your `ss.client.define()` command - your choice.


#### 'system' - System Modules

System modules are similar to regular modules but with one important difference: they are accessed without a leading slash - just like you would `require('url')` or `require('querystring')` in Node.js.

So why do we need this distinction? Because some libraries such as Backbone.js (when used as a module, rather than in a 'libs' directory) depend upon other system modules. In this case Backbone calls `require('underscore')` internally, therefore both `backbone.js` and `underscore.js` must live in a `system` directory.

As SocketStream uses code from [Browserify](https://github.com/substack/node-browserify), the 'system' directory also allows you to use one of Node's inbuilt modules in the browser. Just head over to https://github.com/substack/node-browserify/tree/master/builtins and copy the libraries you need into any directory within `/client/code` called `system`.

Tip: If you're making a new folder called `/client/code/system`, don't forget to add `system` to the list of code directores to serve in the `ss.client.define()` statement within `app.js`.


#### '/entry.js' - A single point of entry

The `entry` module has a special distinction: it is the only module to be required automatically once all files have been
sent to the browser.

An `entry.js` (or `entry.coffee`) file is created for you in the `app` view by default when you make a new project.
It contains a small amount of boiler-plate code which you may modify to handle the websocket connection going down,
reconnecting, and (critically), what module to `require()` next once the websocket connection is established.

The `code` is set to `['libs/jquery.min.js', 'app']`. The first module with an `entry.js` is taken as the main module for the view.
So in this case the view is started automatically with the call `require('./code/app/entry');`. This allows you to share code
directories, and use more than one in a view.

### Implications for ondemand loading in production

In production code is loaded as a single bundle, but as additional modules can be loaded on demand the code is copied into the static assets.
The code should be under `/client/static/assets/<namespace>/code`. By default the namespace is the client name, but if you want to share
code you can define a common namespace. During development this will be served on-the-fly.

### Should I put library X in 'libs' or 'system'?

It depends if it needs access to the `window` variable. For example, Backbone.js works great as a `system` module unless you're using Backbone.history as this requires access to `window`.


### What happens if I try to require a module which doesn't exist?

You'll see an error in the browser's console. In the future SocketStream will be able to catch these problems before they arise.


### Using require in the browser

Within client view code you should generally require another module by relative path (`"./controller"`) or system module (`"facility"`).
If you use absolute path it will be relative to the client directory, so if you want to require a directory module within the default
code directory do something like,

    require("/code/app/part1")

This will load the module for the source file `/code/app/part1/index.js` or `/code/app/part1/index.coffee`. If part1 isn't a directory
it can also be used to load `/code/app/part1.js`. You could also do this explicitly using `require("/code/app/part1/index.js")`,
but this isn't advised as it locks you in to the exact source structure.



### Loading modules on demand

You don't necessarily have to send all modules to the browser at once, you can also [load them on demand](https://github.com/socketstream/socketstream/blob/master/doc/guide/en/loading_assets_on_demand.md).

### Options
- **browserifyExcludePaths {Array} ** - to disable automatically packaging client side code into modules for certain paths, for example `client/code/app/controllers/**/*.js` and `client/code/app/directives/**/*.js` put the following in your server-side `app.js` code:
<pre>
ss.client.set({ browserifyExcludePaths: ['app/controllers', 'app/directives'] });
</pre>
- **entryModuleName {string} ** - to change or disable automatically execuring the client side
code for the entry module. If null or a blank string is given no entry module is executed during load of the view in the browser. If for example your app is `my` and entryModuleName is `app` the view will execute `client/code/my/app.js`. Be aware that this name will apply to all views.
<pre>
ss.client.set({ entryModuleName: null });
</pre>

**Note**, that paths for excluding should be relative to `client/code/` and that file `client/code/app/entry.js` could not be excluded in any cases.

If you need to exclude from automatically packaging certain file, just specify the file's relative path:
<pre>
ss.client.set({ browserifyExcludePaths: ['app/routes.js'] });
</pre>

### Background info

Getting client-code right was a major goal for SocketStream from the beginning.

For too long web developers have had to wade through a mess of unstructured JavaScript files without anyway to manage namespacing or dependencies.

Solutions such as [Require.js](http://requirejs.org) and other AMD approaches have successfully brought order to chaos, but put the onus on the developer to manually track and list dependencies. What's more, they use a different syntax to `require()` files - instantly killing all hopes of sharing the same file between the client and server.

We wanted to do much better with SocketStream. After all, we are in the unique position of managing both the client and server stack. The solution came in the form of [Browserify](https://github.com/substack/node-browserify) - an awesome, lightweight, library which solves all these problems once and for all.

SocketStream doesn't depend upon the Browserify module (as it contains code we don't need), but we use major components from it (including the critical code which performs all the `require()` magic). Our thanks go to Substack for coming up with a clean solution to a very tricky problem.
