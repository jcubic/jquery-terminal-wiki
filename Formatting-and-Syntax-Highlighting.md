### Formatting

Formatting the output of the `echo` method use a special syntax that can be used to format your text; you can make your text bold, italic or underlined or even glow.

```javascript
term.echo('[[b;red;white]hello world]');
```

This example will create a red bold text on a white background.

More about the [syntax of Terminal formatting in API reference](https://terminal.jcubic.pl/api_reference.php#echo).

If you need to echo brackets, the best way is to use the `$.terminal.escape_brackets` function.

### Syntax highlighting

If you want to define *syntax highlighting* for instance taken from *python* or *SQL*, there is the possibility to not have to specify this formatting on your own but use the [PrismJS library](https://prismjs.com/). This will also work while you type. To use *PrismJS*, you first need to include the files needed for the library:

```
https://unpkg.com/prismjs/prism.js
https://unpkg.com/prismjs/themes/prism.css
```

by default *PrismJS* only includes the HTML, CSS and JavaScript syntax. If you need others like *python* or *SQL*, you need to include one of the [component files](https://unpkg.com/prismjs/components/); they are also on *npm* and on [GitHub](https://github.com/PrismJS/prism).

After you have included the *PrismJS* files, you laso need to include the jQuery Terminal Prism wrapper:

```
https://unpkg.com/jquery.terminal@1.x.x/js/prism.js
```

Finally, you can call the `syntax` function somewhere after the terminal and prismJS files have been "imported". It doesn't need to be in `$(function() {});`. To get python highlighting, as an example:

```javascript
$.terminal.syntax('python')
```

There is one additional *syntax highliting* added by the *Terminal prism wrapper* which is for website to highlight HTML, CSS and javascript. This is mainly for CSS and js that is defined in the *style* parameter for CSS and inside `<script>...</script> for javascript.

Syntax highlighters are also useful if you want to have command like *cat* or *less* where

### Custom Syntax highlighting

You can also create your own formatters (this is how terminal's prism.js works). The low level mechanism for formatters consists of arrays:

```javascript
$.terminal.defaults.formatters
```

which can be arrays of functions or arrays with regular expression and strings.

In functions, you can do any text replacement and return strings with the specific Terminal formatting markup.

To add a new formatter, you can push value to this array, overwrite it with a new array or use the helper function `$.terminal.new_formatter`. This function will inject your formatter, before any nested formatting formatter,  that is the default formatter; it allows you to have nested formatting like in HTML `[[;red;]foo [[;blue;]bar] baz]`

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

Because formatters are in the `$.terminal` namespace, they are globals for the whole page, so you can't have two terminals on the page and each one have different formatters (yes you can have multiple terminals on on page).

This may change in the future, but it will be a breaking change so it could be in the next major version. If you're using a 1.x.x version, from unpkg.com, you will not be affected.

NOTE: if you need to create formatter that change the length of the string (like replacing *foo* with *hello world*) then you need to use the special function [$.terminal.tracking_replace](https://terminal.jcubic.pl/api_reference.php#tracking_replace). It will track the cursor position after replacement, which was mostly created by [Stack Overflow user T.J. Crowder](https://stackoverflow.com/a/46756077/387194).

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

Note: one more limitation of formatters is that they are executed on strings between formatting so above formatter will not create infinite loop of replacement. With the exception of formatter like `$.terminal.nested_formatting` which is always added to the list when including the library.

If you want to have formatters based on interpreters, like for instance, a *mysql* command that have *SQL* syntax and a *js* command to have *javascript* formatting etc... then you can create a stack of formatters (stack is the kind of the data structure [more on Wikipedia](https://en.wikipedia.org/wiki/Stack_(abstract_data_type))). Example how to do that is in [example section on a web site](https://terminal.jcubic.pl/examples.php#syntax_highlight).
