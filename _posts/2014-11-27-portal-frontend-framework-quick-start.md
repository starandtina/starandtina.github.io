---
layout: post
title: "Portal Frontend Framework Quick Start"
description: ""
category: ['JavaScript', 'Frontend', 'js-dev']
tags: ['JavaScript', 'Frontend', 'js-dev']
---
{% include JB/setup %}

# Overview

This page describes the guidelines that front-end engineers develop and test the components using the UI framework of Portal and quickly familiar with our basic ideas behind the scene.

# Prerequisites
Installing the following tools before you continue.

* Download and install [Node.js](http://nodejs.org/) and [Ruby](https://www.ruby-lang.org/en/). Following installation, you should be able to invoke **node**, **npm**, **gem** on your command line. If desired, you may optionally use a tool such as **nvm**, **nave** or **rvm** to manage your Node.js installation.
* Download and install a [git client](http://git-scm.com/), if you don't already have one. Following installation, you should be able to invoke **git** on your command line.

If you are working on Mac platform, recommending install them using brew. 

```Shell
 brew install git node
 gem install gerrit-cli
```

# Setup Dev Env

## Step 1: Clone the Repo

```Shell
git clone https://github.com/starandtina/portal
```

## Step 2: Install Dependencies

```Shell
npm install -g grunt-cli
npm install -g bower
npm install && bower install
```

If you have encountered **ERR!** like the one below, please run the commands with sudo.

> Please try running this command again as root/Administrator

## Step 3: Run it

```Shell
grunt serve        // development
grunt serve:dist   // production
```

### Tips:

If you install the environment first time you need run the `grunt serve:dist` before run `grunt serve` to download related libraries;
if you had encountered the error like follows, you need run the command `launchctl limit maxfiles 4096 4096 && ulimit -n 4096` before you do the grunt serve, or you can append this cmd in your `~/.bashrc` or `~/.zshrc` at your choice.

>Running "watch" task
>Waiting...Warning: EMFILE, too many open files 'tmp'

The development mode is used for local dev environment, and it won't compile source files in order to let you debugging or testing your codes freely.

When it's booted up in development environment:

1. First off, compiling LESS and Jade into CSS and JavaScript AMD module respectively;
2. Then, [setting up a local web server](https://github.com/starandtina/portal/blob/master/tasks/options/connect.js) which will be in charge of hosting static files, such as JavaScript/HTML/CSS, etc...;
3. Then, open the default browser and navigate to home page of portal;
4. Then, [setting up local JSON server](https://github.com/starandtina/portal/blob/master/tasks/options/json-server.js) that mocks web backend server, and at the same time this process will be kept alive;
5. Finally, setting up the [watch](https://github.com/starandtina/portal/blob/master/tasks/options/watch.js) task, and then whatever any JavaScript or LESS source files changed, the browser will be live reloaded.

The production mode is used for production environment before deploying on staging or production environments, so you could take a glance for that quickly locally.

When it's booted up in production environment:

1. [Build](https://github.com/starandtina/portal/blob/master/tasks/build_task.js) the whole project, including compile/minify/concat/rev/uglify/... tasks;
2. Then, open the default browser and navigate to home page of portal;
3. Then, [setting up local JSON server](https://github.com/starandtina/portal/blob/master/tasks/options/json-server.js) that mocks web backend server, and at the same time this process will be kept alive;
4. Finally, [setting up a local web server](https://github.com/starandtina/portal/blob/master/tasks/options/connect.js) which will be in charge of hosting static files, such as JavaScript/HTML/CSS, etc...;

By the way, as for the JSON server task, and you could find the JSON data file from [test/mock/db.json](https://github.com/starandtina/portal/blob/master/test/mock/db.json). We wrote one [Grunt task](https://github.com/starandtina/portal/blob/master/tasks/json_server_task.js) which is focus on read data from this file and set up RESTFul API for you automatically. 

Example of db.json:


```JavaScript
{
  "addressBooks": [
    {
      "name": "3",
      "address": "121",
      "id": 3
    }
  ],
  "services": [
    {
      "id": "1",
      "serviceName": "MSSQL Server",
      "icon": "/img/service.png",
      "displayName": "Test Service",
      "description": "description for service"
    }
  ]
}
```

And then, the JSON server will set up the API interface below for your CRUD operation, so the development of front-end could be separated from web backend and we don't care about the changed of schema or whatever other things you could image.

```HTML
http://localhost:3000/addressBooks
http://localhost:3000/services
```

# Define UI Layout -simple

Let's say that you wanna create one `About` page in order to give one brief introduction for our product.

## Step 1: Create Layout Descriptor

We use the descriptor to define the layout.

```JavaScript
define({
  'type': 'components/layout/simple',
  'children': {
    'body': {
      type: 'components/about'
    }
  }
});
```

As for how Descriptor works, please refer to [here](). The type is the path of your component relative to directory of `src/app`.

Here we use the layout of `components/layout/simple`, and it only contains one body named `components/about` that is what we need to create.

## Step 2: Create Component

First off, we need to create the Angular controller named `about.controller.js` which is located at directory of  `src/app/components/about`.

```JavaScript
define([
  'angular',
  'components/about/about.html'
], function (angular) {
  'use strict';

  return ['$scope',
    function ($scope) {
      $scope.name = 'Hello, This is DBaaS!';
    }
  ];
});
```

Then, we will create one template named `about.html.jade` or `about.html`(we support JADE and HTML)which is located at directory of `src/app/components/about`.


```Jade
.about-container
  .page-header 
    h1 About US 
      small DBaaS
```
or

```HTML
<div class='about-container'>
  <div class='page-header'>
    <h1>About US <small>DBaaS</small></h1>
  </div>
</div>
```

Here we support pure HTML and Jade to write your template.
However, we strongly recommend to use Jade, as it's a clean, whitespace sensitive syntax for writing HTML.

## Step 3: Run it

```Shell
grunt serve
```

After local dev environments starts, then navigate to [About](http://localhost:5601/descriptors/about).

# Define Complex Component
From simple component Step 1 to 3 got an basic static page by component, but as for the product mode , we always need to get the data from backend , and to generate the dynamic page.

In the agile development , the backend and front developer always need coding at same time, so if there is an mock server to emulate the data, it will be more efficiently.  

Currently we use the `/test/mock/db.json` as the mock data, and the `/tasks/data_server_task.js` used to define the web server.

## Step 1 : Create Layout Descriptor ( as simple step1)
## Step 2 : Create Component
The controller and html are changed to :
`src/app/components/about/about.controller.js`

```JavaScript
define([
  'angular',
  'components/about/about.html'
], function (angular) {
  'use strict';

  return ['$scope','AddressModel'
    function ($scope,AddressModel) {
      $scope.addressList = AddressModel.query();
    }
  ];
});
```
`src/app/components/about/about.html`

```HTML
<div class='about-container'>
  <div class='page-header'>
    <ul>
  <li ng-repeat="address in addressList"><span ng-binding="address.name"></li>
    </ul>
  </div>
</div>
```
 
## Step 3: Define The Model
Define the Model in the factory `app/factories/addressModle.js`:

```JavaScript
define([
  'angular'
], function (angular) {
  'use strict';
  angular
    .module('vm.vchs.dbaas.factories')
    .factory('AddressModel', function ($resource) {
      return $resource('/api/addresses/:id');
    });
});
```

## Step 4: Make the Mock Data
Pre insert mock data into `/test/mock/db.json`.

```JavaScript
{
  "addresses":[
    {"id":1,"name":"PALO ALTO, CA ,USA "},
    {"id":2,"name":"SHANG HAI, CHINA"},
    {"id":3,"name":"TOKYO, JAPAN"}
  ]
}
```

## Step 5: Run it

```Shell
grunt serve
Write Unit Test
```

Here is the unit test for `AboutController` located in `src/app/about/about.controller.js` using `Jasmine` and `Karma`.

``` JavaScript
define([
  'base/src/app/components/about/about.controller.js',
  'angular-mocks'
], function (Ctrl) {
  'use strict';
  var scope;
  // Initialize a mock scope
  beforeEach(inject(function ($rootScope, $injector) {
    scope = $rootScope.$new();
    $injector.invoke(Ctrl, scope, {
      '$scope': scope
    });
  }));
  describe('Test About Controller', function () {
    it('check name', function () {
      expect(scope.name).toEqual('Hello, This is DBaaS!');
    });
  });
});
```

# Build & Release & Deploy

After you finish your coding, next step is to build, release and deploy it on production environments.
I'd like to build the whole project and optimize it through [Build](https://github.com/starandtina/portal/blob/master/tasks/build_task.js) task, and the build files will be put into dist directory by default.  It include:

* Lint JavaScript/CSS files;
* Clean dist directory;
* Convert Jade template into AMD module;
* Concat CSS files;
...
* Combines related scripts together into build layers and minifies them via [UglifyJS](https://github.com/mishoo/UglifyJS) (the default) or [Closure Compiler](http://code.google.com/closure/compiler/) (an option when using Java).
* Optimising CSS by inlining CSS files referenced by @import and removing comments.

```Shell
grunt build
```

If you want to release it, and you could generate tar and zip file in the tmp directory through `distribute` task.

```Shell
grunt distribute:release
```

Finally, you could deploy it using the compressed package on your web server, such as nginx, apache or third-party CDN(such as [CloudFront](http://aws.amazon.com/cloudfront/)).

```
server {
  # /usr/share/nginx/www/portal is your root directory
  root /usr/share/nginx/www/portal
  index index.html index.htm
  
  server_name localhost;
  
  location / {
    try_files $uri $uri/ /index.html;
  }
 
  location /api {
    rewrite /api/(.*) /$1 break;
    proxy_pass "http://localhost:3000";
  }
 
  location /favicon.ico {
      log_not_found off;
  }
 
  location /docs/docco/ {
    root /home/ubuntu/portal # /home/ubuntu/portal is the directory of documentation
  }
 
  location /docs/styleguide/ {
    root /home/ubuntu/portal;
  }
}
```
 