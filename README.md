# Getting Started

```
npm init -y
npm install webpack webpack-cli --save-dev
```

Adjust the `package.json` file to ensure the package is `private` and remove the `main` entry.

```json
{
  "name": "webpack-demo",
  "version": "1.0.0",
  "description": "",
  "private": true,
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "MIT",
  "devDependencies": {
    "webpack": "^5.38.1",
    "webpack-cli": "^4.7.2"
  }
}
```

## Creating a Bundle

separate the source code (`./src`) from the distribution code (`./dist`).

To bundle `lodash` install the library locally:

```
npm install --save lodash
```

import into script:

```js
import _ from "losdash";

function component() {
  const element = document.createElement("div");

  element.innerHTML = _.join(["Hello", "webpack"], " ");

  return element;
}

document.body.appendChild(component());
```

Since the scripts are being bundled, modify the `<script>` tag to load bundle, instead of the raw `./src` file:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Getting Started</title>
  </head>
  <body>
    <script src="main.js"></script>
  </body>
</html>
```

Running `npx webpack` takes the script at `src/index.js` as the entry point and generates `dist/main.js` as the output.

## Using a Configuration

Using a config file is useful for complex setups.

`webpack.config.js`

```js
const path = require("path");

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "main.js",
    path: path.resolve(__dirname, "dist"),
  },
};
```

Run `npx webpack` again to build.

```
npx webpack --config webpack.config.js
```

If `webpack.config.js` is present it's used by default.

## NPM Scripts

`package.json`

```json
{
  "scripts": {
    "test": "echo\"Error: no test specified\" && exit 1",
    "build": "webpack"
  }
}
```

Now can use `npm run build` instead of the `npx`

# Asset Management

## Setup

`dist/index.html`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Asset Management</title>
  </head>
  <body>
    <script src="bundle.js"></script>
  </body>
</html>
```

`webpack.config.js`

```js
const path = require("path");

module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
};
```

## Loading CSS

```
npm install --save-dev style-loader css-loader
```

`webpack.config.js`

```js
module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  module: {
    rules: [
      {
        test: /\.css$/i,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
};
```

Can now `import './styles.css'` into the file that depends on that styling.

`src/style.css`

```css
.hello {
  color: red;
}
```

`src/index.js`

```js
import _ from 'lodash';
import './style.css';

function component() {
	...
	element.classList.add('hello');

	return element;
}

document.body.appendChild(component())
```

```
npm run build
```

Should minimize css for better load times. Can also use loaders for postcss, sass, and less.

### sass-loader

```
npm install sass-loader sass --save-dev
```

Chain the `sass-loader` with the css-loader and style-loader to immediately apply all styles to the DOM; or the mini-css-extract-plugin to extract into a separate file.

`app.js`

```js
import "./style.scss";
```

`style.scss`

```scss
$body-color: red;

body {
  color: $body-color;
}
```

`webpack.config.js`

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.s[ac]ss$/i,
        use: [
          // Creates `style` nodes from js strings
          "style-loader",
          // Translates CSS into CommonJS
          "css-loader",
          // Compile Sass to CSS
          "sass-loader",
        ],
      },
    ],
  },
};
```

Then run `webpack`.

Read the [documentation](https://webpack.js.org/loaders/sass-loader/).

## Loading Images

There is a built in Asset Modules

`webpack.config.js`

```js
module.exports = {
	...,
	module: {
		rules: [
			...
			{
				test: /\.(png|svg|jpg|jpeg|gif)$\i,
				type: 'asset/resource',
			}
		]
	}
}
```

Then can `import MyImage from './my-image.png'`.
That will be added to the `output` directory and the `MyImage` will contain the final url of that image.

## Loading Fonts

The Asset Modules takes any file you load through them and outputs it to the build directory.

`webpack.config.js`

```js
module.exports = {
	...
	module: {
		rules: [
			...
			test: /\.(woff|woff2|eot|ttf|otf)$/i,
			type: 'asset/resource',
		]
	}
}
```

Add the font files to the src folder. Then incorporate them via `@font-face`.
THe local `url(...)` will be picked up by webpack.

`src/style.css`

```css
@font-face {
  font-family: "MyFont";
  src: url("./my-font.woff2") format("woff2"), url("./my-font.woff") format("woff");
  font-weight: 600;
  font-style: normal;
}

.hello {
  ...;
  font-family: "MyFont";
}
```

# Output Management

Calling another js script.

`webpack.config.js`

```js
const path = require("path");

module.exports = {
  entry: {
    index: "./src/index.js",
    print: "./src/print.js",
  },
  output: {
    filename: "[name].bundle.js",
    path: path.resovle(__dirname, "dist"),
  },
};
```

Then `npm run build`

## Setting up HtmlWebpackPlugin

```
npm install --save-dev html-webpack-plugin
```

`webpack.config.js`

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  entry: {
    index: "./src/index.js",
    print: "./src/print.js",
  },
  plugins: [
    new HtmlWebpackPlugin({
      title: "Output Management",
    }),
  ],
  output: {
    filename: "[name].bundle.js",
    path: path.resovle(__dirname, "dist"),
    clean: true,
  },
};
```

HtmlWebpackPlugin will generate its own `index.html` file.
use the `output.clean` option to clean the `/dist` folder before each build.

# Development

`webpack.config.js`

```js
...
module.exports = {
	mode: 'development',
	entry: {},
	...
}
```

## Using source maps

makes it easier to track down errors and warnings

`webpack.config.js`

```js
module.exports = {
	mode: 'development',
	entry: {},
	devtool: 'inline-source-map',
	...
}
```

## Development tool

### can use `watch` mode

`package.json`

```json
{
  "scripts": {
    "watch": "webpack --watch"
  }
}
```

### `webpack-dev-server`

```
npm install --save-dev webpack-dev-server
```

`webpack.config.js`

```js
module.exports = {
	...
	devtool: 'inline-source-map',
	devServer: {
		static: './dist'
	}
	...
	optimization: {
		runtimeChunk: 'single',
	}
}
```

Tells `webpack-dev-server` to serve the files from the `dist` directory to `localhost:8080`

Add a script to easily run the dev server

`package.json`

```json
{
	...
	"scripts": {
		...,
		"start": "webpack server --open,
	}
}
```

Can run `npm start` from the cli.
