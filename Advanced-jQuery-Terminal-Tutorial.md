1. [Interpreter](#interpreter)
2. [Displaying the content on the terminal](#displaying-the-content-on-the-terminal)
3. [[Events]]
4. [[Authentication]]
5. [[Webpack and Babel]]
6. [[Shortcuts]]
7. [[Saving State]]
8. [[Mobile Web Terminal]]
9. [[Extensions]]

## Interpreter

The interpreter is the first argument of jQuery plugin invocation:

```javascript
$('body').terminal(<interpreter>, { <options> });
```

The interpreter can be created in few different ways, the first argument is overloaded
and allow passing function, object (including nested object), string, or Array of
function object and strings.

### Function

The function is a more low-level, but versatile version of the interpreter. It accepts the whole
command as a string. It's useful if you want to parse the command yourself. You can
use some interpreter like SQL or JavaScript that passes the whole expression to that interpreter.
You can also parse the commands with tools provided by jQuery Terminal like:

* `$.terminal.parse_command`
* `$.terminal.split_command`

Those two functions are very similar except the second one that converts arguments into values.
`parse_command` will convert numbers like strings into numbers and regex like strings into
regular expression objects. Both functions remove quotes from strings.

The function return object:

```javascript
{
   "name": "command",
   "args": [<arguments>],
   "args_quotes": [<Array of quote characters from string>],
   "rest": "<arguments as string>",
}
```

There are also functions for converting arguments into an array:

* `$.terminal.parse_arguments`
* `$.terminal.split_arguments`

Both pairs of functions handle spaces inside strings, escaped spaces and escaped quotes,
So they return proper parsed output like bash would do it.

Example:


```javascript
$('body').terminal(function(command) {
   var cmd = $.terminal.parse_command(command);
   if (cmd.name === 'add') {
      this.echo(cmd.args.reduce((a, b) => a + b));
   }
});
```

when called with: `add 1 2 3 4 5` it will return `15`.

More about parsing in [[Parsing commands]].

### String

You can use string as an interpreter (also in Array and Object) the string need to be a URL
that points to a valid JSON-RPC service. If the service provide `system.describe` (defined
in [1.1 draft of JSON-RPC](https://www.jsonrpc.org/historical/json-rpc-1-1-wd.html))
you can use auto-completion without any extra work. If you don't want to implement non-standard
`system.describe`, you can provide a normal method (with the same name) that returns
standard JSON-RPC response, but then you will need to use the option that tells jQuery Terminal
where he can find the definition of procedures that are provided by the JSON-RPC service.
To do that you can use:

```
{
   describe: "result.procs"
}
```

if the JSON-RPC method `system.describe` return object `{procs: ...}` as described by JSON-RPC 1.1.

If you don't want to use auto-discovery you can use the option:

```
{
   ignoreSystemDescribe: true
}
```

### Object

The object can have functions that will be executed when a command with a specific name is
invoked, it can be another object that creates a nested interpreter (for example MySQL command,
that accept SQL commands) or it can be a string that creates a nested interpreter
from JSON-RPC service.

```javascript
var stack = [];
$('body').terminal({
  stack: {
    push: function(arg) {
      stack.push(arg);
    },
    pop: function() {
      return stack.pop();
    }
  },
  sudo: function(command, ...args) {
    switch(command) {
      case 'ls':
        list_directory(...args);
        break;
      case 'rm':
        remove_file(...args);
        break;
      case 'mv':
        move_file(...args);
        break;
      default:
        this.error('Sudo what?');
    }
  },
  rpc: "service.py"
}, {
  checkArity: false
});
```

This example creates three commands
* `sudo` that allows executing a command with arguments, variadic functions
  require to add option `checkArity` that is set to true by default and don't allow to execute functions
  with more than the required number of arguments. 
* `stack` that creates simple nested command, when you type stack it will change the prompt to `stack>` and
  allow executing two commands `push` and `pop` that is an interface to stack data structure created from Array.
* `rpc` command will change the prompt to `rpc>` and allow to execute commands that are provided by the given URL
  here it expects that in the same directory as the HTML file there is `service.py` that is a server-side script
  that is the implementation of JSON-RPC.


### Array

With an array, there is always created single object, created by merging explicit object
or string that point to JSON-RPC service that have `system.describe`, that allows creating
object-based on procedures in that request. Besides objects created from Array,
there can only be a single function that is executed as a fallback if no commands
were found. Also if JSON-RPC doesn't support `system.describe` method it's handled like
a single function.


### Completion

If an object is created from at least one object (or JSON-RPC service that provide `system.describe`)
then you can use automatic completion by adding the option

```javascript
{
  completion: true
}
```

Details can be found in [[Tab-completion]] page.

## Displaying the content on the terminal

The main function for displaying the content of the terminal is the `echo` method. It has a lot of advanced functions.

* Display of Columns if you echo an array of strings.
* Echo of Promise or [[Function that can react to the size of the terminal|Mobile-Web-Terminal#responsive-text]].
* [[Formatting and Syntax Highlighting]].
* [[Chinese and Japanese character support]].
* [[Invoking Commands and terminal methods from Server]].
* [[Typing Animation]]
* [[Animation]] new in 2.28.0
* [[Forms]] new in 2.28.0

**TODO:**
* Using ASCII libraries [awesome-ascii](https://github.com/jcubic/awesome-ascii)
* Using Web Worker for ANSI ART rendering (see example [bookmarklet](https://github.com/jcubic/terminal-bookmarks/blob/master/16colors.js))
* [Drawing ANSI Art](https://codepen.io/jcubic/pen/pxdxmN?editors=0010)
* Preprocess ANSI with executable
* renderHandler
  * React inside jQuery Terminal & Canvas, see [Codepen demo](https://codepen.io/jcubic/pen/mddwRzE)