# What is spirit.io ?

spirit.io is an extensible Node.JS ORM framework written with Typescript.  
Its goal is to simplify complex application development by writing simple decorated model classes.  
This classes will be translated by decorators and schemas compiler analysis in order to produce models schemas that would be used by spirit.io connectors...  
At the end, the framework will produce CRUD operations usable from server side using all the power of Typescript language, but also from front-end applications a REST API with Express routes will be automatically generated.  

# Prerequisites

To use spirit.io, you need :  
  * Node.JS 6.X
  * At least one spirit.io connector. eg: spirit.io-mongodb-connector
  * The connectors associated runtimes. eg: MongoDB server

# Installing

Installing spirit.io is simple as :  

```sh
# to install the framework
npm install --save spirit.io
# to install connectors
npm install --save spirit.io-mongodb-connector
npm install --save spirit.io-redis-connector
...
```

# Getting Started

First of all, spirit.io is a framework, so you need to create your own project to be able to use it.  
Then, it's important to understand that spirit.io uses [f-promise](https://github.com/Sage/f-promise) API, so please take a look to its documentation to learn all the benefits you will encounter.  

When using `spirit.io` with `Typescript`, the Typescript compilation is done by common typescript compiler `tsc`.  
But the first entry point is that you need to create a spirit.io Server instance.  

There are many ways to declare and start your spirit.io application :

* The simpler way using the `Server` class provided by the framework :

`simple.ts`:  

```ts
import { run } from 'f-promise';
import { Server } from 'spirit.io/lib/application';
import { Request, Response, NextFunction } from 'express';

// Declare your configuration
const config = {
    port: 3000
};

//
// f-promise encapsulation with `run` function is really important here
// as spirit.io framework use `wait` function usually.
//
run(() => {
    let server: Server = new Server(config);
    server.init().start();
    // ====================================
    // Do what you want in your application
    server.app.use('/test', function (req: Request, res: Response, next: NextFunction) {
        res.end('It works !');
        next();
    });
    // ====================================
}).catch(err => {
    console.error(err);
});
```

* The second and proper way using the events emitted by the `Server` instance :

`advanced.ts`:  

```ts
import { run } from 'f-promise';
import { Server } from 'spirit.io/lib/application';
import { Request, Response, NextFunction } from 'express';

const config = {
    port: 3000
};

let server: Server = new Server(config);

// declare `initialized` event listener.
server.on('initialized', function () {
    run(() => {
        console.log("========== Server initialized ============\n");
        // ====================================
        // Do what you want in your application after it has been initialized, 
        // but before it would be started
        server.app.use('/test', function (req: Request, res: Response, next: NextFunction) {
            res.end('It works !');
            next();
        });
        // ====================================

        // Then start the server
        server.start();
    }).catch(err => {
        console.error(err);
    });
});

// declare `started` event listener.
server.on('started', function () {
    run(() => {
        console.log("========== Server started ============\n");
        // ====================================
        // Do what you want in your application after it has been started
        // ...
        // ...
    }).catch(err => {
        console.error(err);
    });
});

// Here is the first entry point...
//
// f-promise encapsulation with `run` function is really important here
// as spirit.io framework use `wait` function usually.
//
run(() => {
    // Call `init` function to initialize your server
    server.init();
}).catch(err => {
    console.error(err);
});
```

* The third and probably the more common way that would be used is to create your own CustomServer class overriding the framework's `Server` class :
Entry point can be implemented as the same way described in simple and advanced examples...

`custom.ts`:  

```ts
import { Server } from 'spirit.io/lib/application';

export class CustomServer extends Server {
    constructor(config?: any) {
        if (!config) config = require('./config').config;
        // Some specific stuff can be done in your constructor...
        super(config);
    }

    init() {
        // Here could be the place to do whatever you want before standard initialization...
        return super.init();
    }

    start(port?: number) {
        // Maybe do something before starting express application ?
        super.start(port || this.config.port);
    }
}
```

