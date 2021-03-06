### Formatting

Formatting the output of the `echo` method uses a special syntax that can be used to format your text; you can make your text bold, italic or underlined, or even glow. Because it uses the special syntax you don't need to worry about [XSS Vulnerability](https://en.wikipedia.org/wiki/Cross-site_scripting).

Basic example:

```javascript
term.echo('[[b;red;white]hello world]');
```

This will create a red bold text on a white background.

More about the [syntax of Terminal formatting in API reference](https://terminal.jcubic.pl/api_reference.php#echo).

If you need to echo brackets, the best way is to use the `$.terminal.escape_brackets` function.

### Links

To render links you can just echo URL or using formatting syntax:

```javascript
term.echo('[[!;;;;https://github.com/jcubic/jquery.terminal]visit repo]');
```

### Images

To render images you can use the same formatting syntax using `@` character as the first style option in brackets:

```javascript
term.echo('[[@;;;;https://placekitten.com/300/300]]');
```

Images handing was added in version 2.8.0, with more updates in 2.9.0.

### Syntax highlighting

If you want to define *syntax highlighting* for instance taken from *python* or *SQL*, there is the possibility to not have to specify this formatting on your own but use the [PrismJS library](https://prismjs.com/). This will also work while you type. To use *PrismJS*, you first need to include the files needed for the library:

```
https://cdn.jsdelivr.net/npm/prismjs/prism.js
https://cdn.jsdelivr.net/npm/prismjs/themes/prism.css
```

by default *PrismJS* only includes the HTML, CSS, and JavaScript syntax. If you need others like *python* or *SQL*, you need to include one of the [component files](https://unpkg.com/prismjs/components/); they are also on *npm* and on [GitHub](https://github.com/PrismJS/prism).

After you have included the *PrismJS* files, you also need to include the jQuery Terminal Prism wrapper:

```
https://cdn.jsdelivr.net/npm/jquery.terminal/js/prism.js
```

Finally, you can call the `syntax` function somewhere after the terminal and prismJS files have been "imported". It doesn't need to be in `$(function() {});`. To get python highlighting, as an example:

```javascript
$.terminal.syntax('python')
```

There is one additional *syntax highlighting* added by the *Terminal prism wrapper* which is for the website to highlight HTML, CSS, and javascript. This is mainly for CSS and js that is defined in the *style* parameter for CSS and inside `<script>...</script> for javascript.

```javascript
$.terminal.syntax('website');
```

Syntax highlighters are also useful if you want to have a command like *cat* or *less* where you show the content of the files.

### ANSI Escape codes

ANSI escape codes are a way to format text most commonly seen in Unix terminals. The formatting looks like this:

```
\x1b[32mHello\x1b[m
```

This will display the text **Hello** in *green*. `1B` is the hexadecimal for 27 which is the escape key.

Explanation of the [ANSI ESCAPE codes can be found on Wikipedia](https://en.wikipedia.org/wiki/ANSI_escape_code).

To use ANSI formatting in jQuery Terminal, you only need to include one file:

```
https://cdn.jsdelivr.net/npm/jquery.terminal/js/unix_formatting.js
```

The file also handles what's called overtyping (which is used, for example, in the output of the man command on Linux/Unix),  where you have text like this
```
A\x08AB\x08BC\x08C
```

which will display **ABC** in bold text. `\x08` is the code for the backspace key; so you simply write two times the same character to make it **bold**; if you use `A\x08_` you will get underline instead. Backspace (`\x08` characters) should work the same as in the Linux terminal.

From version 2.27.0 (2.27.1) the library added an executable utility that allows converting ANSI formatting and ANSI Art files:

```bash
npm install -g jquery.terminal
```

```bash
LC_ALL=C MAN_KEEP_FORMATTING=1 man -P cat man | from-ansi > man.txt
```

This will save the output of the man command and save it into a file.

If you have ANSI art file (you can get those from [16colo.rs](https://16colo.rs/)). You can generate jQuery Terminal formatting file with this command:

```bash
from-ansi -i file.ans -o file.jqt -a
```

to see help use:

```
from-ansi --help
```

or

```
from-ansi
```

### Custom Syntax highlighting

You can also create your own formatters (this is how terminal's prism.js works). The low-level mechanism for formatters consists of arrays:

```javascript
$.terminal.defaults.formatters
```

which can be arrays of functions or arrays with regular expressions and strings.

In functions, you can do any text replacement and return strings with the specific Terminal formatting markup.

To add a new formatter, you can push value to this array, overwrite it with a new array or use the helper function `$.terminal.new_formatter`. This function will inject your formatter, before any nested formatting formatter,  that is the default formatter; it allows you to have nested formatting like in HTML `[[;red;]foo [[;blue;]bar] baz]`.

If you want that your nested formatters inherit style example:

`[[b;;]foo [[i;;]bar] baz]` and you want that bar needs to inherit bold style, then you should use this code to enable that:

```javascript
$.terminal.nested_formatting.__inherit__ = true;
```

See [Inherit Demo](https://codepen.io/jcubic/pen/MWybXZX?editors=0010).

If you use the `new_formatter` function, your formatter also can have nesting.

For instance, if you want the text *hello* to be red and the text *world* to be green, you can use this formatter:

```javascript
$.terminal.new_formatter(function(string) {
    return string.replace(/hello/g, '[[;red;]hello]').replace(/world/g, '[[;green;]world]');
});
```

You can also put two arrays:

```javascript
$.terminal.new_formatter([/hello/g, '[[;red;]hello]']);
$.terminal.new_formatter([/world/g, '[[;green;]world]']);
```

Note: you should use *regular expressions* with `g` to be sure to replace all instances of the string. If your string or regex is without `g`, it will replace only the first instance of the string.

Here is an example to add an underline formatter, that looks like this `_hello world_ _this is underline_`.

```javascript
$.terminal.new_formatter([/_([^_]+)_/g, '[[u;;]$1]']);
```

Because formatters are in the `$.terminal` namespace, they are globals for the whole page, so you can't have two terminals on the page and each one have different formatters (yes you can have multiple terminals on one page).

This may change in the future, but it will be a breaking change so it could be in the next major version. If you're using a 1.x.x version, from unpkg.com, you will not be affected.

NOTE: if you need to create formatter that changes the length of the string (like replacing *foo* with *hello world*) then you need to use the special function [$.terminal.tracking_replace](https://terminal.jcubic.pl/api_reference.php#tracking_replace). It will track the cursor position after replacement, which was mostly created by [Stack Overflow user T.J. Crowder](https://stackoverflow.com/a/46756077/387194).

The cursor movement will be a little weird because the cursor will be at the beginning of the resulting string and then at the end of it while you move the virtual cursor on the original string.
So you will not be able to remove any character in the input string.

To have a correct cursor position, without the above limitations, you need to use the function like this:

```javascript
$.terminal.new_formatter(function(string, position) {
    return $.terminal.tracking_replace(string, /hello/g, '[[;red;]hello world]', position);
});
```

This will be handled for you if you use a simple array. But this is the only option if you need to put some kind of logic in the replacement.

The following code will be roughly the same as above:

```javascript
$.terminal.new_formatter([/hello/g, '[[;red;]hello world]']);
```

Note: one more limitation of formatters is that they are executed on strings between formatting so the above formatter will not create an infinite loop of replacement. With the exception of formatter like `$.terminal.nested_formatting` which is always added to the list when including the library.

If you want to have formatters based on interpreters, like for instance, a *mysql* command that have *SQL* syntax and a *js* command to have *javascript* formatting, etc... then you can create a stack of formatters (stack is the kind of the data structure [more on Wikipedia](https://en.wikipedia.org/wiki/Stack_(abstract_data_type))). An example of how to do that is in [example section on a website](https://terminal.jcubic.pl/examples.php#syntax_highlight).

if you want to see an example of a more advanced formatter you can check [xml_formatting.js](https://github.com/jcubic/jquery.terminal/blob/master/js/xml_formatting.js) that add handy HTML like tags to color the text. It uses a stack data structure to handle nesting (like in HTML).

### Attribute formatter

jQuery terminal formatting also supports setting specific attributes using JSON format.
Below is a formatting that allows printing big font text:

```javascript
$.terminal.defaults.allowedAttributes.push('style');

$.terminal.new_formatter([
    /<big>(.*?)<\/big>/g, '[[;;;;;{"style": "--size: 1.5;letter-spacing: 2px"}]$1]'
]);
```

And this allows using handy XML like format:

```javascript
term.echo('<big>Hello World</big>');
```

Of course you can use formatting directly:

```javascript
term.echo('[[;;;;;{"style": "--size: 1.5;letter-spacing: 2px"}]Hello World]');
```

But using your own formatting is much cleaner and your code looks much better. Especially if you will use this syntax a lot.

By default safe attributes are only `['title', /^aria-/, 'id', /^data-/]`, you can use explicit string as value.
Or using regular expressions.

### Image XML formatter

Another example can be links and images:
```javascript
$.terminal.new_formatter([
    /<img>(.*?)<\/img>/g, '[[@;;;;$1]]'
]);

term.echo('<img>https://placekitten.com/200/287</img>');
```

You can also use formatting that looks similar to HTML:

```javascript
$.terminal.new_formatter([
    /<img src="([^"]+)"(?: alt="([^"]+)")?\/?>/g, '[[@;;;;$1]$2]'
]);

term.echo('<img src="https://placekitten.com/200/287" alt="This is kitten"/>');
```

### Link XML Formatter

Another handy formatter is formatting for creating links:

```javascript
$.terminal.new_formatter([
    /<a href="([^"]+)">([^>]+)<\/a>/g, '[[!;;;;$1]$2]'
]);

term.echo('<a href="https://terminal.jcubic.pl">jQuery Terminal</a>')
```

**NOTE:** if you want to have formatted that will allow you to combine formatters (example `big`, `img` and `a`) the best way is to create a single formatter that will parse this XML like format. Look at XML formatter as example [xml_formatting.js](https://github.com/jcubic/jquery.terminal/blob/master/js/xml_formatting.js), that allow to use XML syntax for colors.

### Extension XML Formatter

In version 2.28.0 xml_formatter was updated to allow to use better XML like syntax. Available tags (except CSS colors) are:

```xml
<large size="<num>">text</large>
<wide spacing="<num>.px">text</wide>
<img src="<URL>" alt="text"/>
<bold>text</bold>
<overline>text</overline>
<strike>text</strike>
<underline>text</underline>
<glow>text</glow>
<italic>text</italic>
<link href="<URL>">text</link>
```

There are also aliases: `b` for bold, `i` for italic, `a` for link, `big` for large.

See [Codepen demo](https://codepen.io/jcubic/pen/poPZGqe?editors=0110) how it can be used.

### Formatting targets

In version 2.18.0 jQuery Terminal allows picking where formatters should be applied:

There are 3 formatting targets **echo** **command** and **prompt**. By default, formatters is applied everywhere but you can
control that.

When using regex formatter you can control where formatter should be used:

```javascript
$.terminal.new_formatter([
    /<img src="([^"]+)"(?: alt="([^"]+)")?\/?>/g, '[[@;;;;$1]$2]', { echo: true }
]);
```

If an object is not set it will apply formatting everywhere, but if you pass the object you need to specify which targets you want
to support. The above formatter for images will only work in echo.

With the function the target is passed as an option parameter:

```javascript
$.terminal.new_formatter(function(string, options) {
   if (options.echo) {
      return $.terminal.tracking_replace(string, /foo/, 'bar');
   }
   return string;
});
```