---
title: Features
nav_title: Features
order: 2
tags: 
 - bind
 - container
description: Framework Features
---


# Bind Framework Features
>> Bind in an Annotation-based Dependency Injection Container (DI Container)
with components auto-loading from file system.

In TypeScript terminology the Annotation is called a Decorator. 

### Auto-loading components from file system. 
>> Loader will scan directory and recursively all subdirectories and will discover 
all classes that have @Component decorator. These classes will be treated at Components
and will be auto-loaded to container.

### 3 ways to inject dependencies.
1. Property Injection
2. Constructor Injection
3. Setter Injection

A Component can have any combination of the injection types.
For example a component can have **Constructor Injection** and **Property Injection** and **Setter Injection**.

The **Constructor Injection** is the most convenient for unit testing. In Unit Testing a dependency must often be replaced by a mock implementation of the dependency object.
Passing all dependencies via a constructor makes it easy to create testable object while substituting the dependency objects with their mocks.


### Named-components and Unnamed user-defined class components
##### Example of Unnamed user-defined component. 

Just add @Component decorator to a class.

```typescript
import { Component } from 'bind-di';

@Component
export class Logger {
  
  info(message: string) {
    console.log(`INFO ${message}`);
  }

  error(message: string) {
    console.error(`ERROR ${message}`);
  }
}
```
This component can be injected as a dependency into any other component
using @Inject decorator.

Below is example of **property injection**.
Notice that injected component logger declares specific type "Logger"

That's how the container knows that it needs to inject an instance of Logger component before returning the instance of WidgetStore.

```typescript
import { Component, Logger } from 'bind-di';
import { Logger } from './logger';

@Component
export class WidgetStore {
 
 @Inject
 private logger: Logger;
 
 updateStatus(widget: string){
 this.logger.info(`Entered updateStatus with ${widget}`);
 // ...rest of the method
 }

}

```

##### Example of Named component. Same as before only now component has unique name.

_name_ can be a string or Symbol.


```typescript
import { Component } from 'bind-di';

@Component('app-logger')
export class Logger {
  
  info(message: string) {
    console.log(`INFO ${message}`);
  }

  error(message: string) {
    console.error(`ERROR ${message}`);
  }
}
```

Now in order to inject the instance of Logger we need to use named @Inject


```typescript
import { Component, Logger } from 'bind-di';
import { Logger } from './logger';

@Component
export class WidgetStore {
 
 @Inject('app-logger')
 private logger: Logger;
 
 updateStatus(widget: string){
   this.logger.info(`Entered updateStatus with ${widget}`);
   // ...rest of the method
 }

}

```

_When using named inject declaring the type is optional, but highly recommended just for the benefit of the code completion.
Also the container may enforce component types even for named components._



### Dependency injection with @Inject decorator and implicit constructor-based injection.

**Constructor Injection**
Using same example of Logger component

First Explicit Constructor Injection. 

Notice we using @Inject decorator on a constructor parameter.

It can be named or unnamed injection.

```typescript
import { Component, Logger } from 'bind-di';
import { Logger } from './logger';

@Component
export class WidgetStore {
 
 constructor(@Inject('app-logger') private logger: Logger){}
 
 updateStatus(widget: string){
   this.logger.info(`Entered updateStatus with ${widget}`);
   // ...rest of the method
 }

}

```

**Implicit Constructor Injection.**
Notice we are not using @Inject decorator. Container will still inject dependency - an instance of Logger class will be passed as argument to Widget constructor.

For Implicit constructor injection the type of constructor argument must be declared.

_Implicit dependency injection only works with constructor injection and only Unnamed Components can be injected this way._
```typescript
import { Component, Logger } from 'bind-di';
import { Logger } from './logger';

@Component
export class WidgetStore {
 
 constructor(private logger: Logger){}
 
 updateStatus(widget: string){
   this.logger.info(`Entered updateStatus with ${widget}`);
   // ...rest of the method
 }

}

```

### Setter Injection
Setter Injection is similar to Property Injection except that
a setter function is called, passing the dependency to the function.

The difference here is that a setter function can have extra logic performed during setting of dependency. 

Example:


```typescript
import { Component, Logger } from 'bind-di';
import { Logger } from './logger';

@Component
export class WidgetStore {
    
 private _logger: Logger;

 @Inject
 set logger(logger: Logger){
   this._logger = logger;     
 }

 updateStatus(widget: string){
   this._logger.info(`Entered updateStatus with ${widget}`);
   // ...rest of the method
 }

}

```
**Setter Injection** can have Named and Unnamed injection. In case of Unnamed injection the typescript's type is required for a property passed to setter function.


### Support for Component Lifecycle scopes.
When a Component is request from the Container via the .getComponent() method
the Container's job is to return an instance of requested Component with all the dependencies either passed to constructor and/or in addition
to using constructor dependencies to also set the property dependencies using simple property assignment.

By default components have a 'Singleton' Lifetime scope. 
This means that an instance of a Component is created only once and all requests to get the same component will 
always return the same instance. 

This is a very important feature of a Dependency Injection container.

But an application may have the good reason to expect a new instance of certain components every time it requests a component via a .getComponent method.

An example may be a controller method in the Rest Api application where a controller object stores some request-specific instance properties using **this** property,
for example a controller may call a helper method that sets the 

**this.currentUser** = userAccount

Naturally it will be a disaster if the instance of such controller is shared between two or most different Http Requests.

There will be a potential for an async method to set the value this.currentUser then call another async method
and while that other async method is being executed another request may come in and end up using this.currentUser value which was set previously for someone else.

So there is a need for Component to have Lifetime scope other than Singleton.

Bind-di Container supports 4 types Lifetime Scopes (with future support for custom scopes):
* Singleton
* NewInstance
* Request
* Session

Scopes are defined using the @Scope decorator

For example we may define scope on our WidgetStore Component

```typescript
import { Component, Inject, Scope, ComponentScope } from 'bind-di';
import { Logger } from './logger';

@Component
@Scope(ComponentScope.NEWINSTANCE)
export class WidgetStore {
    
 private _logger: Logger;

 @Inject
 set logger(logger: Logger){
   this._logger = logger;     
 }

 updateStatus(widget: string){
   this._logger.info(`Entered updateStatus with ${widget}`);
   // ...rest of the method
 }

}

```
In the above example the WidgetStore now has a NEWINSTANCE scope. This means that Container will
create a new instance of the class every time WidgetStore Component is requested.

_There is a shortcut decorator @NewInstance that is equivalent to @Scope(ComponentScope.NEWINSTANCE)_

Important note: a scope of dependency cannot be smaller than the scope of component.
For example a Component that is **Singleton** cannot have dependency component (via @Inject or constructor dependency) with a scope NEWINSTANCE or REQUEST because that would not even make sense.
Remember a singleton Component is created only once - the first time it is requested, so what instance can it set for its' dependency if dependency's scope is NEWINSTANCE?

_The container enforces this dependency scope rule during initialization stage and will throw an error if a component with broader scope has a dependency with smaller scope._

### Support for Lifecycle callbacks 
##### @PostConstruct and @PreDestroy
Often a **Component** (an instance of a class) needs to load some data from file system, 
establish a database connection, connect to an active directory service
 or pre-load some initial data from external source into database.
 
These types of operations are almost always asynchronous in nature and may take some time to complete.

Bind-DI framework provides 2 types of decorators for these types of lifecycle operations - 

**@PostConstruct** and **@PreDestroy**

A method decorated with **@PostConstruct** can perform any kind of one or more asynchronous operations.
When all async operations finish successfully the method must return a Promise\<true\>,
otherwise it must either throw an Error or return Promise\<false\>

When **Container** finishes loading all components (usually from auto-loading all decorated classes from file system), it enters _Initialization_ phase.
This _Initialization_ phase is asynchronous and in that phase the **Container** instantiates all components that have **@PostConstruct**-decorated methods and calls these method.

Components that have **@PostConstruct** decorators are instantiated in order determined by container. The order depends of order of dependency resolution.
When all such components are instantiated the **Container** then calls their initialization (method with @PostConstruct decorator) methods and only after all these containers finished running 
their initialization methods the Promise returned from **Container**'s own initialize() method resolves.

Only after the **Promise** returned from **Container**'s initialize() method resolved the application that uses Bind-DI container can proceed with the rest of their logic.

**@PreDestroy** decorator is similar but a Component's method decorated with **@PreDestroy** is called during container shut-down process, when application exits. The exit is triggered by a shutdown signal in a normal shutdown.

The most common use for a **@PreDestroy** methods is to close connections to databases or to free other resources, for example an application may need to close some socket connections, close previously opened file resources, etc.

**IMPORTANT** - Only **Singleton** Components can have Lifecycle callbacks **@PostConstruct** and **PreDestroy**

### Support for Component Factory concept. 
>> Component can have methods that return other components.

A Component is usually an instance of a class decorated with **@Component** decorator but there may be exceptions.
A Component can return other components from methods that themselves decorated with **@Component**
This allows a component returned from factory methods to be of any type, for example they can be pure functions, strings, numbers, etc.

**IMPORTANT** Only **Singleton** component may be a component factory (have methods that return other Components)

The following example demonstrates a component factory MongoConn which has 3 methods that return other components.
This example also demonstrates how **@PostConstruct** and **@PreDestroy** decorators are used.

Notice that components returned from factory methods are "Named" components. They have to be named because the type of these components are not specific enough for Container to uniquely identify these components.
2 of these components share the same type "Collection" and one is of type "Db" which is a class from mongodb module.

  
Example.

```typescript
import { Collection, Db, MongoClient } from 'mongodb';
import { Component, PostConstruct, PreDestroy } from 'bind-di';
import Settings from '../Settings';
import { Logger } from './logger';

@Component
export default class MongoConn {

  private mdb: Db;

  private mongoClient: MongoClient;

  constructor(private logger: Logger, private settings: Settings){}

  @Component('mongoclient')
  public getDb() {
    return this.mdb;
  }

  @Component('usercollection')
  public getUserCollection(): Collection {
    return this.mdb.collection('users');
  }

  @Component('widgets')
  public getPermissionCollection(): Collection {
    return this.mdb.collection('widgets');
  }

  @PostConstruct
  public init(): Promise<boolean> {

    return MongoClient.connect(this.settings.MONGO_URI, { useUnifiedTopology: true })
      .then((client: MongoClient) => {

        this.mongoClient = client;
        this.logger.info('MONGO CONNECTED');
        this.mdb = client.db();

        return true;
      })
      .catch((e) => {
        this.logger.info(`Mongo Connection Error ${e}`);

        throw e;
      });
  }

  @PreDestroy
  public destructor(): Promise<boolean> {

    this.logger.info('Entered destructor on Mongodb Component Factory');
    return this.mongoClient.close().then(() => {
      this.logger.info('MongoDB Connection Closed Successfully!');
      
      return true;
    });

  }

}

```


