![](http://i.imgur.com/BGrt2QK.png)

## Ionic Framework generator [![Build Status][travis-image]][travis-url] [![NPM version][npm-image]][npm-url] [![Gittip][gittip-image]][gittip-url]

> Yeoman generator for Ionic - lets you quickly set up a hybrid mobile app project

**Note**: This was formally packaged under `generator-ionicjs`, but has been republished as `generator-ionic`.

## Usage
Install `generator-ionic`
```
npm install -g generator-ionic
```

Make a new directory, and `cd` into it
```
mkdir my-ionic-project && cd $_
```

Run `yo ionic`
```
yo ionic
```

Follow the prompts to select from some common plugins and pick a starter template, then spin up a `connect` server with `watch` and `livereload` for developing in your browser
```
grunt serve
```
## Upgrading
Make sure you've commited (or backed up) your local changes and install the latest version of the generator via `npm install -g generator-ionic`, then go ahead and re-run `yo ionic` inside your project's directory.

The handsome devil is smart enough to figure out what files he is attempting to overwrite and prompts you to choose how you would like to proceed. Select `Y` for overwriting your `Gruntfile.js` and `bower.json` to stay up-to-date with the latest workflow goodies and front-end packages.

## Workflow
The included Grunt build system provides sensible defaults to help optimize and automate several aspects of your workflow when developing hybrid-mobile apps using the Ionic Framework.

### Managing libraries with Bower
Install a new front-end library using `bower install --save` to update your `bower.json` file.
```
bower install --save lodash
```
This way, when the Grunt [`bower-install`](https://github.com/stephenplusplus/grunt-bower-install#grunt-bower-install) task is run it will automatically inject your front-end dependencies inside the `bower:js` block of your `app/index.html` file.

### Manually adding libraries
If a library you wish to include is not registered with Bower or you wish to manually manage third party libraries, simply include any CSS and JavaScript files you need **inside** your `app/index.html` [usemin](https://github.com/yeoman/grunt-usemin#blocks) `build:js` or `build:css` blocks but **outside** the `bower:js` or `bower:css` blocks (since the Grunt task overwrites the Bower blocks' contents).

### Local browser development
Running `grunt serve` enhances your workflow by allowing you to rapidly build Ionic apps without having to constantly re-run your platform simulator. Since we spin up a `connect` server with `watch` and `livereload` tasks, you can freely edit your CSS (or SCSS/SASS files if you chose to use Compass), HTML, and JavaScript files and changes will be quickly reflected in your browser.

### Building assets for Cordova
Once you're ready to test your application in a simulator or device, run `grunt cordova` to copy all of your `app/` assets into `www/` and build updated `platform/` files so they are ready to be emulated / run by Cordova.

To compress and optimize your application, run `grunt build`. It will concatenate, obfuscate, and minify your JavaScript, HTML, and CSS files and copy over the resulting assets into the `www/` directory so the compressed version can be used with Cordova.

### Cordova commands
To make our lives a bit simpler, the `cordova` library has been packaged as a part of this generator and delegated via Grunt tasks. To invoke Cordova, simply run the command you would normally have, but replace `cordova` with `grunt` and `spaces` with `:` (the way Grunt chains task arguments).

For example, lets say you want to add iOS as a platform target for your Ionic app
```
grunt platform:add:ios
```
and emulate a platform target
```
grunt emulate:ios
```
or add a plugin by specifying either its full repository URL or namespace from the [Plugins Registry](http://plugins.cordova.io)
```
grunt plugin:add:https://git-wip-us.apache.org/repos/asf/cordova-plugin-device.git
grunt plugin:add:org.apache.cordova.device
grunt plugin:add:org.apache.cordova.network-information
```
## Icons and Splash Screens
Properly configuring your app icons and splash screens to work with Cordova can be a pain to set up, so we've gone ahead and including an `after_prepare` hook that manages copying these resource files to the correct location within your current platform targets.

To get started, you must first add a platform via `grunt platform:add:ios`. Once you have a platform, the packaged `icons_and_splashscreens.js` hook will copy over all placeholder icons and splash screens generated by Cordova into a newly created top-level `resources/` directory inside of your project. Simply replace these files with your own resources (**but maintain file names and directory structure**) and let the hook's magic automatically manage copying them to the appropriate location for each Cordova platform, all without interrupting your existing workflow.

To learn more about hooks, checkout out the `README.md` file inside of your `hooks/` directory.

## Environment Specific Configuration
While building your Ionic app, you may find yourself working with multiple environments such as development, staging, and production. This generator uses [grunt-ng-constant] (https://github.com/werk85/grunt-ng-constant) to set up your workflow with development and production environment configurations right out of the box. 

### Adding Constants
To set up your environment specific constants, modify the respective targets of the `ngconstant` task located towards the top of your Gruntfile.
```
    ngconstant: {
      options: {
        space: '  ',
        wrap: '"use strict";\n\n {%= __ngModule %}',
        name: 'config',
        dest: '<%= yeoman.app %>/scripts/config.js'
      },  
      development: {
        constants: {
          ENV: {
            name: 'development',
            apiEndpoint: 'http://dev.your-site.com:10000/'
          }   
        }   
      },  
      production: {
        constants: {
          ENV: {
            name: 'production',
            apiEndpoint: 'http://api.your-site.com/'
          }   
        }   
      }   
    }, 
```

Running `grunt serve` will cause the `development` target constants to be used. When you build your application for production using `grunt build` or `grunt serve:dist`, the `production` constants will be used. Other targets, such as staging, can be added, but you will need to customize your Gruntfile accordingly. Note that if you change the `name` property of the task options, you will need to update your `app.js` module dependencies as well.

### Using Inside Angular
A `config.js` file is created by `grunt-ng-constant` depending on which task target is executed. This config file exposes a `config` module, that is listed as a dependency inside `app/scripts/app.js`. Out of the box, your constants will be namespaced under `ENV`, but this can be changed by modifying the `ngconstant` targets. It is important to note that whatever namespace value is chosen is what will need to be used for Dependency Injection inside your Angular functions.

The following example shows how to pre-process all outgoing HTTP request URLs with an environment specific API endpoint by creating a simple Angular `$http` interceptor.

```
// Custom Interceptor for replacing outgoing URLs                
.factory('httpEnvInterceptor', function (ENV) {
  return {
    'request': function(config) {
      if (!_.contains(config.url, 'html')) {
        config.url = ENV.apiEndpoint + config.url;
      }
      return config;
    }
  }
})

.config(function($httpProvider) {
  // Pre-process outgoing request URLs
  $httpProvider.interceptors.push('httpEnvInterceptor');
})
```

## Initial Walkthrough
To help you hit the ground running, let's walk through an example workflow together. We're assuming you've followed the [usage](https://github.com/diegonetto/generator-ionic#usage) directions and are inside your app's directory.

We'll start by running our app in a browser so we can make a few changes.
```
grunt serve
```
Play around with livereload by changing some of the styles in `app/styles/main.css` or HTML in one of the files in `app/templates/`. When you're ready, lets go ahead and build the assets for Cordova to consume and also spot check that we didn't bork any code during the build process. We can do that with another handy Grunt task that runs the build process and then launches a `connect` server for use to preview the app with our built assets.
```
grunt serve:dist
```
If everything looks good the next step is to add a platform target and then emulate our app. In order for us to launch the iOS simulator from the command line, we'll have to install the `ios-sim` package. (If you forget to do this, Cordova will kindly remind you).
```
npm install -g ios-sim
grunt platform:add:ios
grunt emulate:ios
```
You may have realized that when the Grunt build process is run, it triggers the Cordova build system as well, so you end up with a beautifully packaged mobile app in a single command.

## Testing Your App
To lessen the pain of testing your application, this generator configures your project with a handful of libraries that will hopefully make testing your application, dare I say, more enjoyable.

![Comic](http://dilbert.com/dyn/str_strip/000000000/00000000/0000000/100000/10000/6000/600/116640/116640.strip.gif)

### Unit Tests
The foundation of our testing solution is built using [Karma](http://karma-runner.github.io/) which was created by the AngularJS team and is all around awesome. Inside of your generated `karma.conf.js` file you will find some basic configuration settings. Notice that we're using [Mocha](http://visionmedia.github.io/mocha/) to structure our tests and pulling in [Chai](http://chaijs.com/), a slick assertion library. You can easily drop Chai and replace Mocha with [Jasmine](http://jasmine.github.io/) depending on your preference.

Your generated `Gruntfile.js` also contains a `karma` task that provides further configuration. Any properties specified via this task will override the values inside `karma.conf.js` when run via `grunt`. If you look closely at this task, you'll notice that we're using [PhantomJS](http://phantomjs.org/) for both Karma targets, but you can easily update the `karma:unit` target to run tests inside of real browsers.

Ok, now that you have some context (and links to read up on the bundled testing libraries), go ahead and run `grunt test` and open up one of the included unit tests - `test/spec/controllers.js`. In your editor of choice, change this line - `scope.pets.should.have.length(4);` to any number other than four and watch what happens. Since your test files are being watched for changes, Grunt knows to go ahead and re-run your test suite, which in this cause should have errored out with a failure message being displayed in your terminal.

Undo your modification and ensure that all tests are passing before continuing on.

**Note** Depending on which starter template you picked, your tests may start off failing.

### End2End Tests
Coming soon!

### Code Coverage
So you've finished writing your tests, why not showoff just how watertight your application has become? Using [Istanbul](http://gotwarlost.github.io/istanbul/) - which was built at Yahoo - we can generate visually engaging code coverage reports that do just that!

Our beloved generator has done all the hard work for you, so go ahead and see these coverage reports in action by running `grunt coverage`.

If this is your first time using Istanbul, take a look around. It will help you spot gaps in your unit tests and lets face it, the more visual gratification we can get during our testing stage, the greater the likelihood of us sitting down and writing these tests to begin with!

## Wrapping it up
If you made it this far then congratulations! You're now up and running with the gorgeous Ionic Framework powered by an intelligent workflow and sophisticated build system - all facilitated by the addition of just a few commands!

## Ripple Emulator (Experimental)
**Be Advised**: [Ripple](http://ripple.incubator.apache.org/) is under active development so expect support for some plugins to be missing or broken.

Add a platform target then run `grunt ripple` to launch the emulator in your browser.
```
grunt platform:add:ios
grunt ripple
```

Now go edit a file and then refresh your browser to see your changes. (Currently experimenting with livereload for Ripple)

**Note**: If you get errors beginning with `Error: static() root path required`, don't fret. Ripple defaults the UI to Android so just switch to an iOS device and you'll be good to go.

![Ripple](http://i.imgur.com/LA4Hip1l.png)


## Special Thanks To
* The pioneers behind [Yeoman](http://yeoman.io/) for building an intelligent workflow management solution.
* The [AngularJS Generator](https://github.com/yeoman/generator-angular) and [Ionic Seed Project](https://github.com/driftyco/ionic-angular-cordova-seed) projects for inspiration.
* The visionaries at [Drifty](http://drifty.com) for creating the [Ionic Framework](http://ionicframework.com/).

## Contribute

See the [contributing docs](https://github.com/diegonetto/generator-ionic/blob/master/contributing.md).

When submitting an issue, please follow the [guidelines](https://github.com/diegonetto/generator-ionic/blob/master/contributing.md#issue-submission). Especially important is to make sure `generator-ionic` is up-to-date, and providing the command or commands that cause the issue.


When submitting a PR, make sure that the commit messages match the [AngularJS conventions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/).

When submitting a bugfix, write a test that exposes the bug and fails before applying your fix. Submit the test alongside the fix.

When submitting a new feature, add tests that cover the feature.

For testing & debugging the generator please refer to the Yeoman Generator [testing documentation](https://github.com/yeoman/generator/wiki/Testing-generators).

## License

[MIT License](http://en.wikipedia.org/wiki/MIT_License)

[travis-image]: http://img.shields.io/travis/diegonetto/generator-ionic.svg?style=flat
[travis-url]: https://travis-ci.org/diegonetto/generator-ionic
[npm-url]:  https://npmjs.org/package/generator-ionic
[npm-image]: http://img.shields.io/npm/v/generator-ionic.svg?style=flat
[gittip-image]: http://img.shields.io/gittip/diegonetto.svg?style=flat
[gittip-url]: https://www.gittip.com/diegonetto
