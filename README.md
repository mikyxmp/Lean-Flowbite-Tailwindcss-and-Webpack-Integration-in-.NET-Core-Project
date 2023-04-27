# Lean-Flowbite-Tailwindcss-and-Webpack-Integration-in-.NET-Core-Project
Lean Flowbite, Tailwindcss, and Webpack Integration in .NET Core Project in 5 steps

Goal: have a complete but lean local compilation of all css and js needed to run on tailwindcss and flowbite, including custom css classes and js customization by extension.

## Minimal project solution

Assume you have a .net core project roughly in the shape of:

<pre>
YourSolution
|- YourProject
  |- wwwroot
  |- Areas
  |- ...
  |- Views
  |...
</pre>

## Step 1: organize wwwroot
In wwwroot, delete Bootstrap libraries. End up with this:

<pre>
YourSolution
|- YourProject
  |- wwwroot
    |- css
      |- output.css
    |- js
      |- bundle.js
</pre>

## Step 2: create source folder for css and js:

<pre>
YourSolution
|- YourProject
  |- wwwroot
  |- Areas
  |- ...
  |- Views
  |- src //this one is new!
    |-input.css //css source
    |-index.js //js source
</pre>

## Step 3: create config files

<pre>
YourSolution
|- YourProject
  |- wwwroot
  |- Areas
  |- …
  |- Views
- package.json
- purgecss.config.js
- tailwind.config.js
- postcss.config.js
- webpack.config.js
- gulpfile.js
</pre>

### Step 3a: package.json
All dependencies needed. Also requires node.js installation.

<pre>
{
  "name": "yourProject", 
  "version": "1.0.0",
  "description": "YourProject",
  "scripts": {
    "css:build": "npx tailwindcss -i ./src/input.css -o ./wwwroot/css/output.css --verbose",
    "clean:css": "rimraf ./wwwroot/css/output.css"
  },
  "dependencies": {
    "@tailwindcss/forms": "^0.5.3",
    "@tailwindcss/typography": "^0.5.9",
    "amqplib": "^0.10.3",
    "create-npm": "^1.3.2",
    "flowbite": "^1.6.4",
    "flowbite-datepicker": "^1.2.2",
    "jquery": "^3.6.4",
    "next-auth": "^4.20.1",
    "rimraf": "^3.0.2",
    "vinyl-sourcemaps-apply": "^0.2.1"
  },
  "devDependencies": {
    "autoprefixer": "^9.8.8",
    "eslint": "^8.4.0",
    "gulp": "^4.0.2",
    "gulp-purgecss": "^5.0.0",
    "postcss": "^8.4.21",
    "postcss-url": "^10.1.3",
    "purgecss": "^5.0.0",
    "tailwind": "^4.0.0",
    "tailwindcss": "^3.2.7",
    "webpack": "^5.76.1",
    "webpack-cli": "^5.0.1"
  },
  "main": "tailwind.config.js",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/you/YourProject.git"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/you/YourProject/issues"
  },
  "homepage": "https://github.com/you/YourProject#readme"
}
</pre>

### Step 3b: purgecss.config.js
Defines an array that includes all css and js files to be read for css class identification and purging. Not included in the array means not considered.
The other config classes inherit this content array.

<pre>
module.exports = [
    ".//*.cshtml",
    "./src//.{html,js}",
    "./src/index.js",
    "./src/input.css",
    "./Views/**/.cshtml",
    "./Areas/Admin/**/*.cshtml"
];
</pre>

### Step 3c: tailwind.config.js
Configuring TailwindCSS, extending themes. Settings may vary depending on your tailwindcss, flowbite modules, and other preferences, e.g. fonts.

<pre>
const colors = require('tailwindcss/colors')
const contentArray = require('./purgecss.config');

module.exports = {
    darkMode: 'class',
    plugins: [
        // ...
        require('@tailwindcss/forms'),
        require('@tailwindcss/typography'),
        require('flowbite/plugin')
    ],
content: contentArray,
theme: {
    extend: {
        colors: {
            primary: { "50": "#eff6ff", "100": "#dbeafe", "200": "#bfdbfe", "300": "#93c5fd", "400": "#60a5fa", "500": "#3b82f6", "600": "#2563eb", "700": "#1d4ed8", "800": "#1e40af", "900": "#1e3a8a" }
        }
    },
    fontFamily: {
        'body': [                
            '-apple-system',
            'BlinkMacSystemFont',
            'Segoe UI',
            'Roboto',
            'Helvetica Neue',
            'Arial',
            'Noto Sans',
            'sans-serif',
            'Apple Color Emoji',
            'Segoe UI Emoji',
            'Segoe UI Symbol',
            'Noto Color Emoji'
        ],
        'sans': [                
            '-apple-system',
            'BlinkMacSystemFont',
            'Segoe UI',
            'Roboto',
            'Helvetica Neue',
            'Arial',
            'Noto Sans',
            'sans-serif',
            'Apple Color Emoji',
            'Segoe UI Emoji',
            'Segoe UI Symbol',
            'Noto Color Emoji'
        ],
        'serif': [
            'Noto Serif',
            'ui-serif',
            'Georgia'
        ]
    }
},
variants: {
    extend: {
        brightness: ['dark'],
    },
},
}
</pre>

### Step 3d: postcss.config.js
PostCSS, Tailwind, plugin config.

<pre>
const tailwindConfig = require('./tailwind.config');

module.exports = {
    plugins: {
        tailwindcss: {},
        autoprefixer: {},
        typography: {}
    },
    mode: 'jit',
    content: tailwindConfig.content
};
</pre>

### Step 3e: webpack.config.js
Bundles all javascript of /src/index.js to one bundle.js file.

<pre>
const path = require('path');

module.exports = {
    mode: 'development',
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'wwwroot/js')
    }
};
</pre>

### Step 3f: gulpfile.js
Purges unnecessary css

<pre>
const { src, dest } = require('gulp');
const purgecss = require('gulp-purgecss');
const contentArray = require('./purgecss.config');

function purge() {
    return src('wwwroot/css/output.css') 
        .pipe(
            purgecss({
                content: contentArray,
                safelist: ['safelist-class-1', 'safelist-class-2'],
            })
        )
        .pipe(dest('wwwroot/css'));
}

exports.purge = purge;
</pre>

## Step 4: what about input.css and index.js?

### Step 4a: input.css

Make sure to import base, components, utilities you need. This all ends up in output.css

<pre>
@tailwind base;
@tailwind components;
@tailwind utilities;

//example custom class
.my-custom-shadow {
        box-shadow: rgba(50, 50, 93, 0.25) 0px 13px 27px -5px, rgba(0, 0, 0, 0.3) 0px 8px 16px -8px;
    }
</pre>

### Step 4b: index.css

Same principle. Place for importing js libraries and flowbite, tailwindcss, js customization.

<pre>
import Flowbite from 'flowbite';
import Datepicker from 'flowbite-datepicker/Datepicker'; //example
</pre>

## Step 5: Run it all during 'Rebuild'
In your project file

<pre>
|- YourProject
</pre>

after

<pre>
&lt;/PropertyGroup&gt;
</pre>

add:

<pre>
&lt;ItemGroup&gt;
	&lt;UpToDateCheckBuilt Include="./src/input.css" Set="Css" /&gt;
	&lt;UpToDateCheckBuilt Include="postcss.config.js" Set="Css" /&gt;
&lt;/ItemGroup&gt;
&lt;ItemGroup&gt;
	&lt;Content Include="gulpfile.js" /&gt;		
  &lt;Content Include="src\index.js" /&gt;
	&lt;Content Include="src\input.css" /&gt;
&lt;/ItemGroup&gt;
&lt;Target Name="Tailwind" BeforeTargets="Build"&gt;
  &lt;Exec Command="npm run css:build" /&gt;
&lt;/Target&gt;
&lt;Target Name="PostBuild" AfterTargets="PostBuildEvent"&gt;
	&lt;Exec Command="webpack" /&gt;
&lt;/Target&gt;
&lt;Target Name="PurgeCSS" AfterTargets="Publish"&gt;
	&lt;Exec Command="gulp purge" /&gt;
&lt;/Target&gt;
&lt;ItemGroup&gt;
	&lt;None Include="wwwroot\css\output.css" /&gt;	
&lt;/ItemGroup&gt;
</pre>

Adjust the Before/After Targets as needed.
Note: remove/exclude the target-entries before publishing, as they may cause problems.
