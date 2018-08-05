## Create Adobe-CEP extension
### Adobe-CEP with React, Material-UI, Native Node modules, Webpack, Babel and ExtendScript

<img src="kap.gif"/>

this Adobe-CEP extension creator bootstraps for creating Adobe CC extensions easily with
modern web technologies and with native node.js modules for session logic
and with support for extendscript (host app). It is built in a semi opinionated
way so you can focus on writing your great extensions.

#### how to build
- `npm run build:dev` / `npm run build:prod` - will build into `./dist` folder
- `npm run deploy:dev` / `npm run deploy:prod` - will deploy `./dist` folder into the extension folder.
if in dev mode, it will create a **symbolic link**, otherwise it will copy the entire folder.
- `npm run release:dev` / `npm run release:prod` - will build and deploy

the output is a `./dist` extension folder
```
dist
  index.html
  .debug
  CSXS/
    manifest.xml
  icons/
    favicon.ico    
  node_modules/
  host/
    index.js
  client-dist/
    bundle.js
    main.css
  session-dist/
    bundle.js
  host/
  libs/
    CSInterface.js
```

#### how to customize
start with `./pluginrc.js`, this is the plugin config I created, here is an example
```javascript
module.exports = {
    extensionBundleId: 'com.hendrix.ps2dl',
    extensionBundleName: 'ps2dl',
    panelName: 'hendrix demo',
    root: __dirname,
    sourceFolder: path.join(__dirname, "src"),
    destinationFolder: path.join(__dirname, "dist")
}
```
when build is happening, then the build will pickup your package id and panel name
and other configurations from this file and will use it against a template that will
generate the `./dist/CSXS/manifest.xml` and `.debug` (in dev mode) file for you.

feel free to modify the contents of the `assets` folder for you own need.

### what does this include ?
this bootstrap is composed of three parts

#### Front end side
inside `src/client-src` you have the entry point for creating ReactJS application.
installing modules is against the project root, see `/project.json`.  
a nice feature, that it has is that you can use `webpack-dev-server` to see
your UI results with watching at the browser, simply use:
- `npm start` or
- `npm run client:dev-server`
this will generate the template html for you and run it in your browser,
and will rebuild on code changes which is nice to have.

#### back end side / session (using node modules)
inside `src/session-src` you have the entry point for using native node.js modules.
Adobe-CEP supports instantiating Node.js runtime as well as Chromium but I believe
most developers would like to use the power of Node.js for doing IO.

Adobe-CEP does provide it's native IO for disk access and also using Chromium
you can use the browser `Fetch` api, but it can lead to very bad code structuring.

I inject the `session` object in the `window` object and therefore it is accessible
even from the front end side.  
Also notice, that this folder has it's own `node-modules` separate from the root folder,
this is because, they are built differently from the Front end side.

using node modules can enhance the functionality.

#### ExtendScript side
inside `src/host`, you will put you `jsx` files, by default I will load `index.jsx`,
but I highly advise to use the session to load a `jsx` file dynamically so it can pick
up it's `#include` dependencies otherwise it won't (this is a known issue)

#### Webpack side
so why am I using **Webpack** ?  
without webpack, you will have to require modules by absolute path, which is not nice,
also I wanted to enjoy a better ES6 syntax.

why are there separate **Webpack** configs for `client` and `session`?  
very good question. It boils down to the following fact, the client/front-end side, uses
pure web technologies and can be bundeled with all of it's dependencies and it has a classic
`web` target for webpack.  
the `session` side uses native node.js modules and has a `node` target in it's config, they
are not to be mixed together or else subtle configuration will not work. this is equivalent
to other projects using `electron`, also, it is not advisable to bundle native node.js modules,
this is not efficient.

#### Build scripts
inside `/build-scripts`, you will find the webpack configs and also the build and deploy
scripts. They use no-fancy node modules to keep things simple (no libs like Gulp).

you can find:  
- `/build-script/build.js development/production` this will build the entire thing
- `/build-script/deploy.js development/production` this will deploy the entire thing into
the adobe extensions folder in debug mode currently, I still need to sign the extension

#### FAQ
**Q:** how do I add more web development modules (like redux) ?  
**A:** simply `npm install redux` from the root `./` directory  

**Q:** how do I add more session native node modules (like fs-extra) ?  
**A:** simply `npm install fs-extra` from the `./src/session-src` directory, when building occurs, these
modules will be copied to the `./dist` folder.  

**Q:** how do I add some js lib without npm ?  
**A:** simply edit `./src/index.html`  

**Q:** how do I add some extendscript files ?  
**A:** you must add them to `./src/host/` folder and then you have two choices. one, is to edit
`./assets/CSXS/manifest.xml` file to declare them, or load them at runtime dynamically (better, you can read
    about it more later)

#### TODO
- add plugin certification
- do some files cleanup
