---
title: "A Tour of the dart:html Library"
description: Learn about the dart:html library and APIs.
permalink: /guides/html-library-tour
---
<?code-excerpt path-base="examples/html"?>

Use the [dart:html][] library to
program the browser, manipulate objects and elements in the DOM, and
access HTML5 APIs. DOM stands for *Document Object Model*, which
describes the hierarchy of an HTML page.

Other common uses of dart:html are manipulating styles (*CSS*), getting
data using HTTP requests, and exchanging data using
[WebSockets](#sending-and-receiving-real-time-data-with-websockets).
HTML5 (and dart:html) has many
additional APIs that this section doesn’t cover. Only web apps can use
dart:html, not command-line apps.

<div class="alert alert-info" markdown="1">
**Note:**
For a higher level approach to web app UIs, see
[AngularDart]({{site.webdev}}/angular).
</div>

To use the HTML library in your web app, import dart:html:

{% comment %}
TODO(chalin): Figure out why we get ERRORs when using prettify
instead of ```.

TODO(kathyw): Consider helping users run these examples in DartPad.
{% endcomment %}

<?code-excerpt "lib/html.dart (import)"?>
{% prettify dart %}
  import 'dart:html';
{% endprettify %}

## Manipulating the DOM

To use the DOM, you need to know about *windows*, *documents*,
*elements*, and *nodes*.

A [Window][] object represents
the actual window of the web browser. Each Window has a Document object,
which points to the document that's currently loaded. The Window object
also has accessors to various APIs such as IndexedDB (for storing data),
requestAnimationFrame (for animations), and more. In tabbed browsers,
each tab has its own Window object.

With the [Document][] object, you can create and manipulate [Element][] objects
within the document. Note that the document itself is an element and can be
manipulated.

The DOM models a tree of
[Nodes.][Nodes] These nodes are often
elements, but they can also be attributes, text, comments, and other DOM
types. Except for the root node, which has no parent, each node in the
DOM has one parent and might have many children.

### Finding elements

To manipulate an element, you first need an object that represents it.
You can get this object using a query.

Find one or more elements using the top-level functions
`querySelector()` and `querySelectorAll()`. You can query by ID, class, tag, name, or
any combination of these. The [CSS Selector Specification
guide](http://www.w3.org/TR/css3-selectors/) defines the formats of the
selectors such as using a \# prefix to specify IDs and a period (.) for
classes.

The `querySelector()` function returns the first element that matches
the selector, while `querySelectorAll()`returns a collection of elements
that match the selector.

<?code-excerpt "lib/html.dart (querySelector)"?>
{% prettify dart %}
  // Find an element by id (an-id).
  Element elem1 = querySelector('#an-id');

  // Find an element by class (a-class).
  Element elem2 = querySelector('.a-class');

  // Find all elements by tag (<div>).
  List<Element> elems1 = querySelectorAll('div');

  // Find all text inputs.
  List<Element> elems2 = querySelectorAll(
    'input[type="text"]',
  );

  // Find all elements with the CSS class 'class'
  // inside of a <p> that is inside an element with
  // the ID 'id'.
  List<Element> elems3 = querySelectorAll('#id p.class');
{% endprettify %}

### Manipulating elements

You can use properties to change the state of an element. Node and its
subtype Element define the properties that all elements have. For
example, all elements have `classes`, `hidden`, `id`, `style`, and
`title` properties that you can use to set state. Subclasses of Element
define additional properties, such as the `href` property of
[AnchorElement.][AnchorElement]

Consider this example of specifying an anchor element in HTML:

<?code-excerpt "test/html_test.dart (anchor-html)" replace="/.*'(.*?)'.*/$1/g"?>
{% prettify html %}
  <a id="example" href="http://example.com">link text</a>
{% endprettify %}

This \<a\> tag specifies an element with an `href` attribute and a text
node (accessible via a `text` property) that contains the string
“linktext”. To change the URL that the link goes to, you can use
AnchorElement’s `href` property:

<?code-excerpt "test/html_test.dart (href)"?>
{% prettify dart %}
  var anchor = querySelector('#example') as AnchorElement;
  anchor.href = 'http://dartlang.org';
{% endprettify %}

Often you need to set properties on multiple elements. For example, the
following code sets the `hidden` property of all elements that have a
class of “mac”, “win”, or “linux”. Setting the `hidden` property to true
has the same effect as adding `display:none` to the CSS.

<?code-excerpt "test/html_test.dart (os-html)" replace="/.*? = '''|''';$//g"?>
{% prettify dart %}
  <!-- In HTML: -->
  <p>
    <span class="linux">Words for Linux</span>
    <span class="macos">Words for Mac</span>
    <span class="windows">Words for Windows</span>
  </p>
{% endprettify %}

<?code-excerpt "test/html_test.dart (os)"?>
{% prettify dart %}
  // In Dart:
  final osList = ['macos', 'windows', 'linux'];
  final userOs = determineUserOs();

  // For each possible OS...
  for (var os in osList) {
    // Matches user OS?
    bool shouldShow = (os == userOs);

    // Find all elements with class=os. For example, if
    // os == 'windows', call querySelectorAll('.windows')
    // to find all elements with the class "windows".
    // Note that '.$os' uses string interpolation.
    for (var elem in querySelectorAll('.$os')) {
      elem.hidden = !shouldShow; // Show or hide.
    }
  }
{% endprettify %}

When the right property isn’t available or convenient, you can use
Element’s `attributes` property. This property is a
`Map<String, String>`, where the keys are attribute names. For a list of
attribute names and their meanings, see the [MDN Attributes
page.](https://developer.mozilla.org/en/HTML/Attributes) Here’s an
example of setting an attribute’s value:

<?code-excerpt "lib/html.dart (attributes)"?>
{% prettify dart %}
  elem.attributes['someAttribute'] = 'someValue';
{% endprettify %}

### Creating elements

You can add to existing HTML pages by creating new elements and
attaching them to the DOM. Here’s an example of creating a paragraph
(\<p\>) element:

<?code-excerpt "lib/html.dart (creating-elements)"?>
{% prettify dart %}
  var elem = ParagraphElement();
  elem.text = 'Creating is easy!';
{% endprettify %}

You can also create an element by parsing HTML text. Any child elements
are also parsed and created.

<?code-excerpt "lib/html.dart (creating-from-html)"?>
{% prettify dart %}
  var elem2 = Element.html(
    '<p>Creating <em>is</em> easy!</p>',
  );
{% endprettify %}

Note that `elem2` is a `ParagraphElement` in the preceding example.

Attach the newly created element to the document by assigning a parent
to the element. You can add an element to any existing element’s
children. In the following example, `body` is an element, and its child
elements are accessible (as a List\<Element\>) from the `children`
property.

<?code-excerpt "lib/html.dart (body-children-add)"?>
{% prettify dart %}
  document.body.children.add(elem2);
{% endprettify %}

### Adding, replacing, and removing nodes

Recall that elements are just a kind of node. You can find all the
children of a node using the `nodes` property of Node, which returns a
List\<Node\> (as opposed to `children`, which omits non-Element nodes).
Once you have this list, you can use the usual List methods and
operators to manipulate the children of the node.

To add a node as the last child of its parent, use the List `add()`
method:

<?code-excerpt "lib/html.dart (nodes-add)"?>
{% prettify dart %}
  querySelector('#inputs').nodes.add(elem);
{% endprettify %}

To replace a node, use the Node `replaceWith()` method:

<?code-excerpt "lib/html.dart (replaceWith)"?>
{% prettify dart %}
  querySelector('#status').replaceWith(elem);
{% endprettify %}

To remove a node, use the Node `remove()` method:

<?code-excerpt "lib/html.dart (remove)"?>
{% prettify dart %}
  // Find a node by ID, and remove it from the DOM.
  querySelector('#expendable').remove();
{% endprettify %}

### Manipulating CSS styles

CSS, or *cascading style sheets*, defines the presentation styles of DOM
elements. You can change the appearance of an element by attaching ID
and class attributes to it.

Each element has a `classes` field, which is a list. Add and remove CSS
classes simply by adding and removing strings from this collection. For
example, the following sample adds the `warning` class to an element:

<?code-excerpt "lib/html.dart (classes-add)"?>
{% prettify dart %}
  var elem = querySelector('#message');
  elem.classes.add('warning');
{% endprettify %}

It’s often very efficient to find an element by ID. You can dynamically
set an element ID with the `id` property:

<?code-excerpt "lib/html.dart (set-id)"?>
{% prettify dart %}
  var message = DivElement();
  message.id = 'message2';
  message.text = 'Please subscribe to the Dart mailing list.';
{% endprettify %}

You can reduce the redundant text in this example by using method
cascades:

<?code-excerpt "lib/html.dart (elem-set-cascade)"?>
{% prettify dart %}
  var message = DivElement()
    ..id = 'message2'
    ..text = 'Please subscribe to the Dart mailing list.';
{% endprettify %}

While using IDs and classes to associate an element with a set of styles
is best practice, sometimes you want to attach a specific style directly
to the element:

<?code-excerpt "lib/html.dart (set-style)"?>
{% prettify dart %}
  message.style
    ..fontWeight = 'bold'
    ..fontSize = '3em';
{% endprettify %}

### Handling events

To respond to external events such as clicks, changes of focus, and
selections, add an event listener. You can add an event listener to any
element on the page. Event dispatch and propagation is a complicated
subject; [research the
details](http://www.w3.org/TR/DOM-Level-3-Events/#dom-event-architecture)
if you’re new to web programming.

Add an event handler using
<code><em>element</em>.on<em>Event</em>.listen(<em>function</em>)</code>,
where <code><em>Event</em></code> is the event
name and <code><em>function</em></code> is the event handler.

For example, here’s how you can handle clicks on a button:

<?code-excerpt "lib/html.dart (onClick)"?>
{% prettify dart %}
  // Find a button by ID and add an event handler.
  querySelector('#submitInfo').onClick.listen((e) {
    // When the button is clicked, it runs this code.
    submitData();
  });
{% endprettify %}

Events can propagate up and down through the DOM tree. To discover which
element originally fired the event, use `e.target`:

<?code-excerpt "lib/html.dart (target)"?>
{% prettify dart %}
  document.body.onClick.listen((e) {
    final clickedElem = e.target;
    // ...
  });
{% endprettify %}

To see all the events for which you can register an event listener, look
for "onEventType" properties in the API docs for [Element][] and its
subclasses. Some common events include:

-   change
-   blur
-   keyDown
-   keyUp
-   mouseDown
-   mouseUp


## Using HTTP resources with HttpRequest

Formerly known as XMLHttpRequest, the [HttpRequest][] class
gives you access to HTTP resources from within your browser-based app.
Traditionally, AJAX-style apps make heavy use of HttpRequest. Use
HttpRequest to dynamically load JSON data or any other resource from a
web server. You can also dynamically send data to a web server.


### Getting data from the server

The HttpRequest static method `getString()` is an easy way to get data
from a web server. Use `await` with the `getString()` call
to ensure that you have the data before continuing execution.

<?code-excerpt "test/html_test.dart (getString)" plaster="none" replace="/await.*;/[!$&!]/g"?>
{% prettify dart %}
  Future main() async {
    String pageHtml = [!await HttpRequest.getString(url);!]
    // Do something with pageHtml...
  }
{% endprettify %}

Use try-catch to specify an error handler:

<?code-excerpt "lib/html.dart (try-getString)"?>
{% prettify dart %}
  try {
    var data = await HttpRequest.getString(jsonUri);
    // Process data...
  } catch (e) {
    // Handle exception...
  }
{% endprettify %}

If you need access to the HttpRequest, not just the text data it
retrieves, you can use the `request()` static method instead of
`getString()`. Here’s an example of reading XML data:

<?code-excerpt "test/html_test.dart (request)" replace="/await.*;/[!$&!]/g"?>
{% prettify dart %}
  Future main() async {
    HttpRequest req = await HttpRequest.request(
      url,
      method: 'HEAD',
    );
    if (req.status == 200) {
      // Successful URL access...
    }
    // ···
  }
{% endprettify %}

You can also use the full API to handle more interesting cases. For
example, you can set arbitrary headers.

The general flow for using the full API of HttpRequest is as follows:

1.  Create the HttpRequest object.
2.  Open the URL with either `GET` or `POST`.
3.  Attach event handlers.
4.  Send the request.

For example:

{% comment %}
TODO: use original source from dart-tutorials-samples/web/portmanteaux/portmanteaux.dart
{% endcomment %}

<?code-excerpt "lib/html.dart (new-HttpRequest)"?>
{% prettify dart %}
  var request = HttpRequest();
  request
    ..open('POST', url)
    ..onLoadEnd.listen((e) => requestComplete(request))
    ..send(encodedData);
{% endprettify %}

### Sending data to the server

HttpRequest can send data to the server using the HTTP method POST. For
example, you might want to dynamically submit data to a form handler.
Sending JSON data to a RESTful web service is another common example.

Submitting data to a form handler requires you to provide name-value
pairs as URI-encoded strings. (Information about the URI class is in
the [URIs section][URIs] of the [Dart Library Tour.][Dart Library Tour])
You must also set the `Content-type` header to
`application/x-www-form-urlencode` if you wish to send data to a form
handler.

<?code-excerpt "test/html_test.dart (POST)"?>
{% prettify dart %}
  String encodeMap(Map data) => data.keys
      .map((k) => '${Uri.encodeComponent(k)}=${Uri.encodeComponent(data[k])}')
      .join('&');

  Future main() async {
    var data = {'dart': 'fun', 'angular': 'productive'};

    var request = HttpRequest();
    request
      ..open('POST', url)
      ..setRequestHeader(
        'Content-type',
        'application/x-www-form-urlencoded',
      )
      ..send(encodeMap(data));

    await request.onLoadEnd.first;

    if (request.status == 200) {
      // Successful URL access...
    }
    // ···
  }
{% endprettify %}


## Sending and receiving real-time data with WebSockets

A WebSocket allows your web app to exchange data with a server
interactively—no polling necessary. A server creates the WebSocket and
listens for requests on a URL that starts with **ws://**—for example,
ws://127.0.0.1:1337/ws. The data transmitted over a WebSocket can be a
string or a blob.  Often, the data is a JSON-formatted string.

To use a WebSocket in your web app, first create a [WebSocket][] object, passing
the WebSocket URL as an argument:

{% comment %}
Code inspired by:
https://github.com/dart-lang/dart-samples/blob/master/html5/web/websockets/basics/websocket_sample.dart

Once tests are written for the samples, consider getting code excerpts from
the websocket sample app.
{% endcomment %}

<?code-excerpt "test/html_test.dart (WebSocket)"?>
{% prettify dart %}
  var ws = WebSocket('ws://echo.websocket.org');
{% endprettify %}

### Sending data

To send string data on the WebSocket, use the `send()` method:

<?code-excerpt "test/html_test.dart (send)"?>
{% prettify dart %}
  ws.send('Hello from Dart!');
{% endprettify %}

### Receiving data

To receive data on the WebSocket, register a listener for message
events:

<?code-excerpt "test/html_test.dart (onMessage)" plaster="none"?>
{% prettify dart %}
  ws.onMessage.listen((MessageEvent e) {
    print('Received message: ${e.data}');
  });
{% endprettify %}

The message event handler receives a [MessageEvent][] object.
This object’s `data` field has the data from the server.

### Handling WebSocket events

Your app can handle the following WebSocket events: open, close, error,
and (as shown earlier) message. Here’s an example of a method that
creates a WebSocket object and registers handlers for open, close,
error, and message events:

<?code-excerpt "test/html_test.dart (initWebSocket)" plaster="none"?>
{% prettify dart %}
  void initWebSocket([int retrySeconds = 1]) {
    var reconnectScheduled = false;

    print("Connecting to websocket");

    void scheduleReconnect() {
      if (!reconnectScheduled) {
        Timer(Duration(seconds: retrySeconds),
            () => initWebSocket(retrySeconds * 2));
      }
      reconnectScheduled = true;
    }

    ws.onOpen.listen((e) {
      print('Connected');
      ws.send('Hello from Dart!');
    });

    ws.onClose.listen((e) {
      print('Websocket closed, retrying in ' + '$retrySeconds seconds');
      scheduleReconnect();
    });

    ws.onError.listen((e) {
      print("Error connecting to ws");
      scheduleReconnect();
    });

    ws.onMessage.listen((MessageEvent e) {
      print('Received message: ${e.data}');
    });
  }
{% endprettify %}


## More information

This section barely scratched the surface of using the dart:html
library. For more information, see the documentation for
[dart:html.][dart:html]
Dart has additional libraries for more specialized web APIs, such as
[web audio,][web audio] [IndexedDB,][IndexedDB] and [WebGL.][WebGL]

For more information about Dart web libraries, see the
[web library overview.][web library overview]

[AnchorElement]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-html/AnchorElement-class.html
[dart:html]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-html/dart-html-library.html
[Dart Library Tour]: {{site.dartlang}}/guides/libraries/library-tour
[Document]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-html/Document-class.html
[Element]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-html/Element-class.html
[HttpRequest]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-html/HttpRequest-class.html
[IndexedDB]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-indexed_db/dart-indexed_db-library.html
[MessageEvent]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-html/MessageEvent-class.html
[Nodes]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-html/Node-class.html
[URIs]: {{site.dartlang}}/guides/libraries/library-tour#uris
[web audio]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-web_audio/dart-web_audio-library.html
[WebGL]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-web_gl/dart-web_gl-library.html
[WebSocket]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-html/WebSocket-class.html
[web library overview]: /guides/web-programming
[Window]: {{site.dart_api}}/{{site.data.pkg-vers.SDK.channel}}/dart-html/Window-class.html
