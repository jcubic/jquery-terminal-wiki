You can save two types of states:

1. You can save state of the whole terminal including prompt, cursor position, lines on terminal or your interpreter.
to save state you can use function
```javascript
var view = term.export_view(); 
```

it will return object that can be use to restore the state of the terminal:

```javascript
term.import_view(view); 
```

NOTE: The object that is returned by export_view will not work with JSON.stringify unless you have a way to serialize functions. But it
will work with few limitations if you strip out functions.

2. Another type of state is history of commands you type in single session that is saved in url hash (text after # that is used to jump
to location with same id in html).

To start saving commands in hash you can invoke function:

```javascript
term.history_state(true);
```

Saving to hash is controlled by option historyState so you can set it to true but be careful because there is limit of URL length.

You can also add command to hash using

```javascript
term.save_state('foo');
```

If you refresh the browser and have option execHash set to true it will execute all commands in order. And you can share the link with commands
to your app.

But there is limitation different browsers have different max URL limit, so best is only save few commands. Setting option `historyState: true`. Also can make your app break because of this limit. So best is to have some command that toggle saving in hash using `term.history_state` function.