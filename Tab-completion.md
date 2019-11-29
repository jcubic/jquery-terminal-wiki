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