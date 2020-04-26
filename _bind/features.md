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
import {Logger} from './logger.ts';

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
import {Logger} from './logger.ts';

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
import {Logger} from './logger.ts';

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
import {Logger} from './logger.ts';

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
import {Logger} from './logger.ts';

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

### Support for Lifecycle callbacks @PostConstruct and @PreDestroy

### Support for Component Factory concept. 
>> Component can have methods that return other components.



