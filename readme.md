# Factories, Services and `$http`

## Learning Objectives

* Explain the purpose of Factories in Angular.
* Use `$http` to pull information from an API.
* Use $stateParams to access query parameters and update the URL.
* Create separate views and routes for each CRUD action.

## Framing

In the last couple of classes, we've been using hard coded values in our controller to act as our "backend". We probably won't ever do that again. Instead we'll be connecting to an external API using resources and providing an interface to models using factories.

# Angular Factories

What is a Factory.

Basically, it is an object. That is really it.

It's purpose is to store data to be shared between multiple controllers.

Even if you have one controller, it is nice to continue the process of seperation of concerns and storing the data for you controller functionality in it's own object.

What makes an Angular Factory is that is is an already created object, and it can be inserted into the controller.

### Factories vs Services

We also have something in Angular called a Service. Services are objects too, just like Factories. The difference is that a Service is instantiated with the keyword `new` to create an instance of itself. 

Like you have seen with constructor functions you can create multiple versions of the same Object that has similar precoded or inherited functionality.

Which One Should I Use?

The answer is it doesn't really matter. You might take a look at this "cheat sheet" of what should be used when:

[http://demisx.github.io/angularjs/2014/09/14/angular-what-goes-where.html](http://demisx.github.io/angularjs/2014/09/14/angular-what-goes-where.html)

Great article comparing Factories, Services, & Providers:

[http://tylermcginnis.com/angularjs-factory-vs-service-vs-provider/](http://tylermcginnis.com/angularjs-factory-vs-service-vs-provider/)

## You Do: Walkthrough of Current App (20 minutes / 0:20)

> With the person next to you, take 15 minutes to walk through the following part of the lesson plan, up to the `Factories` header. Read our descriptions of the different components.

> We'll then take the next 5 minutes for questions.

Run the below commands to clone this class' starter code. You will not be using the code you created in the `ui-router` class.  

```bash
$ git clone git@github.com:estermer/grumblr-ng-factory.git
$ cd grumblr-ng-factory
```

Where we're picking up the app, it has...
* A functioning index route that uses grumbles hardcoded into the index controller
* The makings of a show route. We'll need to build this out.

#### index.html

```html
<!DOCTYPE html>
<html data-ng-app="grumblr">
  <head>
    <title>Angular</title>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.8/angular.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-router/0.2.15/angular-ui-router.min.js"></script>

    <script src="js/grumblr.module.js"></script>
    <script src="js/grumblr.controller.js"></script>
    <script src="js/grumblr.router.js"></script>
  </head>
  <body>
    <h1>Grumblr</h1>
    <main data-ui-view></main>
  </body>
</html>
```
> **data-ng-app**: Establishes the domain of our Angular application.  
>  
> **data-ui-sref:** This creates a link that, when clicked, directs the user to `#/grumbles` without reloading the page.
>
> We use this instead of the usual `href` because `sref` refers to a state and automatically grabs the URL for that state. It's like link helpers in rails.
>  
> **data-ui-view:** Whichever view is triggered by the user will be displayed in the DOM element with this attribute. 
>
> **data-...:** There is no difference between adding data infront of our directives verses not adding it. All it does is eliminates errors that are thrown by HTML5 on the directive syntax.

#### js/ng-views/index.html

```html
<h2>These are all the Grumbles</h2>

<div data-ng-repeat="grumble in vm.grumbles">
  <p>{{grumble.title}}</p>
</div>
```
> **data-ng-repeat:** Allows us to iterate through each item in the array passed in as an argument. In this case, the controller's `grumbles` property.  
>  
> **vm:** Represents the current instance of our index controller.  

#### js/grumblr.module.js

```js
(function(){
  angular.module("grumblr", ["ui.router"]);
})();
```

#### js/grumblr.router.js

```js
(function(){
  angular
    .module("grumblr")
    .config(router);

  router.$inject = ["$stateProvider"];
  
  function RouterFunction($stateProvider){
    $stateProvider
      .state("grumbleIndex", {
        url: "/grumbles",
        templateUrl: "js/ng-views/index.html",
        controller: "GrumbleIndexController",
        controllerAs: "vm"
      })
      .state("grumbleShow", {
        url: "/grumbles/:id",
        templateUrl: "js/ng-views/show.html"
      });
  }
})();
```
> **.module:** A module is a container for controllers, directives, services -- all parts of our application. A module can have sub-modules .
>  
> **ui.router:** A 3rd party module that functions as a router.  Allows our application to have multiple states.  
>  
> **$stateProvider:**  A ui-router service that allows us to define states in our application.  
>  
> **.state:** Used to define an individual state in our application. Arguments include (1) state name and (2) an object that contains information about route, template and controller used.  


#### Index Controller

```js

(function(){
  angular
    .module("grumblr")
    .controller("GrumbleIndexCtrl", GrumbleIndexCtrl);

  function GrumbleIndexControllerFunction(){
    this.grumbles = [
      {title: "These"},
      {title: "Are"},
      {title: "Hardcoded"},
      {title: "Grumbles"}
    ]
  }
})();
```
> **GrumbleIndexCtrl:** The name of this controller.  
>  
> **GrumbleIndexCtrl:** A function that contains this controller's behavior. This is a stylistic decision - we could have passed in an anonymous function to `.controller` if we wanted to.  

You'll notice that, at the moment, we have hard-coded models into the Grumbles controller. Today we'll be learning about `ngResource`, a module that allows us to make calls to that Rails API we'll set up now.

## Set Up Grumblr API (5 minutes / 0:25)

Let's start by cloning and running a Grumblr Rails API in the background. Our front-end Grumblr application will make AJAX calls to this API.

```bash
$ git clone https://github.com/ga-wdi-exercises/grumblr_rails_api.git
$ cd grumblr_rails_api
$ bundle install
$ rails db:create
$ rails db:migrate
$ rails db:seed
$ rails s
```

## Factories (10 minutes / 0:35)

First up, we'll convert the hardcoded data to read from an external API using a factory. A factory, however, is not the only way to accomplish this. Let's see what tools we have at our disposal.

### Factory

A factory is an Angular component that adds functionality to an Angular application. It does this by generating new instances of something. In this case, Grumbles.  

Factories allow us to separate concerns and extract functionality that would otherwise be defined in our controller. We do this by creating an object, attaching properties and methods to it and then returning that object.

Let's start building out a factory in Grumblr.

```bash
$ touch js/grumblr.factory.js
```

```js
(function(){
  angular
    .module("grumblr")
    .factory( "GrumbleFactory", GrumbleFactoryFunction);

  function GrumbleFactoryFunction(){
    return {
      helloWorld: function(){
        console.log( "Hello world!" );
      }
    }
  }
 })();
```
> Factories can also take dependencies. In that case, the arguments passed into a factory will look a little different. We'll see that in play when we learn about `ng-resource` later today.

Now we can call it in a controller...

```js
  GrumbleIndexCtrl.$inject = ["GrumbleFactory"];

  function GrumbleIndexControllerFunction( GrumbleFactory ){
    // When `helloWorld` is called, it runs the function we defined in our factory file.Factory
    GrumbleFactory.helloWorld();
  }
```
> This is nice because it keeps our controller clean. We leave the function declaration(s) to our factory.

### I Do: Create Grumble Factory (15 minutes / 0:50)

> Do not code along. You will have the chance to write all this code in the following exercise.

Let's make a factory that's actually useful. It's purpose: enable us to perform CRUD actions on our Rails Grumblr API.  

There is a service that we can inject into either our factory or our controller that will allow us to make AJAX calls to our Grumblr API... `$http`.

Let's include it in our application.  

### You Do: Create Grumble Factory + Update Index Controller (10 minutes / 1:00)

### Break (10 minutes / 1:10)

### You Do: Show (20 minutes / 1:30)

#### Create a Show Controller

#### Modify the Show `.state()` in `app.js`

#### Update `index.html`

#### Create a Show Controller

#### Update `show.html`

Use what you learned on your first day of Angular to create a show view for a Grumble. It should display the Grumble's title, author name, created at date, content and photo.  

### I Do: New/Create (15 minutes / 1:45)

> Do not code along. You will have the chance to write all this code in the following exercise.

#### Create `grumbleNew` Route

```js
// app.js

$stateProvider
  .state("grumbleIndex", {
    url: "/grumbles",
    templateUrl: "js/ng-views/index.html",
    controller: "GrumbleIndexController",
    controllerAs: "vm"
  })
  .state("grumbleNew", {
    url: "/grumbles/new",
    templateUrl: "js/ng-views/new.html",
    controller: "GrumbleNewController",
    controllerAs: "vm"
  })
  .state("grumbleShow", {
    url: "/grumbles/:id",
    templateUrl: "js/ng-views/show.html",
    controller: "GrumbleShowController",
    controllerAs: "vm"
  });
```

> NOTE: `grumbleNew` is placed before `grumbleShow`. This is important - why? Switching `grumbleNew` and `grumbleShow` may shed some light on this...  

#### Create `new.html`

Let's start by creating a form view for creating Grumbles.

```html
<!-- js/ng-views/new.html -->

<h2>Create Grumble</h2>

<form>
  <input placeholder="Title" data-ng-model="vm.grumble.title" />
  <input placeholder="Author name" data-ng-model="vm.grumble.authorName" />
  <input placeholder="Photo URL" data-ng-model="vm.grumble.photoUrl" />
  <textarea placeholder="Grumble content" data-ng-model="vm.grumble.content"></textarea>
  <button type="button" data-ng-click="vm.create()">Create</button>
</form>
```
> Fields are matched to grumble properties using the `data-ng-model` directive.  

#### Add New Link to `index.html`

This link will trigger the `grumbleNew` state when clicked.

```html
<a data-ui-sref="grumbleNew">New Grumble</a>

<h2>These are all the Grumbles</h2>

<div data-ng-repeat="grumble in vm.grumbles">
  <p><a data-ui-sref="grumbleShow({id: grumble.id})">{{grumble.title}}</a></p>
</div>
```


#### Create new controller

### You Do: New/Create (10 minutes / 1:55)

### Break (10 minutes / 2:05)

### You Do: Edit/Update (15 minutes / 2:20)

The steps here are pretty similar to those of the last "I Do," with a few exceptions. The biggest one is...

#### Define an `update` method in the Factory

The rest of the steps are a bit more straightforward...  

#### Create `grumbleEdit` Route

Follow the same process we did for `grumbleNew`, making sure to use the word `edit` wherever necessary.  

Not sure what URL to use? Think about what the path would look like for an edit form in a Rails app...  

#### Update `index.html`

Let's update our `ng-repeat` div so that it also displays a link with each Grumble that will direct us to an edit page.

```html
<!-- js/ng-views/index.html -->

<div data-ng-repeat="grumble in vm.grumbles">
  <p><a data-ui-sref="grumbleShow({id: grumble.id})">{{grumble.title}}</a></p>
  <a data-ui-sref="grumbleEdit({id: grumble.id})">Edit</a>
</div>
```
#### Create `edit.html`

The form on this page will look a lot like the one in `new.html`, but you'll need to make some changes to it...
* Reference the proper controller instance. You probably called it `vm`.
* Replace your inputs' `placeholder` attribute with `value` so we have some content to work with in our input fields upon page load.
* Set these value attributes to the contents of the Grumble like so...
```html
<input value="vm.grumble.title" ... >
```
* In the button's `ng-click` directive, reference a yet-to-be-defined `.update` method instead of `.create`.

#### Link to Edit Controller in `index.html`

#### Create `edit.controller.js`

### You Do: Delete (10 minutes)

>  We may not get to this in-class.  

Contrary to how we've done things for every other RESTful route, we will not be creating a separate controller for `delete`. This is because we want to be able to delete a Grumble simply by clicking a button on each grumble's show page.

#### Add Delete Button to `edit.html`

When clicked, the delete button will trigger a `destroy` method that we have yet to define in `edit.controller.js`.

```html
<form>
  <input value="vm.grumble.title" data-ng-model="vm.grumble.title" />
  <input value="vm.grumble.authorName" data-ng-model="vm.grumble.authorName" />
  <input value="vm.grumble.photoUrl" data-ng-model="vm.grumble.photoUrl" />
  <textarea value="vm.grumble.content" data-ng-model="vm.grumble.content"></textarea>
  <button data-ng-click="vm.update()">Update</button>
  <button data-ng-click="vm.destroy()">Delete</button>
</form>
```

#### Add Destroy Method to Edit controller

### Closing/Questions (10 minutes / 2:30)

* Why do we use factories and services?
* What do factories and services return?
* What are `ngResource` and `$resource`? What methods do they provide us with?
* How do we use `$resource` and a factory to create and save something to an API/database?

----------

### Grumblr Bonuses

> Only attempt the following once you have set up full CRUD functionality for Grumblr.

#### state.go

Use `state.go` so that when a user creates, edits or deletes something, they are directed to a page that is not the same form.

> We're not covering this in today's class. [Learn more in the `ui-router` documentation](https://github.com/angular-ui/ui-router/wiki/Quick-Reference).

#### Hardcore SPA

Only use a single view and controller (i.e., you should be to execute full CRUD functionality from the index). The user should only be able to see forms for creating and updating after clicking buttons for those respective actions.

> This will require making use of directives we didn't use in today's class, like `ng-show` and `ng-hide`.

### Homework (Optional)

Finish implementing full CRUD functionality for Grumblr using `ngResource`. In other words, finish going through all the code in this lesson plan.  

Links to the starter and solution code can be found in the [`grumblr_angular` repo](https://github.com/ga-wdi-exercises/grumblr_angular).

### Resources

* Angular documentation for [ngResource](https://docs.angularjs.org/api/ngResource) and [$resource](https://docs.angularjs.org/api/ngResource/service/$resource).
* [Angular: What Goes Where?](http://demisx.github.io/angularjs/2014/09/14/angular-what-goes-where.html)
* [Factory vs. Service vs. Provider](http://tylermcginnis.com/angularjs-factory-vs-service-vs-provider/)

## Screencasts
- Dec 16, 2015 (Robin)
  - [Part 1](https://youtu.be/Ni-KnX9eEDI)
  - [Part 2](https://youtu.be/Jm4lmgpQfJ8)
  - [Part 3](https://youtu.be/dP0YsPTnaTU)
  - [Part 4](https://youtu.be/oEFmmQgh4cE)
