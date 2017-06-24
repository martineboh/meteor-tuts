---
title: Security
description: Let's talk Security
disqusPage: 'Chapter 1: Security'
---

Security is a very important aspect of any application. When your code-base grows,
you need to be more and more careful on how you handle this. We'll show you first how we can
secure our methods and publications, then you will be shown some tips and tricks to handle 
it for an evergrowing code base.

## Securing Methods & Publicaions

So, remember the Methods ?

```js
Meteor.methods({
    'do_something': function () {
        // in it you have access to this.userId
        // which represents the logged in user
        // if your user is not logged in it will be null
        console.log(this.userId);
    }
})
```

Same thing in publications:

```js
Meteor.publish('something', function () {
    console.log(this.userId);
    
    return this.ready(); 
})
```

## Managing Roles

Based on the userId you have the ability to check if he is logged in, maybe you have multiple roles in the system,
that you may store as an array at "user document" level, and you can check for that, the thing is, you can do anything you want.

We recommend installing the infamous package, [alanning:roles](https://atmospherejs.com/alanning/roles):

```
meteor add alanning:roles
```


## Security Module

Centralize security in a module:

```js
// file: /imports/api/security.js
import { Roles } from 'meteor/alanning:roles';

export default class Security {
    static checkRole(userId, role) {
        if (!this.hasRole(userId, role)) {
            throw new Meteor.Error('not-authorized');
        }
    }

    static currentUserHasRole(role) {
        if (!Meteor.isClient) {
            throw new Meteor.Error('not-allowed', 'This method is only available on the client');
        }

        return this.hasRole(Meteor.userId(), role);
    }

    static hasRole(userId, role) {
        return Roles.userIsInRole(userId, role);
    }

    static checkLoggedIn(userId) {
        if (!userId) {
            throw new Meteor.Error('not-authorized', 'You are not authorized');
        };
    }
    
    // add other business logic checks here that you use throughout the app
    // something like: isUserAllowedToSeeDocument()
    // always keep decoupling your code if this class gets huge.
}
```

Pretty straight forward right ? The reason we do it like this, the reason we centralize security in one place,
is to remove boilerplate code inside our methods and keep separation of concerns. You can do it however you want it, there is no right or wrong,
depends on your use-case, but in my opinion I find out that it was easier to maintain, and newly onboarded developers were writing secure
code right from the start!

Simple usage in methods:

```js
import Security from '/imports/api/security.js';

Meteor.methods({
    'do_something': function () {
        // throw exception if not logged it:
        Security.checkLoggedIn(this.userId);
        
        // throw exception if not has role:
        Security.checkRole(this.userId, 'ADMIN');
        
        // conditional return
        if (Security.hasRole('ADMIN')) {
            return sensitiveData;
        } else {
            return publicData;
        }
    }
})
```

Simple usage in publications:

```js
import Security from '/imports/api/security.js';

Meteor.publish('posts', function () {
    let filters = {};
    if (!Security.hasRole(this.userId, 'ADMIN')) {
        // if the user is not an admin, we only show posts with "isPublic" true
        filters.isPublic = true;
    }
    
    return Posts.find(filters);
})
```

That's it. With this knowledge you can build very secure apps!
