Animation is new API added in jQuery Terminal 2.28.0. There is new file that you can include in the same way as other files (e.g. with CDN):

```html
<script src="https://cdn.jsdelivr.net/npm/jquery.terminal/js/animation.js"></script>
```

`$.terminal.Animation` abstract class. You can extend this class but you need to define render function.

Example of usage looks like this:

```javascript
class BarAnimation extends $.terminal.Animation {
    constructor(...args) {
        super(...args);
        this._i = 0;
    }
    render(term) {
        if (this._i > term.cols()) {
            this._i = 0;
        } else {
            this._i++;
        }
        return [new Array(this._i).fill('-').join('')];
    }
}
```

The render function needs to return an array of lines (strings) that will be rendered on canvas when echoed:

```javascript
term.echo(new BarAnimation(50)); // 50 frames per second
```

This animation can then be displayed on the terminal. The animation will render a line with dashes that grows to the size of the terminal and then start from the beginning.

Another useful defined class is `FramesAnimation` that you can use to render animation from frames. The first argument needs to be an array of lines (array of arrays of strings).

Here is example of bouncing ball animation:

```javascript
term.echo(new $.terminal.FramesAnimation([
    [
        '  o    ',
        '       ',
        '       ',
        '       '
    ],
    [
        '       ',
        '   o   ',
        '       ',
        '       '
    ],
    [
        '       ',
        '       ',
        '    o  ',
        '       '
    ],
    [
        '       ',
        '       ',
        '       ',
        '     o '
    ],
    [
        '       ',
        '       ',
        '      o',
        '       '
    ],
    [
        '       ',
        '     o ',
        '       ',
        '       '
    ],
    [
        '    o  ',
        '       ',
        '       ',
        '       '
    ],
    [
        '       ',
        '   o   ',
        '       ',
        '       '
    ],
    [
        '       ',
        '       ',
        '  o    ',
        '       '
    ],
    [
        '       ',
        '       ',
        '       ',
        ' o     '
    ],
    [
        '       ',
        '       ',
        'o      ',
        '       '
    ],
    [
        '       ',
        ' o     ',
        '       ',
        '       '
    ]
], 8));
```

You can see those examples on [CodePen](https://codepen.io/jcubic/pen/VweodLO?editors=0010) (the demo also have ASCII camera animation).

## Internals

By default, the animation API uses CanvasRenderer to print the animation. But you can write your own renderer (for instance using WebGL or something else).

The second parameter to `$.terminal.Animation` and the third to `$.terminal.FramesAnimation` is the Renderer class. By default, it's `$.terminal.CanvasRenderer`. If you want to create your own renderer all you have to do is extending `$.terminal.Renderer` (that is abstract class) and implement line and clear methods:

```javascript
class myRenderer extends $.terminal.Renderer {
    clear({width, height}) {
    }
    line(text, x, y) {
    }
}
```

See the source code for details: [animation.js](https://github.com/jcubic/jquery.terminal/blob/master/js/animation.js).

## Style
Even that animation is rendered on a canvas, CSS custom properties (CSS variables) --size, --color, --background still apply.