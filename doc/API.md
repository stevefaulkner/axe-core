# aXe Javascript Accessibility API

## Table of Contents

1. [Section 1: Introduction](#section-1-introduction)
	1. [Get Started](#getting-started)
1. [Section 2: API Reference](#section-2-api-reference)
	1. [Overview](#overview)
	1. [API Notes](#api-notes)
	1. [API Name: axe.getRules](#api-name-axegetrules)
	1. [API Name: axe.configure](#api-name-axeconfigure)
	1. [API Name: axe.a11yCheck](#api-name-axea11ycheck)
		1. [Parameters](#parameters-2)
			1. [Context Parameter](#context-parameter)
			2. [Options Parameter](#options-parameter)
			3. [Callback Parameter](#callback-parameter)
		1. [Results Object](#results-object)
1. [Section 3: Example Reference](#section-3-example-reference)

## Section 1: Introduction

The aXe API is designed to be an improvement over the previous generation of accessibility APIs. It provides the following benefits:

* Runs in any modern browser
* Designed to work with existing testing infrastructure
* Runs locally, no connection to a third-party server is necessary
* Performs violation checking on multiple levels of nested iframes
* Provides list of rules and elements that passed accessibility checking, ensuring rules have been run against entire document

### Getting Started
This section gives a quick description of how to use the aXe APIs to analyze web page content and return a JSON object that lists any accessibility violations found.

The aXe API can be used as part of a broader process that is performed on many, if not all, pages of a website. The API is used to analyze web page content and return a JSON object that lists any accessibility violations found. Here is how to get started:

1. Load page in testing system
2. Optionally, set configuration options for the javascript API (`axe.configure`)
3. Call analyze javascript API (`axe.a11yCheck`)
4. Either assert against results or save them for later processing


## Section 2: API Reference

### Overview

The aXe APIs are provided in the javascript file axe.js. It must be included in the web page under test. Parameters are sent as javascript function parameters. Results are returned in JSON format.

### API Notes

* A Rule test is made up of sub-tests. Each sub-test is returned in an array of 'checks'
* The `"helpUrl"` in the results object is a link to a broader description of the accessibility issue and suggested remediation. All of links point to Deque University help pages and require a valid login to that system.

### API Name: axe.getRules

#### Purpose

To get information on all the rules in the system.

#### Description

Returns a list of all rules with their ID and description

#### Synopsis

`axe.getRules([Tag Name 1, Tag Name 2...]);`

#### Parameters

* `tags` - **optional** Array of tags used to filter returned rules.  If omitted, it will return all rules.

**Returns:** Array of rules that match the input filter with each entry having a format of `{ruleId: <id>, description: <desc>}`

The current set of tags supported are listed in the following table:

| Tag Name           | Accessibility Standard                |
|--------------------|:-------------------------------------:|
| `wcag2a`           | WCAG 2.0 Level A                      |
| `wcag2aa`          | WCAG 2.0 Level AA                     |
| `section508`       | Section 508                           |
| `best-practice`    | Best practices endorsed by Deque      |


#### Example 1

In this example, we pass in the WCAG 2 A and AA tags into `axe.getRules` to retrieve only those rules. The function call returns an array of rules.

**Call:** `axe.getRules(['wcag2aa', 'wcag2a']);`

**Returned Data:**

```javascript
[
  { ruleId: "area-alt", description: "Checks the <area> elements of image…" },
  { ruleId: "aria-allowed-attr", description: "Checks all attributes that start…" },
  { ruleId: "aria-required-attr", description: "Checks all elements that contain…" },
  …
]
```

### API Name: axe.configure

#### Purpose

To configure the format of the data for the current session

#### Description

User specifies the format of the JSON structure passed to the callback of `axe.a11yCheck`

#### Synopsis

```javascript
axe.configure({ reporter: "option" });
```

#### Parameters

* `configurationOptions` - Options object; where the valid name, value pairs are:
  * `reporter` - Used to set the output format that the axe.a11yCheck function will pass to the callback function
     * `v1` to use the previous version's format: `axe.configure({ reporter: "v1" });`
     * `v2` to use the current version's format: `axe.configure({ reporter: "v2" });`

**Returns:** Nothing


### API Name: axe.a11yCheck

#### Purpose

Analyze currently loaded page

#### Description

Runs a number of rules against the provided HTML page and returns the resulting issue list

#### Synopsis

```javascript
axe.a11yCheck(context, options, callback);
```

#### Parameters

* [`context`](#context-parameter): Defines the scope of the analysis - the part of the DOM that you would like to analyze. This will typically be the `document` or a specific selector such as class name, ID, selector, etc.
* [`options`](#options-parameter): (optional) Set of options passed into rules or checks.
* [`callback`](#callback-parameter): The callback function which receives on the [results object](#results-object) as a parameter when analysis is complete

##### Context Parameter

The context object can be passed one of the following:

1. An element reference that represents the portion of the document that must be analyzed
	* Example: To limit analysis to the `<div id="content">` element: `document.getElementById("content")`
2. A CSS selector that selects the portion(s) of the document that must be analyzed. This includes:
	*  A CSS selector as a class name  (e.g. `.classname`)
	*  A CSS selector as a node name (e.g. `div`)
	*  A CSS selector of an element id (e.g. `#tag`)
3. An include-exclude object (see below)

###### Include-Exclude Object

The include exclude object is a JSON object with two attributes: include and exclude. Either include or exclude is required.  If only `exclude` is specified; include will default to the entire `document`.

* A node, or
* An array of arrays of CSS selectors

In most cases, the component arrays will contain only one CSS selector. Multiple CSS selectors are only required if you want to include or exclude regions of a page that are inside iframes (or iframes within iframes within iframes). In this case, the first n-1 selectors are selectors that select the iframe(s) and the nth selector, selects the region(s) within the iframe.

###### Context Parameter Examples

1. Include the first item in the `$fixture` NodeList but exclude its first child

	```javascript
	{
	  include: $fixture[0],
	  exclude: $fixture[0].firstChild
	}
	```
2. Include the element with the ID of `fix` but exclude any `div`s within it

	```javascript
	{
	  include: [['#fix']],
	  exclude: [['#fix div']]
	}
	```
3. Include the whole document except any structures whose parent contains the class `exclude1` or `exclude2`

	```javascript
	{
	  exclude: [['.exclude1'], ['.exclude2']]
	}
	```

##### Options Parameter

The options parameter is flexible way to configure how `a11yCheck` operates. The different modes of operation are:

* Run all rules corresponding to one of the accessibility standards
* Run all rules defined in the system, except for the list of rules specified
* Run a specific set of rules provided as a list of rule ids

###### Options Parameter Examples

1. Run only Rules for an accessibility standard

	There are certain standards defined that can be used to select a set of rules. The defined standards and tag string are defined as follows:

	| Tag Name           | Accessibility Standard                |
	|--------------------|:-------------------------------------:|
	| `wcag2a`           | WCAG 2.0 Level A                      |
	| `wcag2aa`          | WCAG 2.0 Level AA                     |
	| `section508`       | Section 508                           |
	| `best-practice`    | Best practices endorsed by Deque      |

	To run only WCAG 2.0 Level A rules, specify `options` as:

	```javascript
	{
	  runOnly: {
		  type: "tag",
		  values: ["wcag2a"]
		}
	}
	```

	To run both WCAG 2.0 Level A and Level AA rules, you must specify both `wcag2a` and `wcag2aa`:

	```javascript
	{
	  runOnly: {
	    type: "tag",
	    values: ["wcag2a", "wcag2aa"]
	  }
	}
	```

2. Run only a specified list of Rules

	If you only want to run certain rules, specify options as:

	```javascript
	{
	  runOnly: {
	    type: "rule",
	    values: [ "ruleId1", "ruleId2", "ruleId3" ]
	  }
	}
	```

	This example will only run the rules with the id of `ruleId1`, `ruleId2`, and `ruleId3`. No other rule will run.

2. Run all enabled Rules except for a list of rules

	The default operation for a11yCheck is to run all WCAG 2.0 Level A and Level AA rules. If certain rules should be disabled from being run, specify `options` as:
	```javascript
	{
	  "rules": {
	    "color-contrast": { enabled: false },
	    "valid-lang": { enabled: false }
	  }
	}
	```

	This example will disable the rules with the ids of `color-contrast` and `valid-lang`. All other rules will run. The list of valid rule ids is specified in the section below.

##### Callback Parameter

The callback parameter is a function that will be called when the asynchronous `axe.a11yCheck` function completes. The callback function is passed a single parameter - the results object of the `axe.a11yCheck` call.


#### Results Object

The callback function passed in as the third parameter of `axe.allyCheck` runs on the results object. This object has two components – a passes array and a violations array.  The passes array keeps track of all the passed tests, along with detailed information on each one. This leads to more efficient testing, especially when used in conjunction with manual testing, as the user can easily find out what tests have already been passed. Similarly, the violations array keeps track of all the failed tests, along with detailed information on each one.

###### `url`

The URL of the page that was tested.

###### `timestamp`

The date and time that analysis was completed.

###### `passes` and `violations` array

* `description` - Text string that describes what the rule does
* `help` - Help text that describes the test that was performed
* `helpUrl` - URL that provides more information about the specifics of the violation. Links to a page on the Deque University site.
* `id` - Unique identifier for the rule; [see the list of rules](rule-descriptions.md)
* `impact` - How serious the violation is. Can be one of "minor", "moderate", "serious", or "critical" if the Rule failed or `null` if the check passed
* `tags` - Array of tags that this rule is assigned. These tags can be used in the option structure to select which rules are run ([see `axe.allyCheck` parameters below for more information](#a11ycheck-parameters)).
* `nodes` - Array of all elements the Rule tested
	* `html` - Snippet of HTML of the Element
	* `impact` - How serious the violation is. Can be one of "minor", "moderate", "serious", or "critical" if the test failed or `null` if the check passed
	* `target` - Array of selectors that has each element correspond to one level of iframe or frame. If there is one iframe or frame, there should be two entries in `target`. If there are three iframe levels, there should be four entries in `target`.
	* `any` - Array of checks that were made where at least one must have passed. Each entry in the array contains:
		* `id` - Unique identifier for this check. Check ids may be the same as Rule ids
		* `impact` - How serious this particular check is. Can be one of "minor", "moderate", "serious", or "critical". Each check that is part of a rule can have different impacts. The highest impact of all the checks that fail is reported for the rule
		* `message` - Description of why this check passed or failed
		* `data` - Additional information that is specific to the type of Check which is optional. For example, a color contrast check would include the foreground color, background color, contrast ratio, etc.
		* `relatedNodes` - Optional array of information about other nodes that are related to this check. For example, a duplicate id check violation would list the other selectors that had this same duplicate id. Each entry in the array contains the following information:
			* `target` - Array of selectors for the related node
			* `html` - HTML source of the related node
	* `all` - Array of checks that were made where all must have passed. Each entry in the array contains the same information as the 'any' array
	* `none` - Array of checks that were made where all must have not passed. Each entry in the array contains the same information as the 'any' array

#### Example 2

In this example, we will pass the selector for the entire document, pass no options, which means all enabled rules will be run, and have a simple callback function that logs the entire results object to the console log:

```javascript
axe.a11yCheck(document, function(results) {
  console.log(results);
});
```

###### `passes`

* `passes[0]`
  ...
  * `help` - `"Elements must have sufficient color contrast"`
  * `helpURL` - `"https://dequeuniversity.com/courses/html-css/visual-layout/color-contrast"`
  * `id` - `"color-contrast"`
		* `nodes`
			* `target[0]` - `"#js_off-canvas-wrap > .inner-wrap >.kinja-title.proxima.js_kinja-title-desktop"`

* `passes[1]`
   ...

###### `violations`

* `violations[0]`
  * `help` - `"<button> elements must have alternate text"`
  * `helpURL` - `"https://dequeuniversity.com/courses/html-css/forms/form-labels#id84_example_button"`
  * `id` - `"button-name"`
		* `nodes`
			* `target[0]` - `"post_5919997 > .row.content-wrapper > .column > span > iframe"`
			* `target[1]` - `"#u_0_1 > .pluginConnectButton > .pluginButtonImage > button"`

* `violations[1]` ...


##### `passes` Results Array

In the example above, the `passes` array contains two entries that correspond to the two rules tested. The first element in the array describes a color contrast check. It relays the information that a list of nodes was checked and subsequently passed. The `help`, `helpUrl`, and `id` fields are returned as expected for each of the entries in the `passes` array. The `target` array has one element in it with a value of

`#js_off-canvas-wrap > .inner-wrap >.kinja-title.proxima.js_kinja-title-desktop`

This indicates that the element selected by the entry in `target[0]` was checked for the color contrast rule and that it passed the test.

Each subsequent entry in the passes array has the same format, but will detail the different rules that were run as part of this call to `axe.a11yCheck()`.

##### `violations` Results Array

The array of `violations` contains one entry; this entry describes a test that check if buttons have valid alternate text (button-name).  This first entry in the array has the `help`, `helpUrl` and `id` fields returned as expected.

The `target` array demonstrates how we specify the selectors when the node specified is inside of an `iframe` or `frame`. The first element in the `target` array - `target[0]` - specifies the selector to the `iframe` that contains the button. The second element in the `target` array - `target[1]` -  specifies the selector to the actual button, but starting from inside the iframe selected in `target[0]`.

Each subsequent entry in the violations array has the same format, but will detail the different rules that were run that generated accessibility violations as part of this call to `axe.a11yCheck()`.


#### Example 3

In this example, we pass the selector for the entire document, enable two additional best practice rules, and have a simple callback function that logs the entire results object to the console log:

```javascript
axe.a11yCheck(document, {
  rules: {
    "heading-order": { enabled: true },
    "label-title-only": { enabled: true }
  }
}, function(results) {
  console.log(results);
});
```


## Section 3: Example Reference

This package contains examples for [jasmine](examples/jasmine), [mocha](examples/mocha), [phantomjs](examples/phantomjs), [qunit](examples/qunit), [selenium using javascript](examples/selenium), and [generating HTML from the violations array](examples/html-handlebars.md). Each of these examples is in the [doc/examples](examples) folder. In each folder, there is a README.md file which contains specific information about each example.
