---
layout: post
title: Things to consider before we start coding
date: 2022-01-01 20:19:34 +0800
categories: architecure
---
## Backend Configurations
This configuration is the typical file that we use in our backend service. The main goal is to identify the configurations that are crucial for the application restart. We need to store all dynamic setup in database instead of using a flat json file.

As much as possible we don't want to inform our clients saying that we need to restart the backend service in order to apply the new SMTP credentials.

## Naming Conventions
It is good to have a single naming convention for front-end upto backend parameters. As much as possible use the Ubiquitous language instead of thinking for a new parameter name inside a code block. This is to lessen the thinking of parameter names and maximizing the use of copy-paste coding.

eg.
```javascript
// api/controller/task.js
const saveOrUpdate = async (req, res) => {
    const {
        id,
        name,
        category_id,
    } = req.body
    // .. save or update record
    return res.status(200).json(record);
}
```
```dart
// lib/model/task.dart
class Task {
    final int? id;
    String name;
    String? category_id;
    
    Task({
        required this.id,
        required this.name,
        this.category_id,
    });

    factory Task.fromJson(Map<String, dynamic> json) {
        return Task(
            id: json['id'],
            name: json['name'],
            category_id: json['category_id'],
        );
    }

    Map<String, dynamic> toJson() {
        return {
            'id': id,
            'name': name,
            'category_id': category_id
        }
    }
}
```


## Ports and Adapters
Identify the parts in project where we can inject more features.<br>
for example:
<br>front-end UI, it can be the bottom-nav, side-nav, app bar, drawer etc..
<br>in backend, we have the routes and middlewares.

It is about on how we can separate the concerns and code base per feature, also to support client customization plugins.

For example:
We created ProjectA for ClientA, then ClientB want to use the ProjectA, but they have customizations in the current process.

Instead of creating a monolithic project, we can create feature by feature. Meaning we build the whole project from a modular setup.

## Support Customizations
When we say customization, it is already a plugin requirement. And in order to apply a plugin, we need to define a standard installation process.

The below is only based on my own, there might be existing standard implementation to handle customization in frameworks or defined by your current company project architecture.

- create a commons package, this package is responsible for the dependency injection interfaces that we want to expose to client customizations, may also contain POCOs for plugin requirement definition.
- create a package for the authentication, we need this separate so we dont need to require everything just to develop a single feature.
- create a blank package called client_custom, it should follow the standard installation process.

The main project will require the features, if we have client custom, we need to copy / clone the client_custom package then we do dependency overrides.

## Unit Testing
I knew this one is a common dilemma for all of us, most of the time developers will skip this process in order to meet the timeline. As a result we always end up repeating the manual testing.

If we think of it like a hussle for us prior to meet our deadlines, we need to keep in mind that there is no need to have it already coded in full, we only need to code at least a prototype then later on we complete the unit testing gradually.

Because the real problem here is when the client already encounter the issues, we already reach the deadline and there is no time for us to design how we inject the unit test case because everything the client raise will be urgent.

As always, <em>Prevention is better than cure.</em>