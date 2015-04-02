marko-widgets
==============

[![Build Status](https://travis-ci.org/raptorjs/marko-widgets.svg?branch=master)](https://travis-ci.org/raptorjs/marko-widgets) [![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/raptorjs/marko-widgets?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Marko Widgets extends the [Marko templating language](https://github.com/raptorjs/marko) to provide a simple and efficient mechanism for binding behavior to UI components rendered on either the server or in the browser. In addition, changing a widgets state will result in the DOM automatically being updated without writing extra code. Marko Widgets has adopted many of the good design principles promoted by the [React](https://facebook.github.io/react/index.html) team, but aims to be much lighter and often faster.

![eBay Open Source](https://raw.githubusercontent.com/raptorjs/optimizer/master/images/ebay.png)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

# Table of Contents

- [Features](#features)
- [Design Philosophy](#design-philosophy)
- [Sample Code](#sample-code)
	- [Stateless Widget](#stateless-widget)
	- [Stateless Widget with Behavior](#stateless-widget-with-behavior)
	- [Stateful Widget](#stateful-widget)
	- [Stateful Widget with Update Handlers](#stateful-widget-with-update-handlers)
	- [Complex Widget](#complex-widget)
- [Installation](#installation)
- [Glossary](#glossary)
- [Usage](#usage)
	- [Binding Behavior](#binding-behavior)
	- [Widget Config](#widget-config)
	- [Referencing Nested Widgets](#referencing-nested-widgets)
	- [Referencing Nested DOM Elements](#referencing-nested-dom-elements)
	- [Adding Event Listeners](#adding-event-listeners)
		- [Adding DOM Event Listeners](#adding-dom-event-listeners)
		- [Adding Custom Event Listeners](#adding-custom-event-listeners)
	- [Rendering Widgets in the Browser](#rendering-widgets-in-the-browser)
	- [Rendering Widgets on the Server](#rendering-widgets-on-the-server)
- [API](#api)
	- [Widget](#widget)
		- [Methods](#methods)
			- [$(querySelector)](#$queryselector)
			- [addEventListener(eventType, listener)](#addeventlistenereventtype-listener)
			- [appendTo(targetEl)](#appendtotargetel)
			- [destroy()](#destroy)
			- [detach()](#detach)
			- [emit(eventType, arg1, arg2, ...)](#emiteventtype-arg1-arg2-)
			- [getEl(widgetElId)](#getelwidgetelid)
			- [getEls(id)](#getelsid)
			- [getElId(widgetElId)](#getelidwidgetelid)
			- [getWidget(id[, index])](#getwidgetid-index)
			- [getWidgets(id)](#getwidgetsid)
			- [insertAfter(targetEl)](#insertaftertargetel)
			- [insertBefore(targetEl)](#insertbeforetargetel)
			- [isDestroyed()](#isdestroyed)
			- [on(eventType, listener)](#oneventtype-listener)
			- [prependTo(targetEl)](#prependtotargetel)
			- [ready(callback)](#readycallback)
			- [replace(targetEl)](#replacetargetel)
			- [replaceChildrenOf(targetEl)](#replacechildrenoftargetel)
			- [rerender(data, callback)](#rerenderdata-callback)
			- [subscribeTo(targetEventEmitter)](#subscribetotargeteventemitter)
		- [Properties](#properties)
			- [this.el](#thisel)
			- [this.id](#thisid)
- [Changelog](#changelog)
- [Discuss](#discuss)
- [Contributors](#contributors)
- [Contribute](#contribute)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Features

- Simple
	- Clean JavaScript syntax for defining widgets
	- Utilizes [Marko templates](https://github.com/raptorjs/marko) (an HTML-based templating language) for the view
	- Supports stateful and stateless widgets
	- No complex class hierarchy
	- Simple, declarative event binding for both DOM and custom events
	- Lifecycle management for widgets (easily destroy and create widgets)
	- Events bubble up and view state changes trickle down
	- Only need to understand a few concepts to get started
- High performance
	- Lightning fast performance on the server and in the browser (see [Marko vs React: Performance Benchmark](https://github.com/patrick-steele-idem/marko-vs-react))
	- Supports streaming and asynchronous rendering
	- Efficient binding of behavior of UI components rendered on the server and in the browser
	- Efficient updating of the DOM via the following tricks:
		- Batched updates
		- When re-rendering a widget, nested widgets are reused
		- Only widgets whose state changed are re-rendered
		- Full re-rendering of a widget can be short circuited if state transition handlers are provided
		- For container components, nested body DOM nodes are automatically preserved
		- Entire DOM subtrees can be preserved between rendering
		- Smart template compilers to offload as much work to compile time
	- Very efficient event delegation
- Lightweight
	- Extremely small JavaScript runtime (~6.3 KB gzipped)
	- No dependencies on any other JavaScript library such as jQuery
	- Focused exclusively on the UI view (easily mix and match with other libraries/frameworks)

# Design Philosophy

- A UI component should encapsulate view, behavior and styling
- A complex page should be decomposed into modular UI components
- UI components should be used as building blocks
- A component's view should be driven by a pure function that accepts an input state and produces output HTML
- A UI component should be independently testable
- A UI component should not leak its internal implementation
- A UI component should be installable via npm
- A UI component should play nice with other frameworks and libraries
- UI components should be easily composable
- Developers should not need to manually manipulate the DOM

# Sample Code

Marko Widgets allows you to declaratively bind behavior to an HTML element inside a Marko template. The widget provides the client-side behavior for your UI component.

## Stateless Widget

__src/components/app-hello/template.marko__

```html
<div w-bind>
	Hello ${data.name}!
</div>
```

__src/components/app-hello/index.js__

```javascript
module.exports = require('marko-widgets').defineWidget({
	template: require.resolve('./template.marko'),

	getTemplateData: function(state, input) {
		return {
			name: input.name
		};
	},

	init: function() {
		var el = this.el; // The root DOM element that the widget is bound to
		console.log('Initializing widget: ' + el.id);
	}
});
```

Congratulations, you just built a reusable UI component! Your UI component can be embedded in other Marko template files:

```html
<div>
	<app-hello name="Frank"/>
</div>
```

In addition, your UI can be rendered and added to the DOM using the JavaScript API:

```javascript
var widget = require('./app-hello')
	.render({
		name: 'John'
	})
	.appendTo(document.body)
	.getWidget();

// Changing the props will trigger the widget to re-render
// with the new props and for the DOM to be updated:
widget.setProps({
	name: 'Jane'
});
```

## Stateless Widget with Behavior

__src/components/app-hello/template.marko__

```html
<div w-bind
	 w-onClick="handleClick">

	Hello ${data.name}!

</div>
```

__src/components/app-hello/index.js__

```javascript
module.exports = require('marko-widgets').defineWidget({
	template: require.resolve('./template.marko'),

	getTemplateData: function(state, input) {
		return {
			name: input.name
		};
	},

	handleClick: function() {
		this.setSelected(true);
	},

	setSelected: function(selected) {
		if (selected) {
			this.el.style.backgroundColor = 'yellow';
		} else {
			this.el.style.backgroundColor = null;
		}
	}
});
```

## Stateful Widget

Let's create a stateful widget that changes to yellow when you click on it:

__src/components/app-hello/template.marko__

```html
<div w-bind
	 w-onClick="handleClick"
	 style="background-color: ${data.color}">

	Hello ${data.name}!

</div>
```

__src/components/app-hello/index.js__

```javascript

var styleUtil = require('marko-widgets/util/style');

module.exports = require('marko-widgets').defineWidget({
	template: require.resolve('./template.marko'),

	getInitialState: function(input) {
		return {
			name: input.name,
			selected: input.selected || false;
		}
	},

	getTemplateData: function(state, input) {
		var style = ;

		return {
			name: state.name,
			color: state.selected ? 'yellow' : 'transparent'
		};
	},

	handleClick: function() {
		this.setState('selected', true);
	},

	isSelected: function() {
		return this.state.selected;
	}
});
```

## Stateful Widget with Update Handlers

If you want to avoid re-rendering a widget for a particular state property change then simply provide your own method to handle the state change as shown below:

__src/components/app-hello/template.marko__

```html
<div w-bind
	 w-onClick="handleClick"
	 style="background-color: ${data.color}">

	Hello ${data.name}!

</div>
```

__src/components/app-hello/index.js__

```javascript
module.exports = require('marko-widgets').defineWidget({
	template: require.resolve('./template.marko'),

	getInitialState: function(input) {
		return {
			name: input.name,
			selected: input.selected || false;
		}
	},

	getTemplateData: function(state, input) {
		var style = ;

		return {
			name: state.name,
			color: state.selected ? 'yellow' : 'transparent'
		};
	},

	handleClick: function() {
		this.setState('selected', true);
	},

	isSelected: function() {
		return this.state.selected;
	},

	update_selected: function(newSelected) {
		// Manually update the DOM to reflect the new "selected"
		// state" to avoid re-rendering the entire widget.
		if (selected) {
			this.el.style.backgroundColor = 'yellow';
		} else {
			this.el.style.backgroundColor = null;
		}
	}
});
```

## Complex Widget

```html
<div w-bind>
	<app-overlay title="My Overlay"
		w-id="overlay"
		w-onBeforeHide="handleOverlayBeforeHide">
		Body content for overlay.
	</app-overlay>

	<button type="button"
		w-onClick="handleShowButtonClick">
		Show Overlay
	</button>

	<button type="button"
		w-onClick="handleHideButtonClick">
		Hide Overlay
	</button>
</div>
```

Below is the content of `index.js` where the widget type is defined:

```javascript
module.exports = require('marko-widgets').defineWidget({
	init: function() {
		// this.el will be the raw DOM element the widget instance
		// is bound to:
		var el = this.el;
	},

	template: require.resolve('./template.marko'),

	handleShowButtonClick: function(event) {
		console.log('Showing overlay...');
        this.getWidget('overlay').show();
    },

    handleHideButtonClick: function(event) {
		console.log('Hiding overlay...');
        this.getWidget('overlay').hide();
    },

    handleOverlayBeforeHide: function(event) {
        console.log('The overlay is about to be hidden!');
    }
})
```

## Container Widget

A container widget supports nested content. When the container widget is re-rendered, the nested content is automatically preserved.

__src/components/app-alert/index.js__

```html
<div class="alert alert-${data.type}" w-bind>
	<i class="alert-icon"/>
	<span w-body></span>
</div>
```

__src/components/app-alert/template.marko__

```javascript
module.exports = require('marko-widgets').defineWidget({
	init: function() {
		// this.el will be the raw DOM element the widget instance
		// is bound to:
		var el = this.el;
	},

	template: require.resolve('./template.marko'),

	getInitialState: function(input) {
		return {
			type: input.type || 'success'
		}
	},

	getTemplateData: function(state, input) {
		return {
			type: state.type
		};
	},

	getWidgetBody: function(input) {
		return input.message || input.renderBody;
    },

	setType: function(type) {
		this.setState('type', type);
	}
})
```

The widget can then be used as shown below:

```html
<app-alert message="This is a success alert"/>

<app-alert>
	This is a success alert
</app-alert>

<app-alert message="This is a failure alert" type="failure"/>

<app-alert type="failure">
	This is a failure alert
</app-alert>
```

# Installation

```bash
npm install marko-widgets --save
```

# Glossary

A few definitions before you get started:

* A "widget" is the "client-side behavior" of a UI component
* A widget instance has the following characteristics
    * All widget instances are bound to a DOM element
    * All widgets are [event emitters](http://nodejs.org/api/events.html)
* Client-side behavior includes the following:
    * Attaching DOM event listeners (mouse click, keyboard press, etc.)
    * Attaching listeners to other widgets
    * Manipulating the DOM
    * Publishing client-side events
    * etc.

# Usage

## Binding Behavior

Using the bindings for Marko, you can bind a widget to a rendered DOM element using the custom `w-bind` attribute as shown in the following sample template:

```html
<div class="my-component" w-bind="./widget">
    <h1>Click Me</h1>
</div>
```

You can also choose to leave the value of the `w-bind` attribute empty. If the value of `w-bind` is empty then `marko-widgets` will search for a widget module by first checking to see if `widget.js` exists and then `index.js`. Example:

```html
<div class="my-component" w-bind>
    <h1>Click Me</h1>
</div>
```

The widget bound to the `<div>` should then be implemented as a CommonJS module that exports a constructor function. During client-side initialization, a new instance of your widget will be created for each rendered DOM element that the widget is bound to. A sample widget implementation is shown in the following JavaScript code:

__src/pages/index/widget.js:__

```javascript
function Widget(config) {
    var rootEl = this.el; // this.el returns the root element that the widget is bound to
    var self = this;

    rootEl.addEventListener('click', function() {
        self.addText('You clicked on the root element!');
    });
}

Widget.prototype = {
    addText: function(text) {
        this.el.appendChild(document.createTextNode(text));
    }
};

exports.Widget = Widget;
```

In order for everything to work on the client-side we need to include the code for the `marko-widgets` module and the `./widget.js` module as part of the client bundle and we also need to use the custom `<init-widgets>` tag to let the client know which widgets rendered on the server need to be initialized on the client. To include the client-side dependencies will be using the [optimizer](https://github.com/raptorjs/optimizer) module and the taglib that it provides. Our final page template is shown below:

__src/pages/index/template.marko:__

```html
<optimizer-page name="index" package-path="./optimizer.json" />

<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Widgets Demo</title>
    <optimizer-head/>
</head>
<body>
    <!-- Bind a widget to a div element using the "w-bind" attribute -->
    <div class="my-component" w-bind="./widget">
        <h1>Click Me</h1>
    </div>

    <optimizer-body/>
    <init-widgets/>
</body>
</html>
```

The `optimizer.json` that includes the required client-side code is shown below:

__src/pages/index/optimizer.json:__

```javascript
{
    "dependencies": [
        "require: marko-widgets",
        "require: ./widget"
    ]
}
```

In the above example, the final HTML will be similar to the following:

```html
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Widgets Demo</title>
    </head>
    <body>
        <div data-rwidget="/src/pages/index/widget" id="w0" class="my-component">
            <h1>Click Me</h1>
        </div>
        <script src="static/index-8947595a.js" type="text/javascript"></script>
        <span style="display:none;" data-ids="w0" id="rwidgets"></span>
    </body>
</html>
```

To try out and experiment with this code please see the documentation and source code for the [widgets-bind-behavior](https://github.com/raptorjs/raptor-samples/tree/master/widgets-bind-behavior) sample app.

## Widget Config

Arbitrary widget configuration data determined at render time can be provided to the constructor of a widget. There are two options for attaching widget configuration data to a widget and those options are as follows:


__Option 1) Using the `w-config` attribute:__

_template.marko:_

```html
<div w-bind="./widget" w-config="{message: 'Hello World'}">
...
</div>
```

_widget.js:_

```javascript
function Widget(config) {
    console.log(config.message); // Output: 'Hello World'
}

exports.Widget = Widget;
```

__Option 2) As a `widgetConfig` property of the input data model for a Marko template:__

_renderer.js:_

```javascript
var template = require('marko').load(require.resolve('./template.marko'));

exports.renderer = function(input, out) {
    template.render({
            widgetConfig: {
                message: 'Hello World'
            }
        },
        out);
}
```

_template.marko:_

```html
<div w-bind="./widget">
...
</div>
```

_widget.js:_

```javascript
function Widget(config) {
    console.log(config.message); // Output: 'Hello World'
}

exports.Widget = Widget;
```

## Referencing Nested Widgets

The `marko-widgets` taglib also provides support for allowing a widget to communicate directly with nested widgets. A nested widget can be assigned a widget ID (only needs to be unique within the scope of the containing widget) and the containing widget can then reference the nested widget by the assigned widget ID using the `this.getWidget(id)` method.

The following HTML template fragment contains a widget that has three nested [sample-button](https://github.com/raptorjs/raptor-sample-ui-components/tree/master/components/sample-button) widgets. Each nested [sample-button](https://github.com/raptorjs/raptor-sample-ui-components/tree/master/components/sample-button) is assigned an ID (i.e. `primaryButton`, `successButton` and `dangerButton`).

```html
<div class="my-component" w-bind="./widget">
    <div class="btn-group">
        <sample-button label="Click Me" variant="primary" w-id="primaryButton"/>
        <sample-button label="Click Me" variant="success" w-id="successButton"/>
        <sample-button label="Click Me" variant="danger" w-id="dangerButton"/>
    </div>
    ...
</div>
```

The containing widget can then reference a particular nested widget as shown in the following sample JavaScript code:

```javascript
this.getWidget('dangerButton').on('click', function() {
    alert('You clicked on the danger button!');
});
```

Marko Widgets also supports referencing _repeated_ nested widgets as shown below:

```html
<div class="my-component" w-bind="./widget">
    <ul>
		<li for="todoItem in data.todoItems">
			<app-todo-item w-id="todoItems[]" todo-item="todoItem"/>
		</li>
	</ul>
</div>
```

The containing widget can then reference the repeated todo item widgets using the `this.getWidgets(id)` method as shown below:

```javascript
var todoItemWidgets = this.getWidgets('todoItems');
// todoItemWidgets will be an Array of todo item widgets
```

To try out and experiment with this code please see the documentation and source code for the [widgets-communication](https://github.com/raptorjs/raptor-samples/tree/master/widgets-communication) sample app.

## Referencing Nested DOM Elements

DOM elements nested within a widget can be given unique IDs based on the containing widget's ID. These DOM elements can then be efficiently looked up by the containing widget using methods provided. The `w-id` custom attribute can be used to assign DOM element IDs to HTML elements that are prefixed with the widget's ID. For example, given the following HTML template fragment:

```html
<form w-bind="./widget">
    ...
    <button type="submit" w-id="submitButton">Submit</button>
    <button type="button" w-id="cancelButton">Cancel</button>
</form>
```

Assuming the unique ID assigned to the widget is `w123`, the following would be the HTML output:

```html
<form id="w123">
    ...
    <button type="submit" id="w123-submitButton">Submit</button>
    <button type="button" id="w123-cancelButton">Cancel</button>
</form>
```

Finally, to reference a widget's nested DOM element's the following code can be used in the containing widget:

```javascript
var submitButton = this.getEl('submitButton'); // submitButton.id === 'w123-submitButton'
var cancelButton = this.getEl('cancelButton'); // cancelButton.id === 'w123-cancelButton'

submitButton.style.border = '1px solid red';
```

The object returned by `this.getEl(id)` will be a raw [HTML element](https://developer.mozilla.org/en-US/docs/Web/API/element). If you want a jQuery wrapped element you can do either of the following:


Option 1) Use jQuery directly:

```javascript
var $submitButton = $(this.getEl('submitButton'));
```

Option 2) Use the `this.$()` method:

```javascript
var $submitButton = this.$('#submitButton');
```

Marko Widgets also supports referencing _repeated_ nested DOM elements as shown below:

```html
<ul>
	<li for="color in ['red', 'green', 'blue']"
		w-id="colorListItems[]">
		$color
	</li>
</ul>
```

The containing widget can then reference the repeated DOM elements using the `this.getEls(id)` method as shown below:

```javascript
var colorListItems = this.getEls('colorListItems');
// colorListItems will be an Array of raw DOM <li> elements
```

## Adding Event Listeners

Marko Widgets supports attaching event listeners to nested DOM elements and nested widgets. Event listeners can either be registered declaratively in the Marko template or in JavaScript code.

### Adding DOM Event Listeners

A widget can subscribe to events on a nested DOM element.

Listeners can be attached declaratively as shown in the following sample code:

```html
<div w-bind>
	<form w-onsubmit="handleFormSubmit">
		<input type="text" value="email" w-onchange="handleEmailChange">
		<button>Submit</button>
	</form>
</div>
```

And then in the widget:

```javascript
function Widget() {

}

Widget.prototype = {
	handleFormSubmit: function(event, el) {
		event.preventDefault();
		// ...
	},

	handleEmailChange: function(event, el) {
		var email = el.value;
		this.validateEmail(email);
		// ...
	},

	validateEmail: function(email) {
		// ...
	}
}

exports.Widget = Widget;
```

NOTE: Event handler methods will be invoked with `this` being the widget instance and the following two arguments will be provided to the handler method:

1. `event` - The raw DOM event object (e.g. `event.target`, `event.clientX`, etc.)
2. `el` - The element that the listener was attached to (which can be different from `event.target` due to bubbling)


For performance reasons, Marko Widgets only adds one event listener to the root `document.body` element for each event type that bubbles. When Marko Widgets captures an event on `document.body` it will internally delegate the event to the appropriate widgets. For DOM events that do not bubble, Marko Widgets will automatically add DOM event listeners to each of the DOM nodes. If a widget is destroyed, Marko Widgets will automatically do the appropriate cleanup to remove DOM event listeners.

You can also choose to add listeners in JavaScript code by assigning an "element id" to the nested DOM element (only needs to be unique within the scope of the containing widget) so that the nested DOM element can be referenced by the containing widget. The scoped widget element ID should be assigned using the `w-id="<id>"` attribute. For example, in the template:

```html
<div w-bind>
	<form w-id="form">
		<input type="text" value="email" w-id="email">
		<button>Submit</button>
	</form>
</div>
```

And then in the widget:

```javascript
function Widget() {
	var self = this;

	var formEl = this.getEl('form');
	formEl.addEventListener('submit', function(event) {
		self.handleFormSubmit(event, formEl)
	});

	// Or use jQuery if that is loaded on your page:
	var emailEl = this.getEl('email');
	$(emailEl).on('change', function(event) {
		self.handleEmailChange(event, emailEl)
	});
}

Widget.prototype = {
	handleFormSubmit: function(event, el) {
		event.preventDefault();
		// ...
	},

	handleEmailChange: function(event, el) {
		var email = el.value;
		this.validateEmail(email);
		// ...
	},

	validateEmail: function(email) {
		// ...
	}
}

exports.Widget = Widget;
```

### Adding Custom Event Listeners

A widget can subscribe to events on nested widgets. Every widget extends [EventEmitter](http://nodejs.org/api/events.html#events_class_events_eventemitter) and this allows each widget to emit events.

Listeners can be attached declaratively as shown in the following sample code:

```html
<div w-bind="./widget">
	<app-overlay title="My Overlay"
		w-onBeforeHide="handleOverlayBeforeHide">

		Content for overlay

	</app-overlay>
</div>
```

And then in the widget:

```javascript
function Widget() {
}

Widget.prototype = {
    handleOverlayBeforeHide: function(event) {
        console.log('The overlay is about to be hidden!');
    }
};

exports.Widget = Widget;
```

You can also choose to add listeners in JavaScript code by assigning an "id" to the nested widget (only needs to be unique within the scope of the containing widget) so that the nested widget can be referenced by the containing widget. The scoped widget ID should be assigned using the `w-id="<id>"` attribute. For example, in the template:

```html
<div w-bind="./widget">
	<app-overlay title="My Overlay"
		w-id="myOverlay">

		Content for overlay

	</app-overlay>
</div>
```

And then in the widget:

```javascript
function Widget() {
	var self = this;

	var myOverlay = this.getWidget('myOverlay');

	this.subscribeTo(myOverlay)
		.on('beforeHide', function(event) {
			self.handleOverlayBeforeHide(event);
		});
}

Widget.prototype = {
    handleOverlayBeforeHide: function(event) {
        console.log('The overlay is about to be hidden!');
    }
};

exports.Widget = Widget;
```

NOTE: `subscribeTo(eventEmitter)` is used to ensure proper cleanup if the subscribing widget is destroyed.

## Rendering Widgets in the Browser

Marko Widgets provides an API that can be used to create a `render(input[, callback])` function given a renderer:

```javascript
function renderer(input, out) {
	// ...
}

require('marko-widgets').renderable(exports, renderer);
```

The `renderable` method will modify the target object to add a new `render(input[, callback])` method that can be used to render the widget. In addition, the `renderable` method will also store the provided renderer in the `renderer` property of the target object.

An object that is made renderable, can then be rendered on the client as shown below:

_Synchronous render_:

```javascript
var widget = require('fancy-checkbox').render({
		checked: true,
		label: 'Foo'
	})
	.appendTo(document.body)
	.getWidget();

widget.setChecked(false);
widget.setLabel('Bar');
```

_Asynchronous render_:

```javascript
require('fancy-checkbox').render({
		checked: true,
		label: 'Foo'
	},
	function(err, renderResult) {
		if (err) {
			// ...
		}

		var widget = renderResult
			.appendTo(document.body)
			.getWidget();

		widget.setChecked(false);
		widget.setLabel('Bar');
	});
```

## Rendering Widgets on the Server

If a UI component is rendered on the server then that means that the HTML will be produced on the server and that separate JavaScript code will need to run in the browser to bind behavior to the widgets associated with the UI components rendered on the server. Marko Widgets keeps track of the rendered widgets associated with an ["out"](https://github.com/raptorjs/async-writer). The following code illustrates how to get the JavaScript code needed to initialize widgets in the browser:

```javascript
var markoWidgets = require('marko-widgets');
var template = require('marko').load(require.resolve('./template.marko'));

module.exports = function(req, res) {
	template.render(viewModel, function(err, html, out) {
		var initWidgetsCode = markoWidgets.getInitWidgetsCode(out);

		// Serialize the HTML and the JavaScript code to the browser
		res.json({
	            html: html,
	            js: initWidgetsCode
	        });
	});
}
```

And then, in the browser, the following code can be used to initialize the widgets:

```javascript
var result = JSON.parse(response.body);
var html = result.html
var js = result.js;

document.body.innerHTML = html; // Add the HTML to the DOM
eval(js); // Initialize the widgets to bind behavior!
```

# API

## Widget

### Methods

#### $(querySelector)

This is a convenience method for accessing a widget's DOM elements when jQuery is available. This mixin method serves as a proxy to jQuery to ease building queries based on widget element IDs.

Internally, this jQuery proxy method will resolve widget element IDs to their actual DOM element ID by prefixing widget element IDs with the widget ID. For example, where this is a widget with an ID of `w123`:


```javascript
this.$() ➡ $("#w123")
this.$("#myEl") ➡ $("#w123-myEl")
```

The usage of this mixin method is described below:

__`$()`__

Convenience usage to access the root widget DOM element wrapped as a jQuery object. All of the following are equivalent:

```javascript
this.$()
$(this.el)
$("#" + this.id)
```

__`$('#<widget-el-id>')`__

Convenience usage to access a nested widget DOM element wrapped as a jQuery object. All of the following are equivalent:

```javascript
this.$("#myEl")
$(this.getEl("myEl"))
$("#" + this.getElId("myEl"))
```

__`$('<selector>')`__

Convenience usage to query nested DOM elements scoped to the root widget DOM element. All of the following are equivalent:

```javascript
this.$("ul > li")
$("ul > li", this.el)
$("#" + this.id + " ul > li")
```

__`$('<selector>', '<widget-el-id>')`__

Convenience usage to query nested DOM elements scoped to a nested widget DOM element. All of the following are equivalent:

```javascript
this.$("li.color", "colorsUL")
this.$("#colorsUL li.color")
$("li.color", this.getEl("colorsUL"))
$("#" + this.getElId("colorsUL") + " li.color")
```

__`$('#<widget-el-id> <selector>')`__

Convenience usage to query nested DOM elements scoped to a nested widget DOM element. All of the following are equivalent:

```javascript
this.$("#colorsUL li.color")
this.$("li.color", "colorsUL")
$("li.color", this.getEl("colorsUL"))
$("#" + this.getElId("colorsUL") + " li.color")
```

__`$(callbackFunction)`__

Convenience usage to add a listener for the "on DOM ready" event and have the this object for the provided callback function be the current widget instance. All of the following are equivalent:

```javascript
this.$(function() { /*...*/ });
$(function() { /*...*/ }.bind(this));      // Using Function.prototype.bind
$($.proxy(function() { /*...*/ }, this));
```

#### addEventListener(eventType, listener)

#### appendTo(targetEl)

Moves the widget's root DOM node from the current parent element to a new parent element. For example:

```javascript
this.appendTo(document.body);
```

#### destroy()

Destroys the widget by unsubscribing from all listeners made using the `subscribeTo` method and then detaching the widget's root element from the DOM. All nested widgets (discovered by querying the DOM) are also destroyed.

#### detach()

Detaches the widget's root element from the DOM by removing the node from its parent node.

#### emit(eventType, arg1, arg2, ...)

Emits an event. This method is inherited from EventEmitter (see [Node.js Events: EventsEmitter](http://nodejs.org/api/events.html#events_class_events_eventemitter)

#### getEl(widgetElId)

Returns a nested DOM element by prefixing the provided `widgetElId` with the widget's ID. For Marko, nested DOM elements should be assigned an ID using the `w-id` custom attribute.  Returns `this.el` if no `widgetElId` is provided.

#### getEls(id)

Returns an Array of _repeated_ `DOM` elements for the given ID. Repeated DOM elements must have a value for the `w-id` attribute that ends with `[]` (e.g., `w-id="myDivs[]"`)

#### getElId(widgetElId)

Similar to `getEl`, but only returns the String ID of the nested DOM element instead of the actual DOM element.

#### getWidget(id[, index])

Returns a reference to a nested `Widget` for the given ID. If an `index` is provided and the target widget is a repeated widget (e.g. `w-id="myWidget[]"`) then the widget at the given index will be returned.

#### getWidgets(id)

Returns an Array of _repeated_ `Widget` instances for the given ID. Repeated widgets must have a value for the `w-id` attribute that ends with `[]` (e.g., `w-id="myWidget[]"`)

#### insertAfter(targetEl)

#### insertBefore(targetEl)

#### isDestroyed()

#### on(eventType, listener)

#### prependTo(targetEl)

#### ready(callback)

#### replace(targetEl)

#### replaceChildrenOf(targetEl)

#### rerender(data, callback)

#### subscribeTo(targetEventEmitter)

### Properties

#### this.el

The root [HTML element](https://developer.mozilla.org/en-US/docs/Web/API/element) that the widget is bound to.

#### this.id

The String ID of the root [HTML element](https://developer.mozilla.org/en-US/docs/Web/API/element) that the widget is bound to.

# Changelog

See [CHANGELOG.md](CHANGELOG.md)

# Discuss

Chat channel: [![Gitter](https://badges.gitter.im/Join Chat.svg)](https://gitter.im/raptorjs/marko-widgets?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Questions or comments can also be posted on the [RaptorJS Google Groups Discussion Forum](http://groups.google.com/group/raptorjs).

# Contributors

* [Patrick Steele-Idem](https://github.com/patrick-steele-idem) (Twitter: [@psteeleidem](http://twitter.com/psteeleidem))
* [Phillip Gates-Idem](https://github.com/philidem/) (Twitter: [@philidem](https://twitter.com/philidem))

# Contribute

Pull Requests welcome. Please submit Github issues for any feature enhancements, bugs or documentation problems.

# License

Apache License v2.0
