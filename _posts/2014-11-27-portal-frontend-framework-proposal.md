---
layout: post
title: "Portal Frontend Framework Proposal"
description: "Portal Frontend Framework Proposal"
category: ['JavaScript', 'Frontend', 'js-dev']
tags: ['JavaScript', 'Frontend', 'js-dev', 'portal']
---
{% include JB/setup %}

# Overview

This article only covers the high-level front-end architecture of [portal](https://github.com/starandtina/portal), and compiled from the talking with Darcy, Long, not cover what technologies we should use.

# Goals

* Should provide a  functional portal and service catalog
* Allow for services and ISVs to extend the portal with ease
* Allow for a bunch of re-useable and consistent components
* Allow for scalable and extensible quickly and easily

# Terminologies

* FEP: front-end of portal
* BEP: web back-end of portal
* AngularJS: front-end framework for web apps by Google, and it is a toolset for building the framework most suited to your application development. It is fully extensible and works well with other libraries. 
* RequireJS: RequireJS is a JavaScript file and module loader. It is optimized for in-browser use, but it can be used in other JavaScript environments, like Rhino and Node. Using a modular script loader like RequireJS will improve the speed and quality of your code.
* Jade: Jade is a high performance template engine heavily influenced by Haml and implemented with JavaScript for node, and it's a clean, whitespace sensitive syntax for writing html.
* Grunt: A JavaScript task runner for automation.

# Features

These are the features we need to support as for the consistent front-end solution.

## Single Page Application(SPA)

SPA means we should move logic from the server to the client in order to implement RIA. So that means that the role of web-backend will be evolved into a pure data API or web service, and front-end is a JavaScript-heavy client-side.

Overall, the whole of front-end is ajax-based, all necessary resources are dynamically loaded and added to pages as needed, and does not reload at any point in the process. 

As for concern of performance, and we will pre-build all necessary resources(Controllers, Directives, Services, Factories, Templates, etc...) into just one JavaScript file, and it won't be requested if the content of that doesn't changed.

With SPA, and we could provide a more fluid user experience akin to a desktop application because of the complete lack of true page reloads.

## Client-Side Templates

Multi-page web applications create their HTML by assembling and joining it with data on the server, and then shipping the finished pages up to the browser. Most single-page applications—also known as AJAX based apps—do this as well, to some extent. But the portal is different in that the template and data get shipped to the browser also to be assembled there. The role of the server then becomes only to serve as static resources for the templates and to properly serve the data required by those templates.

## Modularity

AngularJS's dependency management doesn't organize the actual file loading dependencies. It doesn't handle placing the script tags in the proper order so say jQuery is loaded before angular is. 

Organize codes with module, so we could create and re-use these modules without polluting global namespace, also we could struct our code into separate folders and files. Another advantage is that we could compile and build the codes for production usage, such as concat, uglify all of our modules into a single package, so it  could improve our page speed. RequireJS is the best choice for that.

## Isolated From Web-backend

We shouldn't depend on the web BEP, and should totally separate from it.

What I mean is that we don't need to care about what technologies BEP use, such as PHP,  Scala + Play or Python + Django, and it only provides the data we need, also not in charge of html rendering, business logic, user interaction, and so on.

## I18N

Once your web app gets to a certain size and popularity, localizing the strings in the interface and providing other locale-specific information becomes more useful. However, it can be cumbersome to work out a scheme that scales well for supporting multiple locales. AngularJS could provided some limited support for this, for date, number and currency filters, this is not what we want. RequireJS allows you to set up a basic module that has localized information without forcing you to provide all locale-specific information up front. It can be added over time, and only strings/values that change between locales can be defined in the locale-specific file. Also AngularJS could do that but we need to write and test it on our own.

L10N fallback rule:  each component may contain a default bundle called Base with no locale tag.  Given a user locale tag of the form ll_CC, the selection rule is as follows:

      1)  if bundle ll_CC exists and also the entry exists, use it.
      2)  when 1) fails, if bundle ll exists and also the entry exists, use it.
      3)  when 2) fails, if the base exists and also the entry exists, use it.
      4)  when 3) fails, display key

## UI abstracted and driven by JSON

UI hierarchy is abstracted, described and driven by JSON object which can be fetched from a JSON file, web service or even constructed within a component. The view will be driven by that object, even the type keyword can be changed on the fly, and view will be changed with that.
The working flow:

1. [Bootstrapping](https://docs.angularjs.org/guide/bootstrap#manual-initialization) the whole app manually;
2. Monitoring [$location.path()](https://docs.angularjs.org/guide/$location#hashbang-and-html5-modes);
3. If it changes, and then [require](http://requirejs.org/docs/api.html) the related JSON descriptor module depending on the value of path;
4. If path is empty, and load the default route defined in the [config.js](https://github.com/starandtina/portal/blob/master/src/config.js) that is **descriptors/default**;
5. If JSON descriptor has been required, and then we will pass that to [parser](https://github.com/starandtina/portal/blob/master/src/app/directives/vm.parser.js);
6. `Parser` will create the component for you depend on the content of JSON descriptor;
7. Finally, [Component](https://github.com/starandtina/portal/blob/master/src/app/directives/vm.component.js) will require its related View/Controller/I18N and other resource files,  then invoke the `Controller` within current scope and then call the init method if the `Controller` had defined that.

As for JSON Descriptor, there are three bi-directional binding between a local scope property and the parent scope of the same property of name.

* type: the path of component
* children: including the child components using Array
* param: just the JSON Descriptor, and all values will be loaded into local scope if it doesn't exist in the local scope

Notes: As we use the async loading with RequireJS, so we could use the **ng-app** directive to specify your Angular app.

### Use Case

#### Define the Layout

The most important usage for descriptor is that we could use it to describe the layout of page.
Take Home page as example, and it looks like the one below:

```
define({
  'type': 'components/layout/default',
  'children': {
    /*
    'header': {
      'type': 'components/layout/header'
    },*/
    'body': [{
      'type': 'components/sideBar',
      'currentNavId': 1
    }, {
      'type': 'components/home'
    }]
    /*,
    'footer': {
      'type': 'components/layout/footer'
    }*/
  }
});
```

We will use the components/layout/default as the component defined by keyword of type and we define the layout of page in the children(also you could use other names, but children is the default one we provided in the vm-component directive).
Here is the Jade template of  components/layout/default. So you could see that if you didn't define your header component and we will use the default one of components/layout/header, it's the same for footer component.

```
.tmpst-header(role='menubar')
  vm-parser(descriptor='children.header' ng-if='children.header')
  vm-component(type='"components/layout/header"' ng-if='!children.header')
.tmpst-body(role='article')
  .container
    .tmpst-full-canvas
      .row
        .col-md-3
          vm-parser(descriptor='children.body[0]')
        .col-md-9
          vm-parser(descriptor='children.body[1]')
.tmpst-footer(role='menubar')
  vm-parser(descriptor='children.footer' ng-if='children.header')
  vm-component(type='"components/layout/footer"' ng-if='!children.header')
```

## Low learning curve

Angular has a very high wow factor at the beginning. It can do some amazing things - like two-way bindings - without having to learn much. And it looks quite easy at first sight. But after you have learnt the very basics it is quite a steep learning curve from there. It is a complex framework with lots of peculiarities. Reading the documentation is not easy as there is a lot of Angular specific jargon and a serious lack of examples.
However, after you learn the framework well what really matters is how productive you are with. You know: conventions, magic, doing as much as possible quickly.

## Performance

The performance is related with two aspects.
One is as for the codes we need to write for business logic and fundamental, for example, scope chain and prototype chain can affect your overall script performance, also it includes the framework of AngularJS we have chosen. I don't think it's the main impact factor for our case, as we have best practices and proved best solution for that.
Once the code is done and tested, and we shouldn't push out our raw source files in use for production environment, and we should build and deploy high-performance JavaScript web apps, strictly following the best practices for making web pages faster provided by The Exceptional Performance team of Yahoo. We will use Grunt performing these repetitive tasks like minification, compilation, unit testing, linting, etc, so that the easier our job becomes.

## Data Transport

This section will describe how to communicate between front-end and back-end of portal. Here is the detailed info.
Also we have defined [the standard of data transmission](http://starandtina.github.io/2014/11/27/json-data-specification/)(Sorry, it's Chinese version).

![portal_of_front_end_architecture](https://cloud.githubusercontent.com/assets/613990/4113099/4b5bf2d2-3244-11e4-9df6-01aad065bbec.png)
{: class='image-wrapper'}

We build up the entire DOM in JavaScript, loading the data via calls to RESTful JSON APIs, and handle state changes in the URL via the hash or HTML5 history API provided by framework.
On our portal, and we could serve the same HTML file for every URL, and that HTML file contains only require.js and a call to load a app.js file. That app.js file maps URLs to controllers, and controllers will know how to hander that.

I think we should figure out whats important to our users and us as the developers, not just make our decision by searching Google or seeing other guys are talking or using some cool things.

# Whats important to users

## Usability

Users could do what we had provided or what they wanna to do quickly and intuitively.
Our interfaces can easily be dynamic and real-time, enabling users to perform many interactions in a small period of time. This is particularly important for our portal interfaces, as users will manipulate many little things that are present on the one screen.

## Linkability

Users could bookmark and link to parts of the site using anchors, as for SPA, it's so hard to implement it.

## Searchability

Users could share what they're using through social networks, such as FB, Twitter, and then could find it on Google.
Since Google bots and crawlers don't handle JavaScript-rendered webpages as well(but lately it seems support that), so we have to do more work to support that,  but this doesn't matter if our web app is private.

## More rich features

Users definitely want more rich features.

## Less Bugs

This is not just users want
[Here](https://wiki.eng.vmware.com/Ueteam/projects/userresearch/VPC_FUE) is the research on vCHS first usage experience, maybe we could leverage on it.

# Whats important to us

## Developer Productivity

Should be iterate fast, extensibility, add or implement new features with ease and happily. AngularJS is the best choice for productivity.

## Testability

Our code should be robust, resilient, and it won't break.
Also as for the APIs, and we could test it separately and rigorously with best tools and libraries covering Unit/Integration/Visual Regression/Manual testing that we need another article to describe it.

## Performance

We should adopt best practices for speeding up our web sites, and make our servers respond quickly and leverage the web cache , serve minimum number of requests and reduce response time.
As for JavaScript, we also should be constantly monitoring that to make sure that we are not pushing the browsers to do too much, and don't make JavaScript run so slow at processing data or DOM.

## Style Guide

* [HTML/CSS](http://google-styleguide.googlecode.com/svn/trunk/htmlcssguide.xml)
* [Google JavaScript Style Guide](http://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml) / [Airbnb JavaScript Style Guide](http://airbnb%20javascript%20style%20guide/)
* [AMD](http://requirejs.org/docs/whyamd.html)
* [AngularJS](https://google-styleguide.googlecode.com/svn/trunk/angularjs-google-style.html)

# Testing

Get more detailed info [here](https://github.com/starandtina/portal/issues/3) .

# Build

We will use [Grunt](http://gruntjs.com/) as the default task runner and help us with repetitive works like concat, minification, compilation, testing, linting, setting up local dev, package, etc... 

# Deploy

We could use bunch of web servers to host FEP, such as nginx, apache or CDN(such as CloudFront) to host static files, such as JavaScript/HTML/CSS/Image/Web Fonts.
The deploy of FEP is totaly separated from BEP, and they are hosted in different web servers, the RESTFul API is the only channel between between them.
If we need to accelerate website, and [varnish](https://www.varnish-cache.org/) is a good choice. 

# Docs

We use [docco](http://jashkenas.github.io/docco/) and [StyleDocco](http://jacobrask.github.io/styledocco/) as documentation generator for JavaScript and CSS from the JavaScript files and the stylesheets.