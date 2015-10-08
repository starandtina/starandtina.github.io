---
layout: post
title: "Backbone MVC Style Guide"
description: "Backbone MVC Style Guide"
category: ['Backbone', 'Styleguide']
tags: ['Front-End', 'JavaScript', 'Backbone', 'Style Guide']
---
{% include JB/setup %}

最近项目中会用到[Backbone](http://backbonejs.org/)作为客户端的**MVC**库来构建[SPA](https://en.wikipedia.org/wiki/Single-page_application)，本文主要介绍开发过程中总结的一些规则。
{: .countheads }

* ToC
{:toc}

### Real MVC

![Real MVC](/assets/images/real_mvc.png)
{: class='image-wrapper'}

简单来说，经典的**MVC**架构是这样工作的。

首先我们会有一个**Model**，它往往是整个MVC架构的中心。如果这个**Model**有更新了，它会通知它的观察者们某个改变发生了。然后是**View**，它就是我们常常能看到的UI，并且**View**一般会监听着**Model**。当**View**收到**Model**发来的更新通知时，**View**会更新它的外观来反映**Model**的更新。作为用户的我们通常是与**View**交互（如点击，输入事件等等），但是呢，**View**并不知道如何处理用户的交互操作。所以呢，**View**会告诉**Controller**用户刚刚做了什么并且假设**Controller**知道如果处理用户的操作。**Controller**这时候会更新**Model**，之后不会一次一次又一次一次的重复重复再重复。

### Views

* Keep views skinny, most of their logic concerns event management.
* Any communication between children and parent views should use events (do not pass an instance of the parent around).
* Should only know about children views.
* Should be scoped to a single element, and not access or alter other parts of the DOM. 
* Should have a unique class name, and all CSS relevant to this controller should be namespaced by this class name.
* Should work even if their element is not appended to the DOM tree. This is useful for testing and demonstrating the controller is properly scoped.
* Should be testable by firing events programatically on their sub elements.
* Should be able to be re-rendered at any time without adverse effects.
* Should not use the DOM to store state — keep any view specific state stored as instance properties in the controller.
* Views are Data-Less, it means data belongs to models not views.

### Templates

* Should be agnostic about template engine
* Should be logic less except for flow control and helper functions.
* Should be passed all the data they need to render directly. It should all be available in the current scope.

### Models

* Should purely be concerned with data, manipulating data and decorating data.
* Should never access controllers or views.
* Should never access or return DOM nodes.
* Should only access or require other models at runtime.
* Should abstract away XHR connections from the rest of the application.
* Should contain all data validations.
* Events are better than callbacks

### DOM

* DOM events only affect models, when a DOM event occurs, such as a click on a button, don’t let it change the view itself. Change the model.
* DOM Changes only when the model changes, the easiest approach is to render again after each change, but you'd better render again your children views
* What is bound must be unbound, when a view is removed from the DOM using the `remove` method, it must unbind from all the events it is bound to.
* Should always scope yourself to your own DOM element instead, using `this.$` instead of jQuery directly

### Routers

* Should be as logic-less as possible.
* Should not know about or access the DOM.

### Global state

* Should be kept to a minimum as it arguably violates MVC. You can't get around it for some things though, like the current user, current view, CSRF token etc.
* Should never be stored in global variables.
* Should all be stored in one place (such as a singleton instance of a model).

### Modules

* Should always use a module system, whether it's CommonJS, AMD, ES6 modules, or something equivalent.
* Never set or access global variables.
