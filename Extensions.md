### Less command

If you want to have *less* like command, you can use an additional file:

```
https://unpkg.com/jquery.terminal@1.x.x/js/less.js
```

This add a new jQuery plugin that can be invoked on Terminal instance, which is also a jQuery Object.

```javascript
$('body').terminal({
    less: function(file) {
       $.get(file, (text) => this.less(text));
    }
});
```

This example shows how to invoke `less` with any syntax highlighting you have. If you don't want to have [formatting](/jcubic/jquery.terminal/wiki/Formatting-and-Syntax-Highlighting) but only for less, then you should not use the function `syntax` but use the function `$.terminal.prism`:

```javascript
$('body').terminal({
    less: function(file) {
       var language = {html: 'website': js: 'javascript', css: 'css'};
       var ext = file.match(/\.([^.]+)$/)[1];
       $.get(file, (text) => this.less($.terminal.prism(language[ext], text)));
    }
});
```

## Animation
See [[Animation]]

## TODO

### Prism
### Echo without newline
### Pipe
### Autocomplete menu
### Emoji

On systems that don't support emoji on the system level (like Windows 10).

### Dterm
### Unix Formatting
### XML Formatting
