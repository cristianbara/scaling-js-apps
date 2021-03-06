# Exercise: Messages Blade (30 mins)

Your team is building the Messages Blade. Sweet!

The Message Blade should have the following functionality:

* Display a list of chat messages
* Display the user who sent a message
* Display the timestamp of a message

## Create the Blade

You already have the application within the BladeRunnerJS installation. Within the root of the application directory you'll see a `blades` sub-directory. This is where blades are to be created.

From the BladeRunnerJS `sdk` directory run the following command to create a Blade:

```
./brjs create-blade modularapp default messages
```

You've just scaffolded your first Blade. You can find the Blade skeleton in
`apps/modularapp/blades/messages`.

You can find out more about what's just been created in the [Create a Blade docs](http://bladerunnerjs.org/docs/use/create_blade/).

## View the Blade in a Workbench

Start the BRJS development server by running the following command from the `sdk`
directory:

```
./brjs serve
```

Now navigate to `http://localhost:7070/modularapp/default/messages/workbench/`
to see your *amazing* Blade.

Feel free to take a look around the Blade assets to see how the code is structured
and familiarise yourself with where things are.

## Build the View

Let's update the view to have the elements we need for our required functionality.

Open up the `blades/messages/resources/html/view.html` file and make it look as follows:

```html
<section class="chat-messages" id="modularapp.messages.view-template">

	<div class="chat-messages-container">
		<div class="chat-message">
			<span class="chat-message-user-id">Some UserId</span>
			<span class="chat-message-text">Some Test</span>
			<span class="chat-message-timestamp">Some Timestamp</span>
		</div>
	</div>

</section>

```

Reload the Workbench to make sure this you see the fake data. We'll add the data bindings later.

Next, let's add some styling. Update `blades/messages/themes/standard/style.css` as follows:

```css
/* Message Blade container element */
.chat-messages {
	min-width: 400px;
	height: 200px;
	overflow: auto;
}

/* individual message */
.chat-message {
	position: relative;
	overflow: auto;

	margin-bottom: 10px;
}

.chat-message-user-id {
	position: absolute;
	top: 0;
	left: 0;

	font-weight: bold;
	width: 150px;
	margin-right: 5px;
	text-align: right;

	cursor: pointer;
}

.chat-message-user-id:after {
	content:': ';
}

.chat-message-text {
	display: inline-block;

	-webkit-box-sizing: border-box;
	-moz-box-sizing: border-box;
	box-sizing: border-box;

	/* space for userId */
	padding-left: 160px;
	/* space for timestamp */
	padding-right: 100px;

	width: 100%;
	min-width: 200px
}

.chat-message-timestamp {
	position: absolute;
	top: 0;
	right: 0;

	width: 100px;

	text-align: right;
	font-style: italic;
	font-size: small;
}
```

How does the Blade look in the Workbench now?

Time to look at the View Model.

## The MessagesViewModel

For this workshop we're using KnockoutJS as our data-binding solution. Knockout uses View Models that are logical representations of the views. A purist opinions of views is that that there
should be no business logic in a View Model, but we're building a reasonably simple
Blade (but feel free to refactor afterwards).

You can find the class definition for the Messages View Model in `blades/messages/src/MessagesViewModel.js`;
yeah, sorry about the folder structure!

Update the `MessagesViewModel` definition to look as follows. **Please note that it requires a `MessageItemViewModel` that we've not created yet so refreshing the workbench will show an error**:

```js
'use strict';

var ko = require( 'ko' );

var MessageItemViewModel = require( './MessageItemViewModel' );

function MessagesViewModel() {
  this.messages = ko.observableArray( [] );
}

MessagesViewModel.prototype.addMessage = function( message ) {
  var model = new MessageItemViewModel( message );
  this.messages.push( model );
};

module.exports = MessagesViewModel;
```

**You'll noticed that the tooling supports Node.js-style `require( 'module' )` calls
and the functionality that the file exposes is determined via assignment to `module.exports`.**

You'll also see that a class called `MessageItemViewModel` is being required (and until you create it the Workbench will inform you it doesn't exist). This
View Model represents individual messages. So, create a `MessageItemViewModel.js` file
in `src/` and set the content as follows:

```js
'use strict';

function MessageItemViewModel( message ) {
  this.text = ko.observable( message.text );
  this.timestamp = ko.observable( message.timestamp );
  this.userId = ko.observable( message.userId );
}

module.exports = MessageItemViewModel;
```

### Loop over the messages Array and Show the Message Data

In order to do this you'll need to remove the fake data and update the view definition (`view.html`) with an
appropriate `data-bind="foreach:..."` property and value. Do this on the element with the class
of `chat-messages-container`.

Once you've done that, show the values of each of the MessageItemViewModel in their respective spans.
You don't need any help on this one, just use `data-bind` for this.

##### Hints

* You want to update the `text` of the `span` elements so use the `text` Knockout
binding.

##### Solution:

If you've not used KnockoutJS before or you just want to check your solution here it is:

```html
<section class="chat-messages" id="modularapp.messages.view-template">

	<div class="chat-messages-container" data-bind="foreach: messages">
		<div class="chat-message">
			<span class="chat-message-user-id" data-bind="text: userId"></span>
			<span class="chat-message-text" data-bind="text: text"></span>
			<span class="chat-message-timestamp" data-bind="text: timestamp"></span>
		</div>
	</div>

</section>

```

### Check your Work in the Workbench

Once you've done this reload the Workbench to see the result of your fine work...
*Err, I can't see anything!*.

Good point. What do you do when you don't have any data? Well, before we look into that
we need to consider *what's shown when there are no messages?*.

### Tell the User when there are No Messages

Add a new element that tells the user there are no messages and is only show
when there are no messages.

##### Solution:

Here's the HTML for the solution:

```html
<section class="chat-messages" id="modularapp.messages.view-template">

	<div class="chat-messages-container" data-bind="foreach: messages">
		<div class="chat-message">
			<span class="chat-message-user-id" data-bind="text: userId"></span>
			<span class="chat-message-text" data-bind="text: text"></span>
			<span class="chat-message-timestamp" data-bind="text: timestamp"></span>
		</div>
	</div>

	<div class="chat-messages-no-messages" data-bind="visible: showNoMessages">
		Nobody's chatting!
	</div>

</section>

```

And the JavaScript for the View Model as part of the solution:

```js
function MessagesViewModel() {
	this.messages = ko.observableArray( [] );
	this.showNoMessages = ko.computed( function() {
		return ( this.messages().length === 0 );
	}, this );
}
```

### Add Fake Data via the Workbench

Okay, back to the fact we don't have any data to help us develop our Blade when it's
in a state where there is data.

The solution is to add some fake data. So, let's take a look at the code.

Back in the `MessagesViewModel` there is an `addMessage` function that takes a
data structure that is passed into the `MessageItemViewModel` constructor. The
new instance is then pushed onto the `messages` Array. Let's use that function
to add some fake data.

Open up `workbench/index.html` and after the `var model = new MessagesViewModel();`
line add the following:

```js
model.addMessage( { userId: 'leggetter', text: 'hello', timestamp: new Date() } );
model.addMessage( { userId: 'andyberry88', text: 'Yo!', timestamp: new Date() } );
```

Reload the workbench and you'll now see some fake data so that you can develop
your workbench in the required state.

### Change the timestamp to a nicely formatted value

We don't like the default timestamp formatting. Update the `MessageItemViewModel` so that
it sets the `timestamp` property to something that's nicer.

BRJS comes with the [momentjs](http://momentjs.com/) library so you can do the following
within `MessageItemViewModel.js` to use it:

```js
var moment = require( 'momentjs' );
```

##### Solution

```
var formattedDate = moment( message.timestamp ).format('h:mm:ss a');
this.timestamp = ko.observable( formattedDate );
```

## Test the MessagesViewModel

A key part of building a quality maintainable application is that it's tested. So,
let's write a test that checks the default message should be blank.

Navigate to `blades/messages/test-unit/tests` and update `MessagesViewModelTest.js`
as follows:

```js
var MessagesViewModelTest = TestCase( 'MessagesViewModelTest' );

var MessagesViewModel = require( 'modularapp/messages/MessagesViewModel' );

MessagesViewModelTest.prototype.testAddingAMessageIncreasesMessageCountByOne = function() {
  var model = new MessagesViewModel();
  var expectedMessageCount = 0;

  model.addMessage( { userId: 'test', text: 'test text', timestamp: new Date() } );
  assertEquals( expectedMessageCount, model.messages().length );
};
```

### Running Tests

There are lots of testing tools available. BladeRunnerJS comes with [js-test-driver](https://code.google.com/p/js-test-driver/)
built-in.

In order to run the tests you first need to start the test server. From the `sdk`
directory run:

```
./brjs test-server --no-browser
```

In your browser navigate to http://localhost:4224/capture. This is the browser window/tab
that the test server will instruct to execute your tests.

Now that the test server is running open another terminal/command prompt and execute
the following from the `sdk` directory:

```
./brjs test ../apps/modularapp/blades/messages
```

This will execute all the tests it finds within that directory. For now, this
is just the single test that we've written.

You should see output similar to the following:

```
› ./brjs test ../apps/modularapp/blades/messages

Testing tests (UTs):
Chrome: Reset
Chrome: Reset
F
Total 1 tests (Passed: 0; Fails: 1; Errors: 0) (7.00 ms)
  Chrome 36.0.1985.125 Mac OS: Run 1 tests (Passed: 0; Fails: 1; Errors 0) (7.00 ms)
    MessagesViewModelTest.testAddingAMessageIncreasesMessageCountByOne failed (7.00 ms): AssertError: expected 0 but was 1
      Error: expected 0 but was 1
          at MessagesViewModelTest.testAddingAMessageIncreasesMessageCountByOne (http://localhost:4224/test/tests/MessagesViewModelTest.js:10:55)

Tests failed: Tests failed. See log for details.
Tests Failed.
```

Let's fix that error by updating the test to correctly check the number of messages:

```js
var MessagesViewModelTest = TestCase( 'MessagesViewModelTest' );

var MessagesViewModel = require( 'modularapp/messages/MessagesViewModel' );

MessagesViewModelTest.prototype.testAddingAMessageIncreasesMessageCountByOne = function() {
  var model = new MessagesViewModel();
  var expectedMessageCount = 1;

  model.addMessage( { userId: 'test', text: 'test text', timestamp: new Date() } );
  assertEquals( expectedMessageCount, model.messages().length );
};
```

If you run the `brjs test` command the test will now pass:

```
› ./brjs test ../apps/modularapp/blades/messages

Testing tests (UTs):
Chrome: Reset
Chrome: Reset
.
Total 1 tests (Passed: 1; Fails: 0; Errors: 0) (7.00 ms)
  Chrome 36.0.1985.125 Mac OS: Run 1 tests (Passed: 1; Fails: 0; Errors 0) (7.00 ms)
Tests Passed.
```

We've now written our first test and made it pass.

For full details see the [Running Test documentation](http://bladerunnerjs.org/docs/use/running_tests/).

### Additional Tests

Now that you know how to write and run tests you can also write test that assert
the functionality the Blade is to provide:

* Check the `MessageItemViewModel` properties are initialised as expected from the
data structure that's passed to its constructor
* Add a message to the `MessagesViewModel` using `addMessage` function and ensure the added message
has the correct `userId`, `text` and `timestamp` values.
* If you've time, ensure the `timestamp` is formatted as expected in the `MessageItemViewModel`

*Heads Up! test functions must have a name with the `test` prefix e.g. `testThisThing`*

## Basic Blade Functionality Complete - push to github

Congratulations! The basic functionality for this Blade is complete. It's time to
commit your changes locally and push them to github.

* `git add blades/messages`
* `git commit -m 'basic input blade functionality'`
* `git pull origin master`
* Fix any merge conflicts - *there shouldn't be any*
* `git push origin master`

## Where next?

If you've completed this really quickly you can always try creating another Blade.
