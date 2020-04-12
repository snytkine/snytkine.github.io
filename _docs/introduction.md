---
title: Introduction
tags: 
 - bind
 - container
description: Introduction and concepts
---


# Getting Started with Dependency Injection

## Concepts

### Inversion of Control
A Dependency Injection is a specific type of Inversion of Control programming principal
In a traditional procedural programming  it is a responsibility of a programmer
to describe the flow of control that program takes in order to execute the block of code

An Inversion of Control principle inverts this flow such that it is responsibility of the framework
to generate the control flow.

### Dependency Injection
A Dependency Injection is a specific type of Inversion of Control.
In Dependency Injection there is this "Container" object that 'knows' how to create instances of classes used by the program
These instances of classes are ofter referred to ad "Components"

So we will use the term "Container" when we talk about the object responsible for constructing instances of classes
We will use the term "Component" when talking about instance objects.

Below is a quick example comparing traditional control flow vs Dependency Injection

Procedural Code.
In the example a programmer need an instance of WeatherService class
and WeatherService depends on instance of ApiClient class and Logger class
and ApiClient depends on instances of Settings and Logger
the Logger also depends on Settings.

Programmer must make sure to create all required instances "dependencies"
before the instance of WeatherService can be created.

There is a problem with this approach. Fist it's reusability issue.
What if the instance of Weather service in needed in another controller on in middleware
Surely we don't want to repeat the same code.

Second issue is that we dont really need to create a new instance of these classes every time.
At the same time some components cannot be shared between requests, meaning that the same instance should be 
reused only during the same request. For example the same instance of Logger can be re-used
in all middlewares but only during the same request.
This is known as "Component Lifetime Scope" or sometimes just "Lifetime Scope"

```typescript
import {WeatherService} from './controllers';
import {ApiClient} from './services';
import {Logger} from './logger';
import {Settings} from './configuration';

export function getWeather(req: http.IncomingMessage, res: http.ServerResponse){
 const logger = new Logger(Settings);
 const client = new ApiClient(Settings, logger);
 const service = new WeatherService(client, logger);
 const result = service.getWeather(req.query.zip_code);
 
 res.end(result);
}
```


_Same controller with Dependency Injection container_

```typescript
import {WeatherService} from './controllers';
import {container} from './container';

export function getWeather(req: http.IncomingMessage, res: http.ServerResponse){

 const service: WeatherService = container.getComponent('weather-service');
 const result = service.getWeather(req.query.zip_code);
 
 res.end(result);
}
```
In above example we just "ask" container to give us an instance
of WeatherService class. The container 'knows' what parameters the
WeatherService expects to be passed to constructor, so it will 
first create the instances of ApiClient and Logger and then
will create instance of WeatherService before returning an instance.

Another important part of this flow is that container 'knows' that by default
the created instances should be created only once and if the instance has been previously created it will 
return the already created instance, therefor providing a **Singleton** pattern.

Not all components should be **Singletons**. 
Some components really should return new instance every time component is requested
while other components must return same instance during the same request but should not share instances between different requests

It is a responsibility of the Container to know about the lifetime scope of each component.

Example of WeatherService class

```typescript
import {ApiClient} from './services';
import {Logger} from './logger';

export class WeatherService {

    private client_: ApiClient;
    private logger_: Logger;

    constructor(client: ApiClient, logger: Logger) {
        this.client_ = client;
        this.logger_ = logger;
    }

    getWeather(zipcode: string){
        this.logger_.info(`Entered getWeather for zipcode=${zipcode}`)
        const res = this.client_.request(`http://weather-api.net/getdata?zip=${zipcode}`)    
        this.logger_.info(`Returning data from client ${res}`);
        
        return res;
    }
}
```
