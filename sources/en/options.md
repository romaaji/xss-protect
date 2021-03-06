Customize Filter Rules
========

When using the `xss()` function, the second parameter could be used to specify
custom rules:

```JavaScript
options = {};  // Custom rules
html = xss('<script>alert("xss");</script>', options);
```

To avoid passing `options` every time, you can also do it in a faster way by
creating a `FilterXSS` instance:

```JavaScript
options = {};  // Custom rules
myxss = new xss.FilterXSS(options);
// then apply myxss.process()
html = myxss.process('<script>alert("xss");</script>');
```

Details of parameters in `options` would be described below.

## Customize Whitelist

By specifying a `whiteList`, e.g. `{ 'tagName': [ 'attr-1', 'attr-2' ] }`. Tags
and attributes not in the whitelist would be filter out. For example:

```JavaScript
// only tag a and its attributes href, title, target are allowed
var options = {
  whiteList: {
    a: ['href', 'title', 'target']
  }
};
// With the configuration specified above, the following HTML:
// <a href="#" onclick="hello()"><i>Hello</i></a>
// would become:
// <a href="#">Hello</a>
```

For the default whitelist, please refer `xss.whiteList`.

## Filter out tags not in the whitelist

By using `stripIgnoreTag` parameter:

+ `true` filter out tags not in the whitelist
+ `false`: by default: escape the tag using configured `escape` function

Example:

If `stripIgnoreTag = true` is set, the following code:

```HTML
code:<script>alert(/xss/);</script>
```

would output filtered:

```HTML
code:alert(/xss/);
```

## Filter out tags and tag bodies not in the whitelist

By using `stripIgnoreTagBody` parameter:

+ `false|null|undefined` by default: do nothing
+ `'*'|true`: filter out all tags not in the whitelist
+ `['tag1', 'tag2']`: filter out only specified tags not in the whitelist

Example:

If `stripIgnoreTagBody = ['script']` is set, the following code:

```HTML
code:<script>alert(/xss/);</script>
```

would output filtered:

```HTML
code:
```

## Filter out HTML comments

By using `allowCommentTag` parameter:

+ `true`: do nothing
+ `false` by default: filter out HTML comments

Example:

If `allowCommentTag = false` is set, the following code:

```HTML
code:<!-- something --> END
```

would output filtered:

```HTML
code: END
```

## Customize the handler function for matched tags

By specifying the handler function with `onTag`:

```JavaScript
function onTag (tag, html, options) {
  // tag is the name of current tag, e.g. 'a' for tag <a>
  // html is the HTML of this tag, e.g. '<a>' for tag <a>
  // options is some addition informations:
  //   isWhite    boolean, whether the tag is in whitelist
  //   isClosing  boolean, whether the tag is a closing tag, e.g. true for </a>
  //   position        integer, the position of the tag in output result
  //   sourcePosition  integer, the position of the tag in input HTML source
  // If a string is returned, the current tag would be replaced with the string
  // If return nothing, the default measure would be taken:
  //   If in whitelist: filter attributes using onTagAttr, as described below
  //   If not in whitelist: handle by onIgnoreTag, as described below
}
```

## Customize the handler function for attributes of matched tags

By specifying the handler function with `onTagAttr`:

```JavaScript
function onTagAttr (tag, name, value, isWhiteAttr) {
  // tag is the name of current tag, e.g. 'a' for tag <a>
  // name is the name of current attribute, e.g. 'href' for href="#"
  // isWhiteAttr whether the tag is in whitelist
  // If a string is returned, the attribute would be replaced with the string
  // If return nothing, the default measure would be taken:
  //   If in whitelist: filter the value using safeAttrValue as described below
  //   If not in whitelist: handle by onIgnoreTagAttr, as described below
}
```

## Customize the handler function for tags not in the whitelist

By specifying the handler function with `onIgnoreTag`:

```JavaScript
function onIgnoreTag (tag, html, options) {
  // Parameters are the same with onTag
  // If a string is returned, the tag would be replaced with the string
  // If return nothing, the default measure would be taken (specifies using
  // escape, as described below)
}
```

## Customize the handler function for attributes not in the whitelist

By specifying the handler function with `onIgnoreTagAttr`:

```JavaScript
function onIgnoreTagAttr (tag, name, value, isWhiteAttr) {
  // Parameters are the same with onTagAttr
  // If a string is returned, the value would be replaced with this string
  // If return nothing, then keep default (remove the attribute)
}
```

## Customize escaping function for HTML

By specifying the handler function with `escapeHtml`. Following is the default
function **(Modification is not recommended)**:

```JavaScript
function escapeHtml (html) {
  return html.replace(/</g, '&lt;').replace(/>/g, '&gt;');
}
```

## Customize escaping function for value of attributes

By specifying the handler function with `safeAttrValue`:

```JavaScript
function safeAttrValue (tag, name, value) {
  // Parameters are the same with onTagAttr (without options)
  // Return the value as a string
}
```
