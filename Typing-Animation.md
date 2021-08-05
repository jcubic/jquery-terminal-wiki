Typing was first added in version 2.24.0 but in current form, with all updates and fixes, you should use at least 2.28.0 to use all the features.

## Higher-level API

### Prompt animation

```javascript
term.set_prompt("name: ", { typing: true, delay: 200 });
```

### echo animation

```javascript
term.echo("Hello", { typing: true, delay: 200 });
```

## Low-level API

Low-level API is single method `terminal::typing`:

```javascript
term.typing(<type>, <delay>, <string>, options);
```

* `terminal::typing` returns a promise that resolves when the animation ends.
* there is only one option, `finalize`
* type is one of the strings: `'prompt'`, `'echo'` or `'enter'`.
* **prompt** animation as the name suggests is an animation of the prompt and at the end, the prompt is set to the given string.
* **echo** animation as the name suggests animate the string that is echo into the terminal.
* **enter** animation is an animation of entering the command and pressing enter, it uses the current prompt as a prefix after the animation starts. Enter animation doesn't have a higher-level counterpart.

To see examples of typing animation see [Trinity Matrix Nmap sequence animation](https://codepen.io/jcubic/pen/eYWybyM) demo.