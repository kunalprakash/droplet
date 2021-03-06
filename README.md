Droplet Editor
=================

[![Build Status](https://travis-ci.org/dabbler0/droplet.svg?branch=master)](https://travis-ci.org/dabbler0/droplet)

Droplet seeks to re-envision "block programming" as "text editing". It is useful as a transitional tool for beginners using languages like Scratch, and is a go-to text editor for everyone on mobile devices (where keyboards don't work so well).

How to Embed
------------
To embed, call new droplet.Editor() on a div.

```html
<html>
<head>
<style>
#editor {
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
}
</style>
</head>
<div id="editor">
</div>
</html>
```

```coffeescript
require ['droplet'], (droplet) ->
  editor = new droplet.Editor document.getElementById('editor'), {
    mode: 'coffeescript'
    palette: [
     {
        name: 'Palette category'
        color: 'blue'
        blocks: [
          {
            block: "for [1..3]\n  ``"
            title: "Repeat some code"
          }
        ]
      }
    ]
  }

  editor.setValue '''
  for i in [1..10]
    document.write 'hello world'
  '''
```

Contributing
============

Droplet uses Grunt and npm to build. Run:

```shell
git pull https://github.com/dabbler0/droplet.git
cd droplet
npm install
grunt all
```

When developing, run:
```shell
grunt testserver
```

This will run the development server and watch the `src/` and `example/` directories for recompilation. Visit `localhost:8000/example/example.html` for a simple running environment. A view debugger is available at `localhost:8000/example/test.html`.

Run `grunt all` to run the tests.

Adding a Language
-----------------
Make a CoffeeScript (or JavaScript) file that looks like this:

```coffeescript
define ['droplet-helper', 'droplet-parser'], (helper, parser) ->
  class MyParser extends parser.Parser
    markRoot: ->

  parser.makeParser MyParser

  return MyParser
```

Put it in `src/myparser.coffee`. Add it to the build system in `requirejs-paths.json`:

```javascript
{
  // etc...
  "droplet-myparser": "myparser" // meaning "myparser.coffee" -- or whatever you named your file
  // etc...
}
```

Require it from the controller:

```coffeescript
define ['droplet-helper',
    'droplet-coffee',
    'droplet-javascript',
    'droplet-myparser', # This is us!
    'droplet-draw' # etc, etc..
    # ...
  ], (helper,
  coffee,
  javascript,
  myparser, # Us again!
  draw, # etc, etc..
  # ...
  ) ->
```

Add a mode option:

```coffeescript
  if mode is 'coffeescript'
    @mode = coffeescript
  else if mode is 'javascript'
    @mode = javascript
  else if mode is 'myparser'
    @mode = myparser
```

Then grunt. Your mode is integrated!

To have your parser actually put blocks in, you will need to do some things in the `markRoot` function. Fields and methods you need to know about:
```coffeescript
# Get the raw text passed into the parser:
@text

# Add a Block
@addBlock({
  # Configure the location of the block (all required)
  bounds: {
    start: {line: Number, column: Number} # Lines and columns are zero-indexed
    end: {line: Number, column: Number}
  }
  depth: Number # Depth in the tree

  # Configure the block you're about to add (all optional)
  color: '#HEXCOLOR'
  precedence: Number
  socketLevel: Number # See "acceptance levels"
  classes: [] # Array of strings.
})

# Add a Socket
@addSocket({
  # Configure the location of the socket (all required)
  bounds: {
    start: {line: Number, column: Number} # Lines and columns are zero-indexed
    end: {line: Number, column: Number}
  }
  depth: Number # Depth in the tree

  # Configure the block you're about to add (all optional)
  precedence: Number
  accepts: {'string': Number} # Maps class names (from block 'classes' array) to an acceptance level
                              # (see "acceptance levels")
})

# Add an Indent
@addIndent({
  # Configure the location of the socket (all required)
  bounds: {
    start: {line: Number, column: Number} # Lines and columns are zero-indexed
    end: {line: Number, column: Number}
  }
  depth: Number # Depth in the tree

  # Configure the indent you're about to add (all optional)
  prefix: '  ' # String that is a prefix of all the lines
})
```

Call these in markRoot to insert Blocks, Sockets and Indents.

You may also want to override the following callbacks:
```coffeescript
# Parens is called whenever a block is dropped into
# another block; you are allowed to change the leading
# and trailing text of the block at this moment (for parentheses, semicolons, etc.)
#
# The default for this is based on the precedence numbers.
MyParser.parens = (leading, trailing, node, context) ->
  # "leading" is the leading text owned by the block and not its children;
  # "trailing" is similar trailing text. "node" is the Block that is being dropped,
  # and context is the Socket or Indent it is being dropped into.
  return [newLeading, newTrailing]

# Text to fill in an empty socket when switching modes:
MyParser.empty = "blarg"
```

Acceptance Levels
-----------------
Acceptance levels determine whether blocks are allowed to be dropped into other blocks (type-checking). This API is subject to change.

Blocks are annotated with a `socketLevel`, which dictates where they are on the scale of "statement only" to "value only"

```coffeescript
helper.BLOCK_ONLY
helper.MOSTLY_BLOCK
helper.ANY_DROP
helper.MOSTLY_VALUE
helper.VALUE_ONLY
```

Blocks are also annotated with a `classes` array, an array of arbitrary string flags.

Sockets are annotated with an `accepts` dictionary, which maps classes to acceptance levels:
```coffeescript
helper.FORBID # Do not allow this block to be dropped no matter what
helper.NORMAL # Allow this block to be dropped it if it is a value
helper.ENCOURAGE_ALL # Allow this block to be dropped no matter what
helper.DISCOURAGE_ALL # Alow this block to be dropped, but only if the user presses the shift key
```
