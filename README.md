
Pa11y
=====

Pa11y is your automated accessibility testing pal. It runs [HTML CodeSniffer][sniff] from the command line for programmatic accessibility reporting.

[![NPM version][shield-npm]][info-npm]
[![Node.js version support][shield-node]][info-node]
[![Build status][shield-build]][info-build]
[![Dependencies][shield-dependencies]][info-dependencies]
[![LGPL-3.0 licensed][shield-license]][info-license]

On the command line:

```sh
pa11y http://example.com/
```

In JavaScript:

```js
const pa11y = require('pa11y');

pa11y('http://example.com/').then((results) => {
    // Do something with the results
});
```

Need a GUI? Try [Koa11y](https://open-indy.github.io/Koa11y/)!

---

## Latest news from Pa11y

We're pleased to announce the Pa11y 5.0 beta is now available! We're switching from PhantomJS to Headless Chrome, as well as many other changes. See the [migration guide][migration-5] for further details.

If you'd like to try out the Pa11y 5.0 beta you can do so with

`npm install -g pa11y@beta`

Feedback is greatly appreciated 😊

---


Table Of Contents
-----------------

- [Requirements](#requirements)
- [Command-Line Interface](#command-line-interface)
- [JavaScript Interface](#javascript-interface)
- [Configuration](#configuration)
- [Actions](#actions)
- [Examples](#examples)
- [Common Questions and Troubleshooting](#common-questions-and-troubleshooting)
- [Tutorials and articles](#tutorials-and-articles)
- [Contributing](#contributing)
- [Support and Migration](#support-and-migration)
- [License](#license)


Requirements
------------

Pa11y requires [Node.js][node] 8+ to run. If you need support for older versions of Node.js, then please use [Pa11y 4.x][4.x].

### OS X

On a Mac, you can install the required dependency with [Homebrew][brew]:

```sh
$ brew install node
```

Alternatively download pre-built packages from the [Node.js][node] website.

### Linux

Depending on your flavour of Linux, you should be able to use a package manager to install the required dependency. Alternatively download pre-built packages from the [Node.js][node] website.

### Windows

On Windows 10, download a pre-built package from the [Node.js][node] website. Pa11y will be usable via the bundled Node.js application as well as the Windows command prompt.

Windows 7 and below users approach with caution – we've been able to get Pa11y running but only after installing Visual Studio and the Windows SDK (as well as Git, and Python). The [Windows installation instructions for node-gyp][windows-install] are a good place to start. If you have had a better experience than this then please do share!


Command-Line Interface
----------------------

Install Pa11y globally with [npm][npm]:

```
npm install -g pa11y
```

This installs the `pa11y` command-line tool:

```
Usage: pa11y [options] <url>

  Options:

    -V, --version                  output the version number
    -n, --environment              output details about the environment Pa11y will run in
    -s, --standard <name>          the accessibility standard to use: Section508, WCAG2A, WCAG2AA (default), WCAG2AAA
    -r, --reporter <reporter>      the reporter to use: cli (default), csv, json
    -l, --level <level>            the level of issue to fail on (exit with code 2): error, warning, notice
    -T, --threshold <number>       permit this number of errors, warnings, or notices, otherwise fail with exit code 2
    -i, --ignore <ignore>          types and codes of issues to ignore, a repeatable value or separated by semi-colons
    --include-notices              Include notices in the report
    --include-warnings             Include warnings in the report
    -R, --root-element <selector>  a CSS selector used to limit which part of a page is tested
    -E, --hide-elements <hide>     a CSS selector to hide elements from testing, selectors can be comma separated
    -c, --config <path>            a JSON or JavaScript config file
    -t, --timeout <ms>             the timeout in milliseconds
    -w, --wait <ms>                the time to wait before running tests in milliseconds
    -d, --debug                    output debug messages
    -S, --screen-capture <path>    a path to save a screen capture of the page to
    -A, --add-rule <rule>          WCAG 2.0 rules to include, a repeatable value or separated by semi-colons
    -h, --help                     output usage information
```

### Running Tests

Run an accessibility test against a URL:

```
pa11y http://example.com
```

Run an accessibility test against a file (absolute paths only, not relative):

```
pa11y ./path/to/your/file.html
```

Run a test with CSV reporting and save to a file:

```
pa11y --reporter csv http://example.com > report.csv
```

Run Pa11y with the Section508 ruleset:

```
pa11y --standard Section508 http://example.com
```

### Exit Codes

The command-line tool uses the following exit codes:

  - `0`: Pa11y ran successfully, and there are no errors
  - `1`: Pa11y failed run due to a technical fault
  - `2`: Pa11y ran successfully but there are errors in the page

By default, only accessibility issues with a type of `error` will exit with a code of `2`. This is configurable with the `--level` flag which can be set to one of the following:

  - `error`: exit with a code of `2` on errors only, exit with a code of `0` on warnings and notices
  - `warning`: exit with a code of `2` on errors and warnings, exit with a code of `0` on notices
  - `notice`: exit with a code of `2` on errors, warnings, and notices
  - `none`: always exit with a code of `0`

### Command-Line Configuration

The command-line tool can be configured with a JSON file as well as arguments. By default it will look for a `pa11y.json` file in the current directory, but you can change this with the `--config` flag:

```
pa11y --config ./path/to/config.json http://example.com
```

For more information on configuring Pa11y, see the [configuration documentation](#configuration).

### Ignoring

The ignore flag can be used in several different ways. Separated by semi-colons:

```
pa11y --ignore "issue-code-1;issue-code-2" http://example.com
```

or by using the flag multiple times:

```
pa11y --ignore issue-code-1 --ignore issue-code-2 http://example.com
```

Pa11y can also ignore notices, warnings, and errors up to a threshold number. This might be useful if you're using CI and don't want to break your build. The following example will return exit code 0 on a page with 9 errors, and return exit code 2 on a page with 10 or more errors.

```
pa11y --threshold 10 http://example.com
```

### Reporters

The command-line tool can report test results in a few different ways using the `--reporter` flag. The built-in reporters are:

  - `cli`: output test results in a human-readable format
  - `csv`: output test results as comma-separated values
  - `json`: output test results as a JSON array

The Pa11y team maintain some additional reporters which can be installed separately via `npm`:

  - `html`: output test results in a self-contained HTML format (`npm install pa11y-reporter-html`)
  - `tsv`: output test results as tab-separated values (`npm install pa11y-reporter-tsv`)

(TODO actually publish the `html` and `tsv` reporters, so that the above isn't lying :grin:)

You can also write and publish your own reporters. Pa11y looks for reporters in your `node_modules` folder (with a naming pattern), and the current working directory. The first reporter found will be loaded. So with this command:

```
pa11y --reporter rainbows http://example.com
```

The following locations will be checked:

```
<cwd>/node_modules/pa11y-reporter-rainbows
<cwd>/rainbows
```

A Pa11y reporter _must_ export a property named `supports`. This is a [semver range] (as a string) which indicates which versions of Pa11y the reporter supports:

```js
exports.supports = '^5.0.0';
```

A reporter should export the following methods, which should all return strings. If your reporter needs to perform asynchronous operations, then it may return a promise which resolves to a string:

```js
begin(); // Called when pa11y starts
error(message); // Called when a technical error is reported
debug(message); // Called when a debug message is reported
info(message); // Called when an information message is reported
results(results); // Called with the results of a test run
```


JavaScript Interface
--------------------

Install Pa11y with [npm][npm] or add to your `package.json`:

```
npm install pa11y
```

Require Pa11y:

```js
const pa11y = require('pa11y');
```

Run Pa11y against a URL, the `pa11y` function returns a [Promise]:

```js
pa11y('http://example.com/').then((results) => {
    // Do something with the results
});
```

Pa11y can also be run with [some options](#configuration):

```js
pa11y('http://example.com/', {
    // Options go here
}).then((results) => {
    // Do something with the results
});
```

Pa11y resolves with a `results` object, containing details about the page and accessibility issues from HTML CodeSniffer. It looks like this:

```js
{
    documentTitle: 'The title of the page that was tested',
    pageUrl: 'The URL that Pa11y was run against',
    issues: [
        {
            code: 'WCAG2AA.Principle1.Guideline1_1.1_1_1.H30.2',
            context: '<a href="http://example.com/"><img src="example.jpg" alt=""/></a>',
            message: 'Img element is the only content of the link, but is missing alt text. The alt text should describe the purpose of the link.',
            selector: 'html > body > p:nth-child(1) > a',
            type: 'error',
            typeCode: 1
        }
        // more issues appear here
    ]
}
```

### Transforming the Results

If you wish to transform these results with the command-line reporters, then you can do so in your code by requiring them in. The `csv`, `tsv`, `html`, `json`, and `markdown` reporters all expose a `process` method:

```js
// Assuming you've already run tests, and the results
// are available in a `results` variable:
const htmlReporter = require('pa11y/reporter/html');
const html = htmlReporter.process(results, url);
```

### Async/Await

Because Pa11y is promise based, you can use `async` functions and the `await` keyword:

```js
async function runPa11y() {
    try {
        const results = await pa11y('http://example.com/');
        // Do something with the results
    } catch (error) {
        // Handle the error
    }
}

runPa11y();
```

### Callback Interface

If you would rather use callbacks than promises or `async`/`await`, then Pa11y supports this. This interface should be considered legacy, however, and may not appear in the next major version of Pa11y:

```js
pa11y('http://example.com/', (error, results) => {
    // Do something with the results or handle the error
});
```

### Validating Actions

Pa11y exposes a function which allows you to validate [action](#actions) strings before attempting to use them.

This function accepts an action string and returns a boolean indicating whether it matches one of the actions that Pa11y supports:

```js
pa11y.validateAction('click element #submit');  // true
pa11y.validateAction('open the pod bay doors'); // false
```


Configuration
-------------

Pa11y has lots of options you can use to change the way Headless Chrome runs, or the way your page is loaded. Options can be set either as a parameter on the `pa11y` function or in a config file used by the command-line interface.

Below is a reference of all the options that are available:

### `actions` (array) **BETA**

Actions to be run before Pa11y tests the page. There are quite a few different actions available in Pa11y, the [Actions documentation](#actions) outlines each of them.

**Note:** actions are currently in a beta state and the API may change while we gather feedback.

```js
pa11y('http://example.com/', {
    actions: [
        'set field #username to exampleUser',
        'set field #password to password1234',
        'click element #submit',
        'wait for path to be /myaccount'
    ]
});
```

Defaults to an empty array.

### `chromeLaunchConfig` (object)

Launch options for the Headless Chrome instance. [See the Puppeteer documentation for more information][puppeteer-launch].

```js
pa11y('http://example.com/', {
    chromeLaunchConfig: {
        executablePath: '/path/to/Chrome',
        ignoreHTTPSErrors: false
    }
});
```

Defaults to:

```js
{
    ignoreHTTPSErrors: true
}
```

### `headers` (object)

A key-value map of request headers to send when testing a web page.

```js
pa11y('http://example.com/', {
    headers: {
        Cookie: 'foo=bar'
    }
});
```

Defaults to an empty object.

### `hideElements` (string)

A CSS selector to hide elements from testing, selectors can be comma separated. Elements matching this selector will be hidden from testing by styling them with `visibility: hidden`.

```js
pa11y('http://example.com/', {
    hideElements: '.advert, #modal, div[aria-role=presentation]'
});
```

### `ignore` (array)

An array of result codes and types that you'd like to ignore. You can find the codes for each rule in the console output and the types are `error`, `warning`, and `notice`. Note: `warning` and `notice` messages are ignored by default.

```js
pa11y('http://example.com/', {
    ignore: [
        'WCAG2AA.Principle3.Guideline3_1.3_1_1.H57.2'
    ]
});
```

Defaults to an empty array.

### `includeNotices` (boolean)

Whether to include results with a type of `notice` in the Pa11y report. Issues with a type of `notice` are not directly actionable and so they are excluded by default. You can include them by using this option:

```js
pa11y('http://example.com/', {
    includeNotices: true
});
```

Defaults to `false`.

### `includeWarnings` (boolean)

Whether to include results with a type of `warning` in the Pa11y report. Issues with a type of `warning` are not directly actionable and so they are excluded by default. You can include them by using this option:

```js
pa11y('http://example.com/', {
    includeWarnings: true
});
```

Defaults to `false`.

### `log` (object)

An object which implements the methods `debug`, `error`, and `info` which will be used to report errors and test information.

```js
pa11y('http://example.com/', {
    log: {
        debug: console.log,
        error: console.error,
        info: console.info
    }
});
```

Each of these defaults to an empty function.

### `method` (string)

The HTTP method to use when running Pa11y.

```js
pa11y('http://example.com/', {
    method: 'POST'
});
```

Defaults to `GET`.

### `postData` (string)

The HTTP POST data to send when running Pa11y. This should be combined with a `Content-Type` header. E.g to send form data:

```js
pa11y('http://example.com/', {
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
    },
    method: 'POST',
    postData: 'foo=bar&bar=baz'
});
```

Or to send JSON data:

```js
pa11y('http://example.com/', {
    headers: {
        'Content-Type': 'application/json'
    },
    method: 'POST',
    postData: '{"foo": "bar", "bar": "baz"}'
});
```

Defaults to `null`.

### `rootElement` (element)

The root element for testing a subset of the page opposed to the full document.

```js
pa11y('http://example.com/', {
    rootElement: '#main'
});
```
Defaults to `null`, meaning the full document will be tested.

### `rules` (array)

An array of WCAG 2.0 guidelines that you'd like to include to the current standard. Note: These won't be applied to `Section508` standard. You can find the codes for each guideline in the [HTML Code Sniffer WCAG2AAA ruleset][htmlcs-wcag2aaa-ruleset].

```js
pa11y('http://example.com/', {
    rules: [
        'Principle1.Guideline1_3.1_3_1_AAA'
    ]
});
```

### `screenCapture` (string)

A file path to save a screen capture of the tested page to. The screen will be captured immediately after the Pa11y tests have run so that you can verify that the expected page was tested.

```js
pa11y('http://example.com/', {
    screenCapture: `${__dirname}/my-screen-capture.png`
});
```

Defaults to `null`, meaning the screen will not be captured.

### `standard` (string)

The accessibility standard to use when testing pages. This should be one of `Section508`, `WCAG2A`, `WCAG2AA`, or `WCAG2AAA`.

```js
pa11y('http://example.com/', {
    standard: 'Section508'
});
```

Defaults to `WCAG2AA`.

### `timeout` (number)

The time in milliseconds that a test should be allowed to run before calling back with a timeout error.

```js
pa11y('http://example.com/', {
    timeout: 500
});
```

Defaults to `30000`.

### `userAgent` (string)

The `User-Agent` header to send with Pa11y requests. This is helpful to identify Pa11y in your logs.

```js
pa11y('http://example.com/', {
    userAgent: 'A11Y TESTS'
});
```

Defaults to `pa11y/<version>`.

### `viewport` (object)

The viewport configuration. This can have any of the properties supported by the [puppeteer `setViewport` method][puppeteer-viewport].

```js
pa11y('http://example.com/', {
    viewport: {
        width: 320,
        height: 480,
        deviceScaleFactor: 2,
        isMobile: true
    }
});
```

Defaults to:

```js
{
    width: 1280,
    height: 1024
}
```

### `wait` (number)

The time in milliseconds to wait before running HTML CodeSniffer on the page.

```js
pa11y('http://example.com/', {
    wait: 500
});
```

Defaults to `0`.


Actions
-------

Actions are additional interactions that you can make Pa11y perform before the tests are run. They allow you to do things like click on a button, enter a value in a form, wait for a redirect, or wait for the URL fragment to change:

```js
pa11y('http://example.com/', {
    actions: [
        'click element #tab-1',
        'wait for element #tab-1-content to be visible',
        'set field #fullname to John Doe',
        'check field #terms-and-conditions',
        'uncheck field #subscribe-to-marketing',
        'wait for fragment to be #page-2',
        'wait for path to not be /login',
        'wait for url to be https://example.com/'
    ]
});
```

Below is a reference of all the available actions and what they do on the page. Some of these take time to complete so you may need to increase the `timeout` option if you have a large set of actions.

### Click Element

This allows you to click an element by passing in a CSS selector. This action takes the form `click element <selector>`. E.g.

```js
pa11y('http://example.com/', {
    actions: [
        'click element #tab-1'
    ]
});
```
You can use any valid [query selector](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector), including classes and types.

### Set Field Value

This allows you to set the value of a text-based input or select box by passing in a CSS selector and value. This action takes the form `set field <selector> to <value>`. E.g.

```js
pa11y('http://example.com/', {
    actions: [
        'set field #fullname to John Doe'
    ]
});
```

### Check/Uncheck Field

This allows you to check or uncheck checkbox and radio inputs by passing in a CSS selector. This action takes the form `check field <selector>` or `uncheck field <selector>`. E.g.

```js
pa11y('http://example.com/', {
    actions: [
        'check field #terms-and-conditions',
        'uncheck field #subscribe-to-marketing'
    ]
});
```

### Wait For Fragment/Path/URL

This allows you to pause the test until a condition is met, and the page has either a given fragment, path, or URL. This will wait until Pa11y times out so it should be used after another action that would trigger the change in state. You can also wait until the page does **not** have a given fragment, path, or URL using the `to not be` syntax. This action takes one of the forms:

  - `wait for fragment to be <fragment>` (including the preceding `#`)
  - `wait for fragment to not be <fragment>` (including the preceding `#`)
  - `wait for path to be <path>` (including the preceding `/`)
  - `wait for path to not be <path>` (including the preceding `/`)
  - `wait for url to be <url>`
  - `wait for url to not be <url>`

E.g.

```js
pa11y('http://example.com/', {
    actions: [
        'click element #login-link',
        'wait for path to be /login'
    ]
});
```

### Wait For Element State

This allows you to pause the test until an element on the page (matching a CSS selector) is either added, removed, visible, or hidden. This will wait until Pa11y times out so it should be used after another action that would trigger the change in state. This action takes one of the forms:

  - `wait for element <selector> to be added`
  - `wait for element <selector> to be removed`
  - `wait for element <selector> to be visible`
  - `wait for element <selector> to be hidden`

E.g.

```js
pa11y('http://example.com/', {
    actions: [
        'click element #tab-2',
        'wait for element #tab-1 to be hidden'
    ]
});
```


Examples
--------

### Basic Example

Run Pa11y on a URL and output the results. [See the example](example/basic/index.js).

### Multiple Example

Run Pa11y on multiple URLs at once and output the results. [See the example](example/multiple/index.js).

### Actions Example

Step through some actions before Pa11y runs. This example logs into a fictional site then waits until the account page has loaded before running Pa11y. [See the example](example/actions/index.js).


Common Questions and Troubleshooting
------------------------------------

See our [Troubleshooting guide](TROUBLESHOOTING.md) to get the answers to common questions about Pa11y, along with some ideas to help you troubleshoot any problems.


Tutorials and articles
------------------------------------

Here are some useful articles written by Pa11y users and contributors:

- [Accessibility Testing with Pa11y](https://bitsofco.de/pa11y/)
- [Using actions in Pa11y](http://hollsk.co.uk/posts/view/using-actions-in-pa11y)
- [Introduction to Accessibility Testing With Pa11y](http://cruft.io/posts/accessibility-testing-with-pa11y/)


Contributing
------------

There are many ways to contribute to Pa11y, we cover these in the [contributing guide](CONTRIBUTING.md) for this repo.

If you're ready to contribute some code, clone this repo locally and commit your code on a new branch.

Please write unit tests for your code, and check that everything works by running the following before opening a <abbr title="pull request">PR</abbr>:

```sh
make ci
```

You can also run verifications and tests individually:

```sh
make verify              # Verify all of the code (ESLint)
make test                # Run all tests
make test-unit           # Run the unit tests
make test-unit-coverage  # Run the unit tests with coverage
make test-integration    # Run the integration tests
```


Support and Migration
---------------------

Pa11y major versions are normally supported for 6 months after their last minor release. This means that patch-level changes will be added and bugs will be fixed. The table below outlines the end-of-support dates for major versions, and the last minor release for that version.

We also maintain a [migration guide](MIGRATION.md) to help you migrate.

| :grey_question: | Major Version | Last Minor Release | Node.js Versions | Support End Date |
| :-------------- | :------------ | :----------------- | :--------------- | :--------------- |
| :heart:         | 5             | N/A                | 8+               | N/A              |
| :heart:         | 4             | N/A                | 4–8              | N/A *            |
| :skull:         | 3             | 3.8                | 0.12–6           | 2016-12-05       |
| :skull:         | 2             | 2.4                | 0.10–0.12        | 2016-10-16       |
| :skull:         | 1             | 1.7                | 0.10             | 2016-06-08       |

If you're opening issues related to these, please mention the version that the issue relates to.

\* Pa11y 4's support end date will be reviewed at the end of this period based on community feedback.


License
-------

Pa11y is licensed under the [Lesser General Public License (LGPL-3.0)][info-license].<br/>
Copyright &copy; 2013–2017, Team Pa11y and contributors



[1.0-json-reporter]: https://github.com/pa11y/reporter-1.0-json
[4.x]: https://github.com/pa11y/pa11y/tree/4.x
[async]: https://github.com/caolan/async
[brew]: http://mxcl.github.com/homebrew/
[migration-5]: https://github.com/pa11y/pa11y/blob/5.x/MIGRATION.md#migrating-from-40-to-50
[node]: http://nodejs.org/
[npm]: https://www.npmjs.com/
[promise]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise
[puppeteer-launch]: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerlaunchoptions
[puppeteer-viewport]: https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetviewportviewport
[semver range]: https://github.com/npm/node-semver#ranges
[sidekick-proposal]: https://github.com/pa11y/sidekick/blob/master/PROPOSAL.md
[sniff]: http://squizlabs.github.com/HTML_CodeSniffer/
[sniff-issue]: https://github.com/squizlabs/HTML_CodeSniffer/issues/109
[windows-install]: https://github.com/TooTallNate/node-gyp#installation

[info-dependencies]: https://gemnasium.com/pa11y/pa11y
[info-license]: LICENSE
[info-node]: package.json
[info-npm]: https://www.npmjs.com/package/pa11y
[info-build]: https://travis-ci.org/pa11y/pa11y
[shield-dependencies]: https://img.shields.io/gemnasium/pa11y/pa11y.svg
[shield-license]: https://img.shields.io/badge/license-LGPL%203.0-blue.svg
[shield-node]: https://img.shields.io/badge/node.js%20support-4–8-brightgreen.svg
[shield-npm]: https://img.shields.io/npm/v/pa11y.svg
[shield-build]: https://img.shields.io/travis/pa11y/pa11y/master.svg
