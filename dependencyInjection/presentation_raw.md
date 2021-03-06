###### DEPENDENCY INJECTION ######
Case: A component depends on another component for a specific functionality

=> We have tightly coupled classes that are hard to maintain.

We can extract the dependency in an Interface and decouple things a bit, BUT
we still need to create the actual object that implements the Interface class or
whatever. => We still have coupling.

_____Inversion of Control(IoC)_____ - describes a design in which custom-written components
in a program receive a flow of control from a generic reusable library.

Usually in text-based programs we have a main function that calls into a menu library
to display a list of available commands. And the selection of a command is our
control-flow mechanism.
With UI-based programs we have a framework that is familiar with the common behavioral
and graphical elements and our custom code "fills in the blanks" for that framework.

Inversion of controll carries the strong connotaion that the resusable code and the
problem-specific code are developed independently even though they operate together.

Some design patterns that follow the inversion control principles:

* software frameworks
* callbacks
* schedulers
* events
* loops
* dependency injection      <- what we gonna talk about

______Dependency Injection_____ - design pattern in which one or more dependencies are
passed by reference into a dependent object and are made part of the client's state.
The pattern separates the creation of a client's dependencies from its own behavior.


______Constructor Injection____
Best suited for concrete dependencies that are needed before a component is
initialized.
<!-- http://programmers.stackexchange.com/questions/177649/what-is-constructor-injection -->

______Setter Injection____
Used when component has optional dependencies
<!-- http://symfony.com/doc/current/components/dependency_injection/types.html -->

______Property Injection____
When setting public properies of a class to a dependent class.

______Interface Injection____

This is simply the client publishing a role interface to the setter methods of the client's dependencies. It can be used to establish how the injector should talk to the client when injecting dependencies.
<!--
// Service setter interface.
public interface ServiceSetter {
    public void setService(Service service);
}

// Client class
public class Client implements ServiceSetter {
    // Internal reference to the service used by this client.
    private Service service;

    // Set the service that this client is to use.
    @Override
    public void setService(Service service) {
        this.service = service;
    }
} -->

Word for an Assembly file used with an Injector


Overall the cool quote I found to describe the therm:
~~~~~"Dependency Injection" is a 25-dollar term for a 5-cent concept.~~~~~


_____Advantages_______

* Adding new functionality to legacy code and better refactoring
* Better unit testing in isolation using stubs/mocks
* Reusability
* Can be used to externalise systems' configurations in config files(build/dev/live)
* Reduction of repetitive code
* Concurrent development of code ( two or more developers can work simultaniously on
a functionality without interfeering with each other's work)
* Loose coupling

______Disadvantages_____

* Hard debugging
* Diminishes encapsulation by requiring knowledge of the component's dependencies


############## How's it done in Angular ############
IN ANGULAR (From Documentation 1.x)
There are only three ways a component (object or function) can get a hold of its dependencies:

* The component can create the dependency, typically using the new operator.
* The component can look up the dependency, by referring to a global variable.
* The component can have the dependency passed to it where it is needed.

The first two options of creating or looking up dependencies are not optimal because they hard code the dependency to the component. This makes it difficult, if not impossible, to modify the dependencies. This is especially problematic in tests, where it is often desirable to provide mock dependencies for test isolation.

The third option is the most viable, since it removes the responsibility of locating the dependency from the component. The dependency is simply handed to the component.


The Three (Annotation-wise) ways to do DI in angular

* Inline Array Annotation

<!--
someModule.controller('MyController', ['$scope', 'greeter', function($scope, greeter) {

}]) -->

* $inject Property Annotation

<!--
var MyController = function($scope, greeter) {

}

MyController.$inject = ['$scope', 'greeter'];
someModule.controller('MyController', MyController);

-->

* Implicit Annotation

<!--
someModule.controller('MyController', function($scope, greeter) {

});
 -->


<!-- TODO: add pic from documentation about the injector
      https://docs.angularjs.org/guide/di
 -->

Responsibility of dependency creation, each Angular application has an injector. The injector is a service locator that is responsible for construction and lookup of dependencies.
Here is an example of using the injector service:

<!-- // Provide the wiring information in a module
var myModule = angular.module('myModule', []); -->
Teach the injector how to build a greeter service. Notice that greeter is dependent on the $window service. The greeter service is an object that contains a greet method.
<!--
myModule.factory('greeter', function($window) {
  return {
    greet: function(text) {
      $window.alert(text);
    }
  };
}); -->
Create a new injector that can provide components defined in our myModule module and request our greeter service from the injector. (This is usually done automatically by angular bootstrap).

<!-- var injector = angular.injector(['myModule', 'ng']);
var greeter = injector.get('greeter'); -->



<!-- The solution to LoD of Angular people... not very elegant IMHO -->
Asking for dependencies solves the issue of hard coding, but it also means that the injector needs to be passed throughout the application. Passing the injector breaks the Law of Demeter. To remedy this, we use a declarative notation in our HTML templates, to hand the responsibility of creating components over to the injector, as in this example:

<div ng-controller="MyController">
  <button ng-click="sayHello()">Hello</button>
</div>
function MyController($scope, greeter) {
  $scope.sayHello = function() {
    greeter.greet('Hello World');
  };
}

<!-- And on after we have read all that in the documentation,
      we get a tiny note on the bottom saying:
      "Note: Angular uses constructor injectin"
      GJ!
 -->


########IN ANGULAR 2.0#########


Will be pretty much the same as the role of the $injector service
<!-- https://docs.angularjs.org/api/auto/service/$injector --> in Angular 1.x,
but won't be limited to instantiating Angular providers.


Sample injection

Before:

<!--
  var injector = new Injector();
  new AppView();
  new Filters();
 -->

After
<!--
  var injector = new Injector();
  injector.get(AppView);
  injector.get(Filters);
 -->

Injector.get( tokenParameter );

Token can be a string/Object etc. ( thnx to ES6 )
Returns a singleton.


What the injector is for - breaking up manually maintained dependencies
and modularizing code.

<!--
import {Inject} from 'di/annotations';
    import {Electricity} from '../electricity';

    @Inject(Electricity)
    export class Heater {
      constructor(electricity) {
        this.electricity = electricity;
      }

      on() {
        console.log('Turning on the coffee heater...');
      }

      off() {
        console.log('Turning off the coffee heater...');
      }
    }
 -->

We can also do mocks

<!--
import {Provide} from 'di/annotations';
import {CoffeeMaker} from './coffee_maker/coffee_maker';

@Provide(CoffeeMaker)
class MockCoffeeMaker {
  brew() { /* brew brew brew coffee*/}
}

export MockCoffeeMaker;

// and in the test

import {MockCoffeeMaker} from './mocks/coffee_maker';

function main() {
  var injector new Injector([MockCoffeeMaker]);
  var coffeeMaker = injector.get(CoffeeMaker);

  coffeeMaker.brew();
}
  -->

Design doc:
https://docs.google.com/document/d/1fTR4TcTGbmExa5w2SRNAkM1fsB9kYeOvfuiI99FgR24/edit#heading=h.2e8op9ntdrm0


##########################################################################
#####################Actor Model

Sample case:
The Dining philosophers problem
<!-- http://en.wikipedia.org/wiki/Dining_philosophers_problem -->


Dummy decision:
All the philosophers to reach for the left fork whenever possible...

All the threads will race for the forks, and here we have a potential lock.

More adequate decision:
Use a "Waiter" object (Mutex)
<!-- Add link to Mutex description -->

that will be asked by the philosophers for atleast 2 free forks.


Still... there's still potential time that a deadlock might occur:

First philosopher asks for a fork. He gets a yes.
Second philosopher asks for a fork. He gets a yes.

By the time the second has gotten a for, the first has already taken it.

______Actor Model_______
An actor is a computational entity that in response to message it receives,
can concurrently:

* Send a finite number of messages to other actors
* Create a finite number of new actors
* Designate the behavior to be used for the next message it receives

There's no assumed sequence to the above actions and they could be carried
out in parallel.

Recepients of messages are identified by address. An actor can communicate
only with addresses it has. It can obtain those from a message it receives,
or if the address is for an actor it has itself created.



#### Back to the problem

Applying this solution we can guarantee to ourselves concurrency for our
waiter and philosophers.


Afcors the Actor model can be applied for far more complex situations, but
the example is sufficient.


____ Potential pitfal - self schizophrenia dew to delegation
<!-- http://en.wikipedia.org/wiki/Schizophrenia_%28object-oriented_programming%29 -->

Base class defines method foo

Child class extends Base in some matter and defines bar

bar method makes a call to foo method when invoked, but
foo is executed in the context of Base.

____Advantages

