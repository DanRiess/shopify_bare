 ## Getting Started ##
 **Why?**: This was built as a starter theme to bring modern build tools into Shopify development since Shopfiy Slate has been deprecated. 
 This starter theme is as stripped down as possible so you can start with a blank canvas. Enjoy.

 **Pre-requisites**: Need to have [Shopify Theme Kit](https://shopify.github.io/themekit/), [Node/NPM](https://nodejs.org/), [Gulp Cli](https://gulpjs.com/docs/en/getting-started/quick-start) installed.
 
 ### Configuration ###
After cloning repo then `cd project-name` and run command `npm install`.
This will install all of the build dependencies.

 Go to `dist/` folder and place your *`config.yml`* with the live theme and development theme (live should come after development):
```
#development config
development:
  theme_id: [development_theme_id]
  password: [password]
  store: [store_url]

#staging config
staging:
  theme_id: [staging_theme_id]
  password: [password]
  store: [store_url]

#live/ production
live:
  theme_id: [live_theme_id]
  password: [password]
  store: [store_url]
  read-only: true

```
  Once you have the *`config.yml`* setup you can run `npm run dev` inside the projects root. This will start development on your dev theme by opening a preview link, starting themekit to watch for changes, and also start watching for changes in your *`src/`* folder.

### Commands ###
  List of commands you can use with npm:
 - `npm run dev` opens development theme preview link, start watching src files, and starts theme kit watch
 - `npm run dev:deploy` deploy your development theme
 - `npm run dev:watch` watch development theme,
 - `npm run dev:open` open preview link for development theme
 - `npm run dev:download` downlaod all development theme files to `/dist`
 - `npm run stage:deploy` deploys to development theme
 - `npm run stage` builds, deploys then opens preview to staging theme
 - `npm run stage:open` opens preview to staging theme
 - `npm run stage:deploy` deploys to staging theme
 - `npm run stage:download` downloads all staging files to `/dist`
 - `npm run live:deploy` deploy to live (first must chage read only to fals - in `config.yml`
 - `npm run live:download` download all live theme files to `/dist`
 - `npm run build` generates production ready css and js
 - `npm run build:dev` generates css and js with no minifier etc...
 - `npm run build:staging` generates production ready css and js (same as npm run build)
 - `npm run build:css` generates development css
 - `npm run build:js` generates development js
 - `npm run build:img` compresses images in /src/images and places in assets
 - `npm run watch:src` auto generates development code as you make changes to `/src` scss and js
 - `npm run watch:dist` start theme kit watch on `/dist` files to update theme on shopify
 - `npm run log:purgedCSS` genrates unused css selectors that were purged 

 ### Project Structure ###
 ```
 ├─ node_modules/
 ├─ dist/
 │   ├─ assets/                   - built css and js files  go here
 │   ├─ config.yml                - where shopify's config file gets placed
 │   └─ ...                       - where we edit liquid files
 ├─  src/                         - this src folder where we edit our js and sass files
 │    ├─ images/                  - put images here for compression
 │    │   ├─ placeholder.png      
 │    │   └─ ...                  - other images
 │    ├─ js/
 │    │   ├─ modules/             - place to put javascript modules
 │    │   ├─ main.js              - main js files used across site
 │    │   └─ ...                  - other pages of site js go here ie.e cart.js
 │    ├─ sass/ 
 │    │   ├─ modules/             - place to put scss modules
 │    │   │   └─ _headers.scss    - example of partials starts with _
 │    │   ├─ styles.scss          - main styles used across the site
 │    │   ├─ product.scss         - example of template page sass
 │    │   └─ ...     
 │    └─ tmp/                     - used for temp output from gulp, i.e. rejected css
 │    │   ├─ styles.rejected.css  - shows all selectors that were purged from styles.scss
 │    │   └─ ...                  - other purged files       
 │  
 ├─ gulpfile.js                   - use this to edit build pipelines if needed
 └─  ... 
 ```
### Rollup ###
 [Rollup](https://rollupjs.org/guide/en/) is used to bundle [ES6 javascript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) within other files and use in the browser. This also performs tree shaking so it's only imports what we use.
 We can import form node_modules but usually tree shaking doesn't work as it used commonJS. You can read the difference [here](https://sazzer.github.io/blog/2015/05/12/Javascript-modules-ES5-vs-ES6/). Rollup also has other plugins we can add as needed.
 Rollup is configured in `*gulpfile.js`*

### PurgeCSS ###

 [PurgeCSS](https://purgecss.com/) looks at the **.liquid** files in the *`/dist`* folder as well as at **.js** files in *`/src/js/`* folder. It will look for any css selectors and not strip those out of the generated css. That means that any css selectors not in the **.js** or **.liquid** files (in their respective directories) will be removed from the css automatically upon each generation of that file. To see what selectors are being stipped we can run `npm run log:purgedCSS`. This will generated the related css files in *`/src/tmp/`*. 

 The way PurgeCSS parses js files is by making each word a selector. This is helpfull because it gets all css that needs to stay in stylesheets but sometimes can leave css that isn't being used (though this seems rare). 

 **Example:** If we had no paragrpah tags in the .liquid files but had a pragraph as a selector for some styles i.e. ` p { color: red}`. We would want this removed because it isn't in use, but if we have a variable in js all named p (i.e. `var p = 2;`) it may not remove it from the css. 

### *Caveats ###
I wanted to address a few things I have noticed thus far. 

  1. **FIXED** First, seems to be something between Theme Kit's Watch and Rollup. I'm using gulp-better-rollup as a wrapper around rollup to make it easier to use with gulp. With this, I have noticed that if I make a change in a JS dependency (ex. a module being imported in main.js or antoher entry file) the files get built but Theme Kit Watch doesn't catch it. I have to go into main.js and hit save to trigger Theme Kit to catch the changes. 

  Fixed by using gulp-touch-custom to edit modify file data to current time when build is run. 

2. This one isn't really on the same level as the above but something I would like to mention. At this, I'm not watching the .liquid files with Gulp. This means JS and CSS builds don't get triggered when you only make a change to a *`.liquid`* file. Most of the time this is not an issue. The only issue I have noticed is if a CSS selector or class is purged and then you add it to a .liquid file to use then you have to trigger a CSS rebuild. The other solution is to have gulp rebuild CSS and possibly JS src files upon each *`.liquid`* file change. If you want to do this go to line 118 in *`gulpfile.js`* and change it to this `watch(config.deepSass, config.liquid, series('build:css'));`

### *Note ###
 I got the shopify theme files (*`/dist/`*) from antoher repo but it looked a bit old. I tried to cleanup the files and include all the minimum required files that shopify needs.
 You can replaced the entire contents of the *`/dist`* folder with any shopify starter theme. Just make sure it's the sam estructure(i.e. *`/dist/assets`*, *`/dist/snippets`*,  etc...).Please let me know if I missing any files or base code that is needed. 
 
 P.S. I currently don't have the *`layout/checkout.liquid`* since that's for *Shopify Plus+* only.