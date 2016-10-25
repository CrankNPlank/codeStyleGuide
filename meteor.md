# Meteor.JS - Coding Style Guide

The following articles and repos heavily influenced the development of this 
style guide:
* [Meteor.js Code Style](https://guide.meteor.com/code-style.html)

## Collections

Collections should be named as a plural noun, in 
[PascalCase](https://en.wikipedia.org/wiki/PascalCase). The name of the 
collection in the database (the first argument to the collection constructor) 
should be the same as the name of the JavaScript symbol.

```javascript
// Defining a collection
Lists = new Mongo.Collection('Lists');
```

Fields in the database should be camelCased, just like your JavaScript variable 
names

```javascript
// Inserting a document with camelCased field names
Widgets.insert({
  myFieldName: 'Hello, World!',
  otherFieldName: 'Goodbye, World!'
});
```

## Methods & Publications

Method and publication names should be camelCased, and namespaced to the module 
they are in:

```javascript
// In imports/api/todos/methods.js
updateText = new ValidatedMethod({
  name: 'todos.updateText',
  ...
});
```

Note that this code sample uses the 
[ValidatedMethod package recommended in the Methods article](https://guide.meteor.com/methods.html#validated-method). 
If you aren’t using that package, you can use the name as the property passed 
to `Meteor.methods`.

Here’s how this naming convention looks when applied to a publication:

```javascript
// Naming a publication
Meteor.publish('lists.public', function listsPublic() {
  ...
});
```

## Files, Exports, & Packages

Using ES2015 `import` and `export` features to manage code lets you better 
understand the dependencies between different parts of your code, and makes it 
easier to know where to look if you need to read the source code of a 
dependency.

Each file in your app should represent one logical module. Avoid having 
catch-all utility modules that export a variety of unrelated functions and 
symbols. Often, this can mean that it’s good to have one class, UI component, 
or collection per file, but there are cases where it is OK to make an 
exception, for example if you have a UI component with a small sub-component 
that isn’t used outside of that file.

When a file represents a single class or UI component, the file should be named 
the same as the thing it defines, with the same capitalization. So if you have 
a file named `ClickCounter.js` that exports a class:

```javascript
export default class ClickCounter { ... }
```

When you import it, it'll look like this:

```javascript
import ClickCounter from './ClickCounter.js';
```

*NOTE:* Imports use relative paths, and include the file extension at the end 
of the filename

### Atmosphere Packages

For [Atmosphere packages](https://guide.meteor.com/atmosphere-vs-npm.html), as 
the older pre-1.3 `api.export` syntax allowed more than one export per package, 
you’ll tend to see non-default exports used for symbols. For instance:

```javascript
// You will need to destructure here, as Meteor could export more symbols
import {Meteor} from 'meteor/meteor';

// This will not work
import Meteor from 'meteor/meteor';
```

## Templates & Components

Since Spacebars templates are always global, can’t be imported and exported as 
modules, and need to have names that are completely unique across the whole 
app, we recommend naming your Blaze templates with the full path to the 
namespace, separated by underscores. Underscores are a great choice in this 
case because then you can easily type the name of the template as one symbol 
in JavaScript.

```html
<template name="Lists_show">
  ...
</template>
```

If this template is a "smart" component that loads server data and accesses the 
router, append `_page` to the name:

```html
<template name="Lists_show_page">
  ...
</template>
```

Often when you are dealing with templates or UI components, you’ll have several 
closely coupled files to manage. They could be two or more of HTML, CSS, and 
JavaScript files. In this case, we recommend putting these together in the same 
directory with the same name:

```bash
# The Lists_show template from the Todos example app has 3 files:
show.html
show.js
show.less
```

The whole directory or path should indicate that these templates are related to 
the Lists module, so it’s not necessary to reproduce that information in the 
file name. Read more about directory structure below.

If you are writing your UI in React, you don’t need to use the underscore-split 
names because you can import and export your components using the JavaScript 
module system.

# Meteor.JS Application Structure

How to structure your Meteor app with ES2015 modules, ship code to the client 
and server, and split your code into multiple apps.

## ES2015 Modules

* Replaces CommonJS and AMD modules
* In ES2015, you can make variables available outside of a file using the 
  `export` keyword
  * To use the variables somewhere else, you must `import` them using the path 
  to the source
* Called *modules* because they represent a unit of reusable code
* Explicitly importing the modules and packages you use helps you write your 
  code in a modular way
  * Avoids the introduction of global symbols
  * Avoids "action at a distance"

To opt-in to the new module system, code must be placed inside the `imports/` 
directory in your application. More information is available in the `modules 
package README`. Automatically included with every Meteor app as part of the 
`ecmascript meta-package`.

### Using `import` & `export`

You can `import` not only JavaScript, but also CSS and HTML to control load 
order:

```javascript
// Import using relative path
import '../../api/lists/methods.js';

// Import module with index.js from absolute path
import '/imports/startup/client';

// Import Blaze compiled HTML from relative path
import './loading.html';

// Import CSS from absolute path
import '/imports/ui/style.css';
```

Meteor also supports the standard ES2015 modules `export` syntax:

```javascript
// Named export
export const listRenderHold = LaunchScreen.hold();

// Named export
export { Todos };

// Default export
export default Lists;

// Default export
export default new Collection('lists');
```

#### Importing From Packages

In Meteor, you can use the `import` syntax to load npm packages on the client 
or server and access the package’s exported symbols as you would with any other 
module. You can also import from Meteor Atmosphere packages, but the import 
path must be prefixed with `meteor/` to avoid conflict with the npm package 
namespace. For example, to import `moment` from npm and `HTTP` from Atmosphere:

```javascript
// Default npm importation
import moment from 'moment';

// Named importation from Atmosphere
import { HTTP } from 'meteor/http';
```

### Using `require`

In Meteor, `import` statements compile to CommonJS `require` syntax. However, 
as a convention we encourage you to use `import`.

With that said, in some situations you may need to call out to `require` 
directly. One notable example is when requiring client or server-only code 
from a common file. As `imports` must be at the top-level scope, you may not 
place them within an `if` statement, so you’ll need to write code like:

```javascript
if (Meteor.isClient) {
  require('./client-only-file.js');
}
```

Note that dynamic calls to `require()` (where the name being required can 
change at runtime) cannot be analyzed correctly and may result in broken 
client bundles.

If you need to require from an ES2015 module with a `default` export, you can 
grab the export with `require("package").default`.

Another situation where you’ll need to use `require` is in CoffeeScript files. 
As CS doesn’t support the `import` syntax yet, you should use `require`:

```javascript
{ FlowRouter } = require 'meteor/kadira:flow-router'
React = require 'react'
```

## File Structure

All application code should be placed inside the `imports/` directory. This 
means that the Meteor build system will only bundle and include that file if it 
is referenced from another file using an `import` (a.k.a. "lazy evaluation or 
loading").

Meteor will load all files outside of any directory named `imports/` in the 
application using the 
[default file load order](https://guide.meteor.com/structure.html#load-order) 
rules (also called “eager evaluation or loading”). It is recommended that you 
create *exactly two eagerly loaded files*, `client/main.js` and 
`server/main.js`, in order to define explicit entry points for both the client 
and the server. Meteor ensures that any file in any directory named `server/` 
will only be available on the server, and likewise for files in any directory 
named `client/`. This also precludes trying to `import` a file to be used on 
the server from any directory named `client/`, even if it is nested in an 
`imports/` directory, and vice versa for importing client files from `server/`.

These `main.js` files won’t do anything themselves, but they should import some 
**startup** modules which will run immediately, on client and server 
respectively, when the app loads. These modules should do any configuration 
necessary for the packages you are using in your app, and import the rest of 
your app’s code.

### Example Directory Layout

The following is an example application structure:

```bash
imports/
  startup/
    client/
      index.js       # import client startup through a single index entry point
      routes.js      # set up all routes in the app
      useraccounts-configuration.js # configure login templates
    server/
      fixtures.js    # fill the DB with example data on startup
      index.js       # import server startup through a single index entry point

  api/
    lists/           # a unit of domain logic
      server/
        publications.js        # all list-related publications
        publications.tests.js  # tests for the list publications
      lists.js                 # definition of the Lists collection
      lists.tests.js           # tests for the behavior of that collection
      methods.js               # methods related to lists
      methods.tests.js         # tests for those methods

  ui/
    components/      # all reusable components in the application
                       # can be split by domain if there are many
    layouts/         # wrapper components for behaviour and visuals
    pages/           # entry points for rendering used by the router

client/
  main.js            # client entry point, imports all client code

server/
  main.js            # server entry point, imports all server code
```

#### Structuring Imports

* Put all code that runs when the app starts in an `imports/startup` directory
* Split data and business logic from UI rendering code
  * Use `imports/api` for data and business logic. Within `imports/api`, split 
  the code into directories based on the domain the for which the code is 
  providing an API - typically, this corresponds to the collections defined in 
  the app:
    * For example, above we have the `imports/api/lists` and 
    `imports/api/todos` domains.
      * Inside each directory, we define the collections, publications, and 
      methods used to manipulate the relevant domain data.
  * Use `imports/ui` for presentation/UI logic. Typically, it makes sense to 
  group files into directories based on the type of UI side code they define, 
  i.e. top level `pages`, wrapping `layouts`, or reusable `components`.
* For each module defined above, it makes sense to co-locate the various 
auxiliary files with the base JavaScript file. For instance, a Blaze UI 
component should have its template HTML, JavaScript logic, and CSS rules in the 
same directory. A JavaScript module with some business logic should be 
co-located with the unit tests for that module.

#### Startup Files

Some of your code isn’t going to be a unit of business logic or UI, it’s just 
some setup or configuration code that needs to run in the context of the app 
when it starts up. In the example app above, the 
`imports/startup/client/useraccounts-configuration.js` file configures the 
`useraccounts` login templates. The `imports/startup/client/routes.js` 
configures all of the routes and then imports **all** other code that is 
required on the client:

```javascript
import { FlowRouter } from 'meteor/kadira:flow-router';
import { BlazeLayout } from 'meteor/kadira:blaze-layout';
import { AccountsTemplates } from 'meteor/useraccounts:core';

// Import to load these templates
import '../../ui/layouts/app-body.js';
import '../../ui/pages/root-redirector.js';
import '../../ui/pages/lists-show-page.js';
import '../../ui/pages/app-not-found.js';

// Import to override accounts templates
import '../../ui/accounts/accounts-templates.js';

// Below here are the route definitions
```

Both files are then imported in `imports/startup/client/index.js`:

```javascript
import './useraccounts-configuration.js';
import './routes.js';
```

This makes it easy to then import all the client startup code with a single 
import as a module from our main eagerly loaded client entry point 
`client/main.js`:

```javascript
import '/imports/startup/client';
```

On the server, we use the same technique of importing all the startup code in 
`imports/startup/server/index.js`:

```javascript
// This defines a starting set of data to be loaded if the app is loaded with 
// an empty db.
import '../imports/startup/server/fixtures.js';

// This file configures the Accounts package to define the UI of the reset 
// password email.
import '../imports/startup/server/reset-password-email.js';

// Set up some rate limiting and other important security settings.
import '../imports/startup/server/security.js';

// This defines all the collections, publications and methods that the 
// application provides as an API to the client.
import '../imports/api/api.js';
```

Our main server entry point `server/main.js` then imports this startup module. 
You can see that here we don’t actually import any variables from these files - 
we just import them so that they execute in this order.

### Importing Meteor "pseudo-globals"

For backwards compatibility Meteor 1.3 still provides Meteor’s global 
namespacing for the Meteor core package as well as for other Meteor packages 
you include in your application. You can also still directly call functions 
such as `Meteor.publish`, as in previous versions of Meteor, without first 
importing them. However, it is recommended best practice that you first load 
all the Meteor “pseudo-globals” using the 
`import { Name } from 'meteor/package'` syntax before using them. For instance:

```javascript
import { Meteor } from 'meteor/meteor';
import { EJSON } from 'meteor/ejson';
```

## Default File Load Order

You may combine both eager loading and lazy loading using `import` in a single 
app. Any import statements are evaluated in the order they are listed in a file 
when that file is loaded and evaluated using these rules.

There are several load order rules. They are **applied sequentially** to all 
applicable files in the application, in the priority given below:
1. HTML template files are **always** loaded before everything else
2. Files beginning with `main.` are loaded last
3. Files inside **any** `lib/` directory are loaded next
4. Files with deeper paths are loaded next
5. Files are then loaded in alphabetical order of the entire path:
  ```bash
  nav.html
  main.html
  client/lib/methods.js
  client/lib/styles.js
  lib/feature/styles.js
  lib/collections.js
  client/feature-y.js
  feature-x.js
  client/main.js
  ```
  For example, the files above are arranged in the correct load order. 
  `main.html` is loaded second because HTML templates are always loaded first, 
  even if it begins with `main.`, since rule 1 has priority over rule 2. 
  However, it will be loaded after `nav.html` because rule 2 has priority over 
  rule 5.

  `client/lib/styles.js` and `lib/feature/styles.js` have identical load order 
  up to rule 4; however, since `client` comes before `lib` alphabetically, it 
  will be loaded first.

## Special Directories

* `imports/`
  * Not loaded anywhere
  * Files must be imported using `import`
* `node_modules/`
  * Not loaded anywhere
  * **node.js** packages installed into `node_modules/` directories must be 
  imported using `import`, or by using `Npm.depends` in `package.js`
* `client/`
  * Not loaded on the server
  * Similar to wrapping code in `if (Meteor.isClient) { ... }`
  * Automatically concatenated and minified when in production mode
  * In development mode, JS and CSS files are not minified (to make debugging 
  easier)
  * CSS files still combined into single file for dev/prod consistency
  * HTML files are scanned by Meteor for three top-level elements: `<head>`, 
  `<body>`, and `<template>`
    * Head and body sections are separately concatenated into a single head and 
    body, which are passed to the client on initial page load
* `server/`
  * Not loaded on the client
  * Similar to wrapping code in `if (Meteor.isServer) { ... }`
  * Any sensitive code not for client (e.g. passwords, authentication 
  mechanisms, etc.) should be kept in the `server/` directory
  * Meteor gathers all JS files, excluding anything under `client/`, `public/`, 
  and `private/` subdirectories, and loads them into a Node.js server instance
    * *In Meteor, server code runs in a single thread per request, not in the 
    asynchronous callback style typical of Node*
* `public/`
  * All files served as-is to the client
  * When referencing, do not include `public/` in the URL
    * Write the URL as if they were all in the top level
    * E.g. reference `public/bg.png` as `<img src="/bg.png" />`
    * Best place for `favicon.ico`, `robots.txt`, and similar files
* `private/`
  * Only accessible from server code
  * Can be loaded via the `Assets` API
  * Can be used for private data files, and any files in project you don't want 
  accesible from the outside
* `client/compatibility/`
  * For compatibility with JS libraries that rely on variables declared with 
  var at the top level, being exported as globals
  * Executed without being wrapped in a new variable scope
  * Executed before other client-side JS files
  * Recommended to use npm for 3rd party JS libraries and use `import` to 
  control when files are loaded
* `tests/`
  * Not loaded anywhere
  * Use only for test code you want to run using a test runner outside of 
  Meteor's built-in test tools
* Misc. directories also not loaded as part of app code
  * Files/directories whose names start with a dot, e.g. `.meteor` and `.git`
  * `packages/` - used for local packages
  * `cordova-build-override/` - used for 
  [advanced mobile build customizations](https://guide.meteor.com/mobile.html#advanced-build)
  * `programs` - for legacy reasons

## Files Outside Special Directories

All JavaScript files outside special directories are loaded on both the client 
and the server. Meteor provides the variables `Meteor.isClient` and 
`Meteor.isServer` so that your code can alter its behavior depending on whether 
it’s running on the client or the server.

CSS and HTML files outside special directories are loaded on the client only 
and cannot be used from server code.

## Splitting Into Multiple Apps

If you are writing a sufficiently complex system, there can come a time where 
it makes sense to split your code up into multiple applications. For example 
you may want to create a separate application for the administration UI (rather 
than checking permissions all through the admin part of your site, you can 
check once), or separate the code for the mobile and desktop versions of your 
app.

Another very common use case is splitting a worker process away from your main 
application so that expensive jobs do not impact the user experience of your 
visitors by locking up a single web server.

There are some advantages of splitting your application in this way:
1. Your client JavaScript bundle can be significantly smaller if you separate 
out code that a specific type of user will never use.
2. You can deploy the different applications with different scaling setups and 
secure them differently (for instance you might restrict access to your admin 
application to users behind a firewall).
3. You can allow different teams at your organization to work on the different 
applications independently.

However there are some challenges to splitting your code in this way that 
should be considered before jumping in.

### Sharing Code

The primary challenge is properly sharing code between the different 
applications you are building. The simplest approach to deal with this issue is 
to simply deploy the same application on different web servers, controlling the 
behavior via different settings. This approach allows you to easily deploy 
different versions with different scaling behavior but doesn’t enjoy most of 
the other advantages stated above.

If you want to create Meteor applications with separate code, you’ll have some 
modules that you’d like to share between them. If those modules are something 
the wider world could use, you should consider publishing them to a package 
system, either npm or Atmosphere, depending on whether the code is 
Meteor-specific or otherwise.

If the code is private, or of no interest to others, it typically makes sense 
to simply include the same module in both applications (you can do this with 
private npm modules). There are several ways to do this:
* a straightforward approach is simply to include the common code as a git 
submodule of both applications.
* alternatively, if you include both applications in a single repository, you 
can use symbolic links to include the common module inside both apps.

### Sharing Data

Another important consideration is how you’ll share the data between your 
different applications.

The simplest approach is to point both applications at the same `MONGO_URL` 
and allow both applications to read and write from the database directly. This 
works well thanks to Meteor’s support for reactivity through the database. When 
one app changes some data in MongoDB, users of any other app connected to the 
database will see the changes immediately thanks to Meteor’s livequery.

However, in some cases it’s better to allow one application to be the master 
and control access to the data for other applications via an API. This can help 
if you want to deploy the different applications on different schedules and 
need to be conservative about how the data changes.

The simplest way to provide a server-server API is to use Meteor’s built-in DDP 
protocol directly. This is the same way your Meteor client gets data from your 
server, but you can also use it to communicate between different applications. 
You can use DDP.connect() to connect from a “client” server to the master 
server, and then use the connection object returned to make method calls and 
read from publications.

### Sharing Accounts

If you have two servers that access the same database and you want 
authenticated users to make DDP calls across the both of them, you can use the 
**resume token** set on one connection to login on the other.

If your user has connected to server A, then you can use `DDP.connect()` to 
open a connection to server B, and pass in server A’s resume token to 
authenticate on server B. As both servers are using the same DB, the same 
server token will work in both cases. The code to authenticate looks like this:

```javascript
// This is server A's token as the default `Accounts` points at our server
const token = Accounts._storedLoginToken();

// We create a *second* accounts client pointing at server B
const app2 = DDP.connect('url://of.server.b');
const accounts2 = new AccountsClient({ connection: app2 });

// Now we can login with the token. Further calls to `accounts2` will 
// be authenticated
accounts2.loginWithToken(token);
```

You can see a proof of concept of this architecture in an 
[example repository](https://github.com/tmeasday/multi-app-accounts).











