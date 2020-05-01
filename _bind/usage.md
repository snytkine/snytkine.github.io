---
title: Bind-DI Container Usage
nav_title: Usage
order: 4
description: How to use Bind-DI Container
---

# How to use Bind-DI Dependency Injection container
Bind-DI is Decorator based Dependency Injection framework.

This means that all components, component lifetime scopes and dependencies are expressed
via applying simple **Decorators** on Classes, Properties and property Setters.

This also means that there is no need for any type of external configuration file.

An important feature of **Bind-DI** Framework is auto-discovery and auto-loading of components from the file system.

There are 3 phases of container lifetime.

| Phase | What takes place |  
| ---- | ---- |
| Loading | Recursively scan directories, discover exported classes that are decorated with @Component or with other related decorators. Add components to Container<br>The discovery and loading of classes from file system is synchronous operation. |
| Initialization | 1. Validate that all loaded components that have dependencies (via @Inject or via implicit constructor dependencies) can find all dependencies in container. <br>2. Validate that there are no dependency loops.<br>3. Instantiate all components that have _@PostConstruct_ decorator and call these post-construct methods and wait for Promise(s) returned from post-construct to resolve. Only after every post-construct method has resolved the initialize() method itself is resolved. <br>At this point container is ready for use.|
| Runtime | In this phase **Container** is responsible for returning components to the client. The client uses .getComponent method, passing **Identity** of requested component and **Container** returns instance of requested component, complete with all necessary component dependencies.|



