### Table of Contents
1. [Introduction](#user-content-introduction)
2. [Unpkg CDN](#user-content-unpkg-cdn)
3. [Html page](#user-content-html-page)
4. [Initialization](#user-content-initialization)
6. [Accessing Terminal Object](#user-content-accessing-terminal-object)
7. [Creating the interpreter](#user-content-creating-the-interpreter)
8. [What can you echo?](#user-content-what-can-you-echo)
9. [Greetings](#user-content-greetings)
16. [Command line history](#user-content-command-line-history)
19. [Prompt](#user-content-prompt)
20. [Reading text from user](#user-content-reading-text-from-user)
21. [Masking password](#user-content-masking-password)
23. [Key Shortcuts](#user-content-key-shortcuts)
25. [Asynchronous Commands](#user-content-asynchronous-commands)
27. [Executing commands from JavaScript](#user-content-executing-commands-from-javascript)
29. [Updating lines](#user-content-updating-lines)

### Introduction

[jQuery Terminal is a **JavaScript library**](https://terminal.jcubic.pl/) that requires **jQuery** to work properly. To create an app using jQuery Terminal, first, you need to create an HTML page, and then include jQuery and both the JavaScript and CSS files from jQuery Terminal. You can also use **webpack** or other *bundlers* but we will not discuss this here.
In this tutorial, you will learn all the basics needed to create your own Terminal on a webpage and also the explanations for all the features.

You can also read this article: [How to create interactive terminal like website?](https://itnext.io/how-to-create-interactive-terminal-like-website-888bb0972288).

### Unpkg CDN

The best way to get jQuery Terminal is from [unpkg.com](https://unpkg.com/#/) that serves files from [npm repository](https://npmjs.com). With it, you don't need to host the files yourself. It's useful if you want to create a quick proof of concept or if you want to use one of the demo services like Codepen, JSFiddle.

With unpkg.com CDN you can specify that you always want the latest version but not if they have breaking changes. For instance, if the latest version is 1.22.1, you can specify that you want version 1.22 and all the bug fixes using URL:

```
https://unpkg.com/jquery.terminal@1.22.x
```

This URL will redirect to file /js/jquery.terminal.js with the proper version number. If you want to have all the bug fixes for version 1 and all the features, you can use 1.x.x instead of 1.22.x. And if you want a minified version of the library you can specify this URL:

```
https://unpkg.com/jquery.terminal@2.x.x/js/jquery.terminal.min.js
```

It's the same principle for the CSS file, you can use an URL like:

```
https://unpkg.com/jquery.terminal@2.x.x/css/jquery.terminal.min.css
```

### HTML page

A basic HTML page using unpkg.com CDN will look like this

```html
<!DOCTYPE html>
<html>
<head>
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://unpkg.com/jquery.terminal@2.x.x/js/jquery.terminal.min.js"></script>
<link rel="stylesheet" href="https://unpkg.com/jquery.terminal@2.x.x/css/jquery.terminal.min.css"/>
</head>
<body>
</body>
</html>
```

Of course, you can use local files instead by downloading them directly, by installing with npm or bower, or even by cloning the git repository.

### Initialization

To initialize the plugin and create your terminal, you need to invoke the plugin. To do that, you need to select some element of the page, by using a CSS selector with jQuery.

To select an element with a specific ID you can use `$('#terminal')` (you will need to have `<div id="terminal"></div>` in html) or if you want to attach the terminal to `body` (to have full screen terminal) you can use `$('body')`.

If you have your script using

```html
<script>
/* your code */
</script>
```

or

```html
<script src="script.js"></script>
```

inside `<head>`, then you need to wrap your code inside the jQuery Ready function that will be invoked when DOM (Document Object Model - html representation in memory) will be
ready so jQuery will find `#terminal` or `body`. to do that you need this code

```javascript
$(function() {
   $('#terminal');
});
```

If you have your code after the body (or before closing `</body>`) you don't need to wrap it with `$(function() {});`

When you select your jQuery object, you need to invoke terminal plugin:

```javascript
$(function() {
   $('#terminal').terminal();
});
```


### Creating the interpreter

You know how to invoke the plugin; you need now to specify the interpreter (the first argument of the plugin) and define some options; you probably want at least to remove default greetings, which is discussed later in this document.

There are 3 types of interpreters: function, object, and strings.

* **function** is created when you pass the interpreter as a function

```javascript
$(function() {
   $('#terminal').terminal(function(command) {
    
   });
});
```

* an **object**

```javascript
$(function() {
   $('#terminal').terminal({
      open: function(value) {
         
      },
      sub: {
         get: function(arg) {
         }
      }
      close: function(arg1, arg2) {
      }
   });
});
```

This will create a terminal with three commands: open, close, and sub. If you type `open something` the open function will be invoked and the value parameter will equal something.

The terminal will parse all your commands, convert number-like strings into numbers and Regular Expressions into RegExp objects. You can disable this behavior using
[processArguments](https://terminal.jcubic.pl/api_reference.php#processarguments) option. This option can be a *boolean* and you can set it to `false` or it can be a function where you
can process the value in your own way (in this function for instance you can use JSON.parse to get objects out of strings).

By default, if you have an object like this and you type `close foo`, the terminal will throw an error because you didn't specify a second argument.

To prevent this behavior you can use the option [checkArity](https://terminal.jcubic.pl/api_reference.php#checkarity) set to `false`.

In this example, you have `sub` which is not a function but another object; when you type `sub` in the terminal the prompt will change to `sub> ` and you will be able to type `get`, but not `open` and `close`. They will not work.

This is called a nested interpreter, which can also be created when you have a function as an interpreter and by using the [push method](https://terminal.jcubic.pl/api_reference.php#push).

*  the third option is to use a string that points to a [JSON-RPC service](https://en.wikipedia.org/wiki/JSON-RPC) URL; if the service is on the same domain, you can use it as is, but if it's on a different domain or port then you need to use a mechanism called [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing).

```javascript
$(function() {
   $('#terminal').terminal("service.py");
});
```

* There is also one more type of interpreter which is an array that can combine all the above types but with some limitations:

You can have only one function that is executed when no other command is found. You can also have one single JSON-RPC service without `system.describe` (or if you turn it off using [describe option](https://terminal.jcubic.pl/api_reference.php#describe)) which was part of the historical 1.1 version of JSON-RPC specification.

It's a special method that should return a different object than normal JSON-RPC, the spec for the object is in [system.describe section in API reference page](https://terminal.jcubic.pl/api_reference.php#system.describe). JSON-RPC is converted to a simple function while JSON-RPC with `system.describe` is converted to object.

If you want to have automatic completion for JSON-RPC and only want normal JSON-RPC methods you can specify option [describe](https://terminal.jcubic.pl/api_reference.php#describe) as a string with dot-separated values that point to property procs from the spec, which should be an array of objects with names and params. (`describe: "result"` will work if proc array is normal JSON-RPC response).

Here is how the array should look like:

```javascript
[
   {"name":"echo", "params":["string"]}
]
```

Here how it looks to invoke the terminal with an array interpreter:

```javascript
$(function() {
   $('#terminal').terminal([
      {
         open: function() {
         },
      },
      "service.py",
      function(command) {
          this.echo("You've typed " + command);
      }
   ]);
});
```

### Accessing Terminal Object

Terminal objects have useful methods that you can invoke; the whole list of methods is in the [API documentation](https://terminal.jcubic.pl/api_reference.php). To access this object, which is a standard jQuery object with additional methods, you can use:

* the object value returned by the plugin:

```javascript
var term = $('#terminal').terminal(function(command) {
    
});
term.echo('foo');
```

* the `this` keyword in every function (if you can't you can report a bug).
```javascript
$('#terminal').terminal(function(command) {
    this.echo('foo');
});
```

* the third option is to specify the Terminal object as the last parameter of some functions (it should be in all of them). It is a legacy feature to not break the API.

```javascript
$('#terminal').terminal(function(command, term) {
    term.echo('foo');
});
```

This is useful, for example, when you use ES5 and `setTimeout` and want to call `echo` after a delay (with ES6 you can use the *arrow* function).

### What can you echo?

The `echo` method of the terminal instance is the most common function that you will use with the terminal, is quite versatile; you can use `echo` to print stuff on the terminal, you can call it with the following:
* String
* Array of strings (it will print the array as columns)
* Any other value that can have toString method (so also numbers or boolean that are boxed in Number object or any custom object that have toString method).
* Promise that resolves to any of the above

A limitation is that you can't call `echo` with an array of other values, only an array of strings. And JavaScript objects are printed as `[object Object]` (because this what's returned by the toString method), but this may change and new printing objects may be introduced in the future.

From version 2.9.0 new feature was added to allow echo DOM nodes and jQuery Objects. So you can use:

```javascript
var div = $('<p>Hello <strong>World</strong></p>')
term.echo(div);
```

You can also echo any HTML element like canvas or video. And with the new API in 2.9.0 you can echo anything if you add `renderHandler`. (TODO add a link to API or create Wiki Page, for now, you can read example at [this Codepen demo](https://codepen.io/jcubic/pen/mddwRzE)). 

Check also [[Formatting and Syntax Highlighting]] for details about the internal syntax of echo colors, basic styles, images, and links.

### Greetings

The second argument to the terminal plugin is an object where you can put your options. The "greetings" is one of them. It is the first text that is echoed into the terminal.

You also can use `term.echo` to achieve the same effect but to get the same behavior as with greetings, you will
need to put it in the `onInit` event because greetings are shown after login. More on `login` and `onInit` in later sections.

```javascript
$(function() {
   $('#terminal').terminal("service.py", {
      greetings: "Welcome to Python Service"
   });
});
```

Greetings option also accept function as argument. It need to be function with one argument which is setGreeting function (so it's a callback function), you can use it like this:

```javascript
$(function() {
   $('#terminal').terminal("service.py", {
      greetings: function(callback) {
         var text = "Welcome to Python Service";
         callback(this.cols() > text.length ? text : 'Welcome');
   });
});
```

You can use a callback to call greetings at any time (e.g. as a response to server call).

NOTE: if you want to create an ASCII logo as greetings, you will need to escape special characters. If you create string with double quotes "", you need to escape double quote `"` and slash `\` with `\"` and `\\`. The same with strings with ``` ` ``` and `'`, you will need to escape those with slashes. You can also use library filget.js that render ASCII where you don't need to worry about escaping. Here is [Figlet.js Demo](https://codepen.io/jcubic/pen/VwvEvmN?editors=0010).

From version 2.10.0 you can return both string and promise that resolves to string from greetings. (promise was added in 2.9.0 and bug with returning string was fixed in 2.10.0).

### Command line history

Each time you press `<Enter>`, the command is saved in the history (you can disable this by using the history option). History is saved in **localStorage** or with **cookies**.

You can move through history commands using the up and down arrows.

If you have more than one terminal on the page. Or more than one application with the terminal on the same origin (protocol+domain+port) you can share the command line history or make each terminal have a different history. To make terminals share history you just need to give them the same `name` (using an option) or do nothing because by default each terminal will have the same name. But if you want each terminal to have different history you just give them different names.

You can filter which commands should go to history and which don't use the `historyFilter` option. It can be a regular expression that, if match,  will not add the command to history (you can use regex /^\s/ to not store commands that begin with space like in bash). You can also specify a function where you can use some logic and return true or false.

You can also limit the number of commands in history using the `historySize` option; if you set it to `false`, it will not restrict the size (but it may fill up your **localStorage** or the **cookies**).

If you want the history to persist only during the current session, you can use *in memory* history, instead of localStorage or cookies, by using the memory option set to true.

But this will make all the data actually saved to memory (the same will happen with all data required by terminal like authentication token or interpreter names).

### Prompt

The prompt is the string at the beginning of the command line, you can specify the prompt using:
* prompt option
* if you have a nested interpreter created from a nested object
* if you use push to create a new interpreter
* if call term.set_prompt method

the prompt can be a string (it can have formatting) or function with a single callback as an argument that should be called with a new prompt.

```javascript
$(function() {
   $('#terminal').terminal("service.py", {
      prompt: "[[;red;]>>> ]"
   });
});

term.set_prompt(function(set_prompt) {
    if (this.get_token()) {
        set_prompt(this.login_name() + '@server$ ');
   } else {
        set_prompt('guest@server$ ');
   }
});
```

this will display different prompts to login in to users and normal users, about authentication later.

As with any terminal function, you can also return promise from prompt function:

```javascript
$('#terminal').terminal("service.py", {
    prompt: () => new Promise((resolve => setTimeout(() => resolve('>>> '), 400)))
});
```

### Reading text from user

If you're in interpreter function and want to ask user for input and then process it (like in C scanf) you can use function read:

```javascript
$('#terminal').terminal({
   name: function(name) {
      this.read('last name: ', last_name => {
         this.echo('Your name is ' + name + ' ' + last_name);
      });
   }
});
```

read function also return a promise so you can use this code instead:

```javascript
$('#terminal').terminal({
   name: function(name) {
      this.read('last name: ').then(last_name => this.echo('Your name is ' + name + ' ' + last_name));
   }
});
```

This will change the prompt to last name: and wait until the user press enter, then it will resolve the promise, so you will see text on the screen with
your name.

Unfortunately, you can't return the promise and then return the string, because when you return a promise terminal will be paused, so you will not
see last name: prompt.

Another option to read text from users is to use a nested interpreter using push where you can have logic and ask for more than one thing.
This is the code that will work the sam as read:

```javascript
$('#terminal').terminal({
    name: function(name) {
        this.push(function(last_name) {
            if (last_name) {
                this.echo('Your name is ' + name + ' ' + last_name).pop();
            }
        }, {
            prompt: 'last name: '
        });
    }
});
```

But there are differences, by default read will disable history with push you can have a history if you want but you still can disable it using
history `this.cmd().history().disable();` cmd is another plugin that is used internally but you can also use it on your own, it have simplified API.
Here is [demo if simple terminal created using cmd plugin](https://codepen.io/jcubic/pen/XaoqGp). The other difference is, that the read will invoke callback and resolve promise with empty last name if the user press enter (and doesn't type anything), here it will only work if last_name is not empty.

### Masking password

If you want to read the password from the user you can use the method set mask. It can be any string and each character you type will be replaced with this
character, you can also use an empty string to have a password like in some Unix commands like `mysql`.

```javascript
$('#terminal').terminal(function() {
    this.set_mask('-').read('pass: ').then(pass => this.echo("Your password is " + pass));
});
```

this will set the mask to `-` character if you use true it will us asterisk.

### Key Shortcuts

The best way to add key shortcuts is to use keymap option which accept object with shortcut as strings and function that will be invoked on this shortcut:

```javascript
$(function() {
    $('#terminal').terminal({}, {
        keymap: {
            'CTRL+P': function(e, original) {
                 this.echo('my shortcut');
            }
       }
    });
});
```

the original argument is original function from terminal if set, for instance you can change behavior of enter key using:

```javascript
$(function() {
    var count = 0;
    $('#terminal').terminal({}, {
        keymap: {
            'ENTER': function(e, original) {
                 original();
                 this.set_prompt(this.get_prompt().replace(/[0-9]*> /, ++count + '> '));
            }
       }
    });
});
```

NOTE: this is just an example of `ENTER` to have this behavior it's better to create a prompt as a function.

this will change the prompt with a counter for each enter key.

The order of mnemonics looks like this CTRL+META+SHIFT+ALT and at the beginning is special modifier HOLD. The shortcut will be invoked when you hold the key.

### Asynchronous Commands

If you have Async commands, which mean commands that execute later in time you should pause the terminal and resume it after you've done processing. This will ensure
that your command will be invoked properly (mostly because of one feature which is invoking commands from URL hash or using exec method, discussed later in this document).
If you don't care about the order of commands you don't need to pause the terminal.

To pause the terminal and don't accept input from a user, you can use `pause()` method and to resume the state you can use `resume()` method. You can also return a promise
from the interpreter, but the resolved value will be printed on the terminal, to avoid this you can resolve to undefined (or add empty then if you don't use Promise constructor).

```javascript
$('#terminal').terminal(function(command, term) {
    return new Promise(function(resolve) {
        setTimeout(function() {
            term.echo('delay command ' + command);
            resolve();
        }, 1000);
    });
});

$('#terminal').terminal(function(command) {
    this.pause();
    setTimeout(() => {
        this.echo('delay command ' + command).resume();
    }, 1000);
});
```

You can also use any function that return a promise like fetch:

```javascript
$('#terminal').terminal(function(command, term) {
    return fetch('/server').then(function(res) {
        return res.text();
    });
    // or ES6 shortcut
    // return fetch('/server').then(r => r.text());
});
```

Async/await also works.

```javascript
$('#terminal').terminal({
    get: async function(path) {
        let res = await fetch(path);
        return res.text();
    }
});
```

### Executing commands from JavaScript

There are few different types of executing commands, you can use method `term.exec` to invoke different command:

```javascript
$('#terminal').terminal({
    open: function() {
        this.echo('you try to open').exec('close');
    },
    close: function() {
        this.echo('you closed');
    }
});
```

if you type open you will see:

```
> open
you try to open
> close
you closed
```

If you don't want that `> close` to be echoed (by default exec act like if you type that command and press enter) you can call:

```javascript
term.exec('close', true);
```
You can also put the array in exec:
```javascript
term.exec([
   'git clone https://github.com/jcubic/jquery.terminal.git',
   'cd jquery.terminal',
   'git checkout devel'
]);
```

and it will invoke commands in order, so if you have async commands it's important to have pause/resume or to returning promises from the interpreter.

The second type of exec is invoking commands using URL, if you saved some commands in URL hash and if you refresh the browser, all the commands will be
executed in order the same as exec with an array. To see this in action check [404 error page](https://terminal.jcubic.pl/404) and type: `record start` and `wikipedia terminal emulator`, it will create a URL hash that store your commands, and after you refresh it will execute your second command. With this, you can save the whole session but be careful because URLs have a limited number of characters. You can write something and share the link. One of the examples on the website show [this url](https://tinyurl.com/ybbydgtf). That prints the definitions from Hacker (also known as [Jargon File](https://en.wikipedia.org/wiki/Jargon_File)). See [[Saving State]] to check how to enable saving commands in a hash.

The last type of exec is invoking terminal shortcuts, you can do that with `term.invoke_key()` method. 
```javascript
term.invoke_key("CTRL+R");
```

this will invoke control + R key which by default is bind to [reverse search](/jcubic/jquery.terminal/wiki/Reverse-history-search). This will invoke any shortcut added by the keymap option as well as
build in keymap options.

You can also execute commands from the server see [[Invoking-Commands-and-terminal-methods-from-Server]].

### Updating lines

You can also update the lines that get echo to the terminal using the update method.

```javascript
term.update(0, "hello world")
```

this will replace the greetings, you can also put negative values into the first argument. To update the last line you can use:

```javascript
term.echo("hello");
term.update(-1, "hello world");
```

there is also a function that returns an index of the last command which is `term.last_index`. This function is better if you need to update the last line
and you know that it's last so you can use last_index and you need to call update later when there is already some text added after.

```javascript
term.update(term.last_index(), "hello")
```

this will be the same as -1.

If you want to remove last line you can use `term.remove_line(-1)` which is just alias for `term.update(-1, null);`


You can Read about more advanced stuff in [[Advanced jQuery Terminal Tutorial]].
