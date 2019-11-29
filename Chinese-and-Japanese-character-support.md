With special <a href="https://github.com/timoxley/wcwidth">wcwidth</a> library you can have proper text wrapping on terminal with wider characters like those from
Chinese or Japanese. The package is on npm but it don't include the file proper for browsers (no UMD - Universal Module Definition), you can spam the author
on this <a href="https://github.com/timoxley/wcwidth/issues/2">issue on github</a>.

But you can grab the version I've created using browserify, you can get it from jsdelivr (you can also host it yourself):

```html
<script src="https://cdn.jsdelivr.net/gh/jcubic/static/js/wcwidth.js"></script>
```
