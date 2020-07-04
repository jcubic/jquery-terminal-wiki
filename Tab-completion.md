Tab completion is another useful feature of jQuery Terminal. By default, if you press `<TAB>`, it will be inserted into the terminal (even though it will be replaced by 4 spaces, it will still be treated as one character).

To change the default behavior, you need to use the [completion option](https://terminal.jcubic.pl/api_reference.php#completion).

It you have an object as first argument (interpreter) or a JSON-RPC string with `system.describe`

```javascript
$('#terminal').terminal({
  open: function() {
  },
  close: function() {
  },
  'open-foo': function() {
  }
}, {
  completion: true
});
```

If you type `op` and press `<TAB>`, it will insert `open` then if you press `<TAB>` a second time, it will list `open` and `open-foo`.

The second possibility is if you use a function as first argument (interpreter), then you can use an array:

```javascript
$('#terminal').terminal(function(command) {
    
}, {
  completion: ['pwd', 'pwdhash', 'pwdhash-bin', 'pwdx']
});
```

If you need logic in the completion process: say for instance you have the command `git` and want to complete it into `git clone` and `git checkout`, you can use this:

```javascript
$('#terminal').terminal(function(command) {
    
}, {
  completion: function(string, callback) {
      if (this.get_command().match(/^git /)) {
         callback(['checkout', 'clone', 'pull', 'push', 'commit']);
      } else {
        callback(['ls', 'cat', 'git']);
      }
  }
});
```

Additionally, you can return a promise from completion; you can use `Promise.resolve()`, but what is more useful is if you make the completion on the server:

```javascript
$('body').terminal(function(command) {
    
}, {
  completion: function(string, callback) {
      return fetch('completion.py', {string, command: this.get_command()}).then(res => res.json());
  }
});
```

## Full command completion

if you want to complete part of the word or have different separators like parenthesis. There is a way when you can use full command and return full command as completion. To be able to show just the word you complete you can also change double tab to not show whole commands.

This is example from [LIPS - Scheme based lisp](https://jcubic.github.io/lips/):

```javascript
$('body').terminal(function(command) {

}, {
    completionEscape: false,
    wordAutocomplete: false,
    doubleTab: function(string, maches, echo_command) {
        echo_command();
        this.echo(maches.map(command => {
            return lips.tokenize(command).pop();
        }));
    },
    completion: function(string) {
        let tokens = lips.tokenize(this.before_cursor(), true);
        tokens = tokens.map(token => token.token);
        // Array.from is need to for jQuery terminal version <2.5.0
        // when terminal is outside iframe and lips is inside
        // jQuery Terminal was using instanceof that don't work between iframes
        if (!tokens.length) {
            return Array.from(env.get('env').apply(env).toArray());
        }
        const last = tokens.pop();
        if (last.trim().length) {
            const globals = Object.getOwnPropertyNames(window);
            const prefix = tokens.join('');
            const re = new RegExp('^' + jQuery.terminal.escape_regex(last));
            var commands = env.get('env').apply(env).toArray().concat(globals).filter(name => {
                return re.test(name);
            }).map(name => prefix + name);
            return Array.from(commands);
        }
    }
});
```

You can test it in previous link, when you type `(let ((x 10)) (dis` and if you press tab it will complete `display`. And if you type `(begin (string` and press double tab it will give you all function names that start with "string".