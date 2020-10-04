To import jQuery in webpack you need imports-loader (tested with Webpack 4).

First install jQuery Terminal

```bash
npm install --save jquery.terminal
```

Then install webpack/babel env:
```bash
npm install --save-dev @babel/core @babel/preset-env @babel/preset-env \
                       css-loader style-loader imports-loader webpack \
                       webpack-cli webpack-dev-server
```

create `webpack.config.js` file

```javascript
var path = require('path');

module.exports = {
    entry:  {
        app: path.resolve('./') + '/index.js'
    },
    output: {
        path: path.resolve('./dist'),
        filename: "[name].js"
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                loader: 'babel-loader',
                query: {
                    presets: ['@babel/preset-env']
                }
            },
            {
                test: /\.css$/,
                loaders: ['style-loader', 'css-loader']
            }
        ]
    }
};
```

create `index.html`:

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="app.js"></script>
  </head>
  <body></body>
</html>
```

Then main script:

```javascript
import $ from 'jquery';
import terminal from 'imports-loader?additionalCode=var%20define=false;!jquery.terminal';
terminal(window, $);
// you can also use this
// const $ = terminal(window);
// since terminal function return jQuery instance that got imported
$(function() {
    $('body').terminal((cmd, t) => {
        t.echo('user said ' + cmd);
        // user will be able to enter formatting like type [[;red;]hello]
        // and it will display red text, if you don't want that use
        // $.terminal.escape_brackets to escape the string from user
        // this is important if you enable invokeMethods option
    }, {
        greetings: 'Webpack + jQuery Terminal'
    });
});
```
> Note: if you want fully working terminal you also should import css.

to import extensions like (unix formatting use) best is to use same imports-loader and look the source code, for UMD definition how to call the function.

```javascript
import terminal from 'imports-loader?additionalCode=var%20define=false;!jquery.terminal';
import 'jquery.terminal/css/jquery.terminal.css';
import unix from 'imports-loader?additionalCode=var%20define=false;!jquery.terminal/js/unix_formatting';
const $ = terminal(window);
unix(window, $);
// unix formatting also return jQuery object, so you can stack the functions
// const $ = unix(window, terminal(window));
```

> NOTE: define in import, will force to use AMD in UMD definition that is inside jQuery Terminal,
> and that definition return function where you're passing window object and optional jQuery Instance.

If you want to use Prism with Webpack and Babel it's little bit more work to load the languages, check:

https://github.com/jcubic/terminal-webpack-prism

that have working configuration of webpack with PrismJS.

> NOTE: old syntax `define=>false` no longer work with latest webpack and import-loader.