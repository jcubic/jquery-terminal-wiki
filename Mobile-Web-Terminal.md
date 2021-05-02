jQuery Terminal works out of the box on mobile. Be sure to not use examples that may hang around somewhere that use onBlur event function and return false. This will prevent the keyboard on mobile from popup because it will keep the terminal in focus and on mobile the terminal has to be disabled on init so you can activate it with a finger tap.

### Responsive size:

You can use this responsive CSS to make the size of the terminal change to make to fit into the screen:
```css
@media screen and (min-width: 1200px) {
    :root {
        --size: 1.4;
    }
}
```

### Responsive text

If you want the text to respond to the size of the terminal, like with default greetings. You can echo a function where you will check the size of the terminal and render proper text. An example is the rendering of [fliglet](https://www.npmjs.com/package/figlet), Where you wrap the text depending on the size of the terminal.

```javascript
term.echo(function() {
   if (this.cols() > 100) { 
      return bigText();
   } else {
      return smallerText();
   }
});
```

You can return a string, an array of strings, or a Promise of them.

You can see how this works in [Pipe demo](https://codepen.io/jcubic/pen/vYLvvXx?editors=0110).



