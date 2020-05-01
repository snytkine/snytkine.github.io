---
title: Decorators
nav_title: Decorators
order: 3
description: Decorators
---

# Bind-DI Decorators

| Decorator | Parameters | Applies to | Description |
| ---- | ---- | ---- | ----|
| @Component | Optionally takes a string or Symbol. If string of Symbol is passed it becomes the unique Identifier of component | Class or Method of Component factory | If applied to a method of a class, that class must itself be decorated with @Component and must be a Singleton component |
| @Inject | Optionally takes a string or Symbol as Identifier of dependency component | Class method<br><br>Class setter function<br><br>Constructor argument | Component with smaller Lifecycle scope cannot be a dependency of component with broader Lifecycle scope. For example a NewInstance scoped component cannot be a dependency of Singleton-scoped component. This rule is enforced during container initialization stage. |
| @PostConstruct | None | Class method | Method decorated with this decorator must return Promise\<boolean\> or throw Error with description of a problem. <br/>Only **Singleton** Component can have method with this decorator and _Only One_ method of a class can have this decorator. |
| @PreDestroy | None | Class method | Method decorated with this decorator must return Promise\<boolean\> or throw Error with description of a problem. <br/>Only **Singleton** Component can have method with this decorator and _Only One_ method of a class can have this decorator. | 
| @Scope | One of values of ComponentScope enum<br> | Component Class | Components have default scope. Default scope for all components is defined by Container's .defaultScope parameter. <br>This decorator overrides the default scope of component |
| @Singleton | None | Class | This decorator is a shorthand for @Scope(ComponentScope.SINGLETON) |
| @NewInstance | None | Class | This decorator is a shorthand for @Scope(ComponentScope.NEWINSTANCE) |
| @Environment | Variable number of strings | Component Class | A Component that has @Environment("DEV", "TEST") will only be added to Container if NODE_ENV environment variable is set to either 'DEV' or 'TEST'. This allows to have multiple components with the same unique Identifier but with different values of @Enviromnent and have different components loaded in Development, Testing, Production, etc.|
| @EnvOverride | None | Component Class | Component decorated with @EnvOverride is turned into a Proxy object. Any property of Component class can now be overwritten by the value set in Environment. 
