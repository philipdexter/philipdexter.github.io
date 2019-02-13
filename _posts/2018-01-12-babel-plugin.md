---
layout: post
title: A Short Guide to Creating Babel Plugins
---

[Babel](http://babeljs.io/) is a JavaScript compiler
with plugin support.

Imagine a Babel plugin
that transforms all string literals into
`"babel"`.
The source below does that.

```javascript
module.exports = function () {
  return {
    visitor: {
      StringLiteral: function a(path) {
        path.node.value = 'babel';
      }
    }
  };
};
```

Here's how to use it:

```bash
$ npm install babel babel-cli
$ ./node_modules/.bin/babel --plugins ./a.js
a = 'hi';
^D
a = 'babel';
```

It works with files too:

```bash
$ cat b.js
var s = 'start';

function go() {
  var e = 'end';
  return s + e;
}

console.log(go());
$ ./node_modules/.bin/babel --plugins ./a.js ./b.js
var s = 'babel';

function go() {
  var e = 'babel';
  return s + e;
}

console.log(go());
$ ./node_modules/.bin/babel --plugins ./a.js ./b.js | node
babelbabel
```

Now imagine the same babel plugin that, in addition, transforms every
variable declaration in a function initialized to `'*'` to be
abstracted as a function argument.
The source below does that.

```javascript
module.exports = function (babel) {
  t = babel.types

  return {
    visitor: {
      StringLiteral: function a(path) {
        path.node.value = 'babel';
      },
      VariableDeclaration: function a(path) {
        var var_name = path.node.declarations[0].id.name;
        var var_val = path.node.declarations[0].init.value;
        if (var_val === '*') {
          path.parentPath.parentPath.node.params.push(t.identifier(var_name));
          path.node.declarations[0].init = t.identifier(var_name);
        }
      },
    }
  };
};
```

Here's how to use it:

```bash
$ cat c.js
function go(t) {
  var c = 'hi';
  var a = '*';
  for (var i = 0; i < t; ++i) {
    console.log(c + ' ' + a);
  }
}

go(5, 'stranger')
$ ./node_modules/.bin/babel --plugins ./a.js ./c.js | node
babelbabelbabel
babelbabelbabel
babelbabelbabel
babelbabelbabel
babelbabelbabel
$ ./node_modules/.bin/babel --plugins ./a.js ./c.js
function go(t, a) {
  var c = 'babel';
  var a = a;
  for (var i = 0; i < t; ++i) {
    console.log(c + 'babel' + a);
  }
}

go(5, 'babel');
```
