# transformime

[![Build Status](https://travis-ci.org/nteract/transformime.svg)](https://travis-ci.org/nteract/transformime)

![Optimus MIME](https://cloud.githubusercontent.com/assets/6437976/8895696/db154a04-3397-11e5-91ca-296b957658a6.png)

Transforms MIMEtype+data to HTML Elements

Intended to be used in context of Jupyter and IPython projects, particularly by display areas.

## Installation

```
npm install transformime
```

## Usage

Transformime works in the browser (via browserify), converting a mimebundle (MIME type + data) into HTML Elements. `transformime.transform` returns promises for all the HTMLElements.

### Using the helper function `createTransform`

```javascript
var transformime = require("transformime");

var transform = transformime.createTransform();

transform({
    "text/plain": "Hello, World!",
}).then(function onSuccess(result) {
    mimeType = result.mimetype;
    htmlElement = result.el;
    console.log("Tranformime: Result:", mimeType, htmlElement);
}, function onError(err) {
    console.error("Transformime: Error:", err);
});
```

### Transform a single mimetype

```javascript
> t = new transformime.Transformime();
> p1 = t.transform({'text/html': '<h1>Woo</h1>'}, document)
Promise { <pending> }
> p1
Promise {
  { mimetype: 'text/html',
  el:
   { _events: {},
     _childNodes: [Object],
     _childNodesList: null,
     _ownerDocument: [Object],
     _childrenList: null,
     _version: 1,
     _parentNode: null,
     _memoizedQueries: {},
     _readonly: false,
     _namespaceURI: 'http://www.w3.org/1999/xhtml',
     _prefix: null,
     _localName: 'div',
     _attributes: {},
     _settingCssText: false,
     _style: [Object] } } }
> p1.then(function(result){
...   console.log(result.el.innerHTML);
...   console.log(result.el.textContent);
... });
Promise { <pending> }
> <h1>Woo</h1>
Woo
```

Images get handled as base64 encoded data and become embedded elements.

```javascript
> // Send an image over
> p2 = t.transform({'image/png': 'R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7'}, document)
> p2.then(function(result){
...     console.log(result.el.src);
... })
data:image/png;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7
```

### Transform the richest element
```javascript
> var mimes = {'text/html': '<code>import this</code>', 'text/plain': 'import this'}
> var p3 = t.transform(mimes, document);
> p3.then(function(result){
...    console.log(result.mimetype + ": " + result.el.innerHTML)
... })
text/html: <code>import this</code>
```

### Working with iframes

```javascript
> // Create an arbitrary iframe and slap it in the body of the document
> var iframe = document.createElement("iframe");
> document.querySelector('body').appendChild(iframe);
> var idoc = iframe.contentDocument;
> // Pass the iframe's document as the document context for the created element
> var p5 = t.transform({'text/html': '<h1>mimetic</h1>'}, idoc);
> p5.then(function(result){
... idoc.querySelector('body').appendChild(result.el);
... })
> idoc.querySelector('body').innerHTML
'<div><h1>mimetic</h1></div>'
```

## Development

```
git clone https://github.com/nteract/transformime
cd transformime
npm install
npm run build
```

## API Reference
### Functions

<dl>
<dt><a href="#createTransform">createTransform([transformers], [doc])</a> ⇒ <code>function</code></dt>
<dd><p>Helper to create a function that transforms a MIME bundle into an HTMLElement
using the given document and list of transformers.</p>
</dd>
</dl>

<a name="Transformime"></a>

## Transformime

Transforms mimetypes into HTMLElements

**Kind**: global class  

* [Transformime](#Transformime)
    * [new Transformime(transformers)](#new_Transformime_new)
    * [.transform(bundle, document)](#Transformime+transform) ⇒ <code>Promise.&lt;{mimetype: string, el: HTMLElement}&gt;</code>
    * [.del(mimetype)](#Transformime+del)
    * [.get(mimetype)](#Transformime+get) ⇒ <code>function</code>
    * [.set(mimetype, transformer)](#Transformime+set) ⇒ <code>function</code>
    * [.push(transformer, mimetype)](#Transformime+push) ⇒ <code>function</code>
    * [._proxy(transformer, mimetype)](#Transformime+_proxy) ⇒ <code>function</code>

<a name="new_Transformime_new"></a>

### new Transformime(transformers)
Public constructor


| Param | Type | Description |
| --- | --- | --- |
| transformers | <code>Array.&lt;function()&gt;</code> | list of transformers, in reverse priority order. |

<a name="Transformime+transform"></a>

### transformime.transform(bundle, document) ⇒ <code>Promise.&lt;{mimetype: string, el: HTMLElement}&gt;</code>
Transforms a mime bundle, using the richest available representation,
into an HTMLElement.

**Kind**: instance method of <code>[Transformime](#Transformime)</code>  

| Param | Type | Description |
| --- | --- | --- |
| bundle | <code>any</code> | {mimetype1: data1, mimetype2: data2, ...} |
| document | <code>Document</code> | Any of window.document, iframe.contentDocument |

<a name="Transformime+del"></a>

### transformime.del(mimetype)
Deletes all transformers by mimetype.

**Kind**: instance method of <code>[Transformime](#Transformime)</code>  

| Param | Type | Description |
| --- | --- | --- |
| mimetype | <code>string</code> &#124; <code>Array.&lt;string&gt;</code> | mimetype the data type (e.g. text/plain, text/html, image/png) |

<a name="Transformime+get"></a>

### transformime.get(mimetype) ⇒ <code>function</code>
Gets a transformer matching the mimetype

**Kind**: instance method of <code>[Transformime](#Transformime)</code>  
**Returns**: <code>function</code> - Matching transformer  

| Param | Type | Description |
| --- | --- | --- |
| mimetype | <code>string</code> | the data type (e.g. text/plain, text/html, image/png) |

<a name="Transformime+set"></a>

### transformime.set(mimetype, transformer) ⇒ <code>function</code>
Sets a transformer matching the mimetype

**Kind**: instance method of <code>[Transformime](#Transformime)</code>  
**Returns**: <code>function</code> - inserted transformer function (may be different than arg)  

| Param | Type | Description |
| --- | --- | --- |
| mimetype | <code>string</code> &#124; <code>Array.&lt;string&gt;</code> | the data type (e.g. text/plain, text/html, image/png) |
| transformer | <code>function</code> |  |

<a name="Transformime+push"></a>

### transformime.push(transformer, mimetype) ⇒ <code>function</code>
Appends a transformer to the transformer list.

**Kind**: instance method of <code>[Transformime](#Transformime)</code>  
**Returns**: <code>function</code> - inserted transformer function (may be different than arg)  

| Param | Type |
| --- | --- |
| transformer | <code>function</code> |
| mimetype | <code>string</code> &#124; <code>Array.&lt;string&gt;</code> |

<a name="Transformime+_proxy"></a>

### transformime._proxy(transformer, mimetype) ⇒ <code>function</code>
Create a proxy to a transformer, using another mimetype.

**Kind**: instance method of <code>[Transformime](#Transformime)</code>  
**Returns**: <code>function</code> - transformer  

| Param | Type |
| --- | --- |
| transformer | <code>function</code> |
| mimetype | <code>string</code> &#124; <code>Array.&lt;string&gt;</code> |

<a name="createTransform"></a>

## createTransform([transformers], [doc]) ⇒ <code>function</code>
Helper to create a function that transforms a MIME bundle into an HTMLElement
using the given document and list of transformers.

**Kind**: global function  

| Param | Type | Description |
| --- | --- | --- |
| [transformers] | <code>Array.&lt;function()&gt;</code> | List of transformers, in reverse priority order. |
| [doc] | <code>Document</code> | E.g. window.document, iframe.contentDocument |
