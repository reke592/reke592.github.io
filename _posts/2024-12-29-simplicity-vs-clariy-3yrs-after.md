---
layout: post
title: Simplicity vs Clarity (3yrs after)
date: 2025-01-25 22:34:00 +0800
categories: architecture
---

Almost three years after writing the first version. Many things have happened that changed the way how I think. In this post, I will discuss how to start writing a scalable a NodeJS backend.

1. Understanding the Project Requirements
2. Lowering the Impact of Code Refactor

Disclaimer:
The insights shared in this post are based solely on my personal experience and may not apply universally. Always consider your specific context and requirements when applying these ideas.

### 1. Understanding the Project Requirements

Every project begins with gathering data from clients. Most of the time, clients share their current processes, sample outputs, and their specific requirements. In response, we ask how we can access the necessary data, and we typically conclude the meeting with an initial plan for starting the project.

- **Expect the unexpected** - I don't want you to overthink, but this is often the reality. Clients usually only know what they know. The initial MVP of the project often sparks their creativity, leading them to think of additional possibilities to improve their current process. To handle this, it's crucial to design the code architecture to be as flexible as possible.

- **Don't create extra features based on your own ideas** - When you're riding the momentum, control yourself. We all want to impress clients with our output, but adding extra features can lead to misalignment with their actual requirements. Don't over do, you will end up exhausted. -- If this feature already exist in other parts of the project then it's okay to include it, but the safest approach is to let the client ask for what they truly need.

  _e.g._ The client only ask to display a Grid view of their data, then out of curiosity, we added an existing widget plugin to export the grid data as CSV. -- When the client see this, they will ask to support more formats. Let say they need this in PDF format, then later on they will ask if possible to add headers with their logo, then later on they will ask if possible to add the table of contents, yadayadayada..

### 2. Lowering the Impact of Code Refactor

Most of the time we start small using the MVC pattern because thats what we learned in school -- minimalistic approach following SOLID, DRY and KISS principles. Then it will slowly grow into a huge in-house framework.

MVC is not a silver bullet in codebase design. You might also heard of other things like DDD and Clean Architecture. Still not a silver bullet to solve our problems.

- MVC is good for small-medium projects.
- DDD and Clean Architecture is good for enterprise projects. I'm a fan of DDD, but the way the internet articles describe it makes it seems like an over-engineered solution for everything.

There is a sweet spot in DDD and Clean Architecture, We don't need to follow everything, just pick what's necessary.

#### Given Scenario:

We are working to develop a NodeJS backend that integrates the records coming from biometric devices to an ERP system. Currently we have the documentations for biometric device `Model-X` by `Manufacturer-X`.

Initially we think of a backend structure like this one below.

- M - data models
- V - API return
- C - API controllers

```
backend/
'-- api/
|   '-- controllers/
|   |   '-- x-controller.js
|   '-- middlewares/
|   |   '-- auth.js
|   |   '-- errors.js
|   '-- models/
|       '-- x-model.js
'-- node_modules/
'-- package.json
'-- package-lock.json
'-- index.js
```

Let's discuss the possible issues of the above codebase.

In the long run, the `index.js` will became bloated due to startup procedures and server route definitions. We need to separate them like `server` and `startup`. We prefer the `startup` to be a directory instead of a single file, because we have many things to startup, e.g. environment configuration and subprocess entrypoints.

```
backend/
'-- api/
|   '-- controllers/
|   |   '-- x-controller.js
|   '-- middlewares/
|   |   '-- auth.js
|   |   '-- errors.js
|   '-- models/
|       '-- x-model.js
'-- startup/
|   '-- environment.js
'-- node_modules/
'-- index.js
'-- package.json
'-- package-lock.json
'-- server.js
```

```js
// file: startup/environment.js

const fs = require("fs");
const envFile = process.argv[2] || ".env";

if (fs.existsSync(envFile)) {
  require("dotenv").config({ path: envFile });
}

// environment:
const _yes = new Set(["true", "1", "yes", "ok"]);
const PORT = Number(process.env.PORT) || 4000;
const USE_TLS = _yes.has(process.env.USE_TLS?.trim());

module.exports = {
  PORT,
  USE_TLS,
};
```

One common mistake that we are not aware of is the way how we use the `path` library to resolve the backend required directories.

```js
// file: ./api/controllers/some_controller.js

const path = require("path");
const somePath = path.join(__dirname, "..", "..", "data");

// do something...
```

The snippet above will be fine if the directory is dedicated only for that controller. Let say in the future we have many controllers referring to that directory. Another scenario, running the same backend scripts but different output directory location. The script above is not flexible enough. To fix that we need to have a common utiliy for directories and a centralized path definitions.

```js
// file: ./commons/utils/directories.js

const fs = require("fs");
const path = require("path");

/**
 * recursively create a directory
 * @returns {string} path
 */
const mkdirSync = (_path) => {
  // create the directory
  if (!fs.existSync(_path)) {
    fs.mkdirSync(_path, { recursive: true });
    console.log("created", _path);
  }
  return _path;
};

/**
 * usage: let resolved = fileExistSync(reportsDir, 'file.ext');
 * @returns {string} path
 */
const fileExistSync = (...paths) => {
  let resolved = path.resolve(path.join(paths));
  if (fs.existSync(resolved)) {
    return resolved;
  }
  return false;
};

module.exports = {
  mkdirSync,
  fileExistSync,
};
```

Initialize the directories along with the startup environments

```js
// file: startup/environment.js

const fs = require("fs");
const envFile = process.argv[2] || ".env";
const path = require("path");
const { mkdirSync } = require("../commons/utils/directories");

if (fs.existsSync(envFile)) {
  require("dotenv").config({ path: envFile });
}

// environment:
const _yes = new Set(["true", "1", "yes", "ok"]);
const PORT = Number(process.env.PORT) || 4000;
const USE_TLS = _yes.has(process.env.USE_TLS?.trim());

// directories:
const rootDir = mkdirSync(path.resolve(path.join(__dirname, "..")));
const logsDir = mkdirSync(process.env.LOGS_DIR || path.join(rootDir, "logs"));
const dataDir = mkdirSync(process.env.DATA_DIR || path.join(rootDir, "data"));
const reportsDir = mkdirSync(
  process.env.UPLOADS_DIR || path.join(dataDir, "reports")
);
const uploadsDir = mkdirSync(
  process.env.UPLOADS_DIR || path.join(dataDir, "uploads")
);

module.exports = {
  PORT,
  USE_TLS,
  rootDir,
  logsDir,
  dataDir,
  reportsDir,
};
```

Then we require the definitions in controllers.

```js
// file: api/controllers/some_controller.js

const { dataDir } = require("../../startup/environment");
const { fileExistSync } = require("../../commons/utils/directories");

// do something...
```

The `controllers` will end up bloated in the long run if we have direct dependencies to data layer `models`. Also we will not be able to create a reusable business logic if we tightly-coupled everything in controllers. In order to resolve this problem, we need a separate layer for the backend logic. We prefer to call it `services`. This directory contains the business logic scripts grouped by bounded context to fullfil the features of our backend service.

Here we prefer to have a filename convention. `./services/<context>/<logic>.js`

Data `models/` are heavly used in the project. We should avoid putting them inside the `api/`. The clean approach is to add the `infrastructure/` directory for services that are not really part of backend service. Like for example database, cache and third-party integrations.

Just think of it like the backend service integrates data to the database service and caching service.

_e.g._ The biometric device has a heartbeat request.

```js
// file: services/device/handleDeviceHeartbeat.js

const { ErrorUnregisteredDevice } = require("../../commons/errors");
const {
  findRegisteredDeviceByCodeAndSerialNumber,
} = require("../../infrastructure/data/models/devices");

/**
 * @typedef {Object} Params
 * @property {string} manufacturer unique code for device manufacturer
 * @property {string} serial_no the serial number of device
 * @property {string} ip_address the ip address of device
 */

/**
 * process device heartbeat
 * @param {Params} params from controllers
 * @param {Object} context a temporary storage for the controller execution context.
 * @returns {boolean} result if success or not.
 */
async function handleDeviceHeartbeat(params, context) {
  let device = await findRegisteredDeviceByCodeAndSerialNumber(
    params.manufacturer,
    params.serial_no
  );

  if (!device) {
    throw new ErrorUnregisteredDevice(
      `${params.manufacturer}/${params.serial_no}`
    );
  }

  // do something...

  return true;
}

module.exports = {
  handleDeviceHeartbeat,
};
```

Using the above approach, we can now reuse the `handleDeviceHeartbeat`. Let say we now have `Model-Z` from `Manufacturer-Z`.

```js
// file: api/controllers/x-controller.js

const MANUFACTURER = "Manufacturer-X";
const {
  handleDeviceHeartbeat,
} = require("../../services/device/handleDeviceHeartbeat");

// based on device Model-X API documentation
router.post("/heartbeat", async (req, res) => {
  try {
    await handleDeviceHeartbeat(
      {
        manufacturer: MANUFACTURER,
        serial_no: req.query.SN,
        ip_address: req.query.IP,
      },
      req
    );
    res.status(200).send("OK");
  } catch (e) {
    if (e instanceof ErrorUnregisteredDevice) {
      res.status(500).send("Unregistered Device");
    } else {
      throw e;
    }
  }
});
```

```js
// file: api/controllers/z-controller.js

const MANUFACTURER = "Manufacturer-Z";
const {
  handleDeviceHeartbeat,
} = require("../../services/device/handleDeviceHeartbeat");

// based on device Model-Z API documentation
router.post("/ping", async (req, res) => {
  try {
    await handleDeviceHeartbeat(
      {
        manufacturer: MANUFACTURER,
        serial_no: req.body.serial,
        ip_address: req.body.from,
      },
      req
    );
    res.status(200).send(1);
  } catch (e) {
    if (e instanceof ErrorUnregisteredDevice) {
      res.status(200).send(0);
    } else {
      throw e;
    }
  }
});
```

Next is the `server.js`. This file contains the route definitions and `HttpServer` instance.

By this way the `index.js` can decide how the server will listen -- single process or cluster.

This approach also allow us to use the existing node modules like supertest for the backend testing.

```js
// file: server.js

const { USE_TLS } = require("./startup/environment");
const fs = require("fs");
const express = require("express");
const router = express.Router();
const app = express();

// disable server tokens
app.set("x-powered-by", false);

// request body parser
router.use(express.json());

// routes
router.use("/x", require("./api/controllers/x-controller"));
router.use("/z", require("./api/controllers/z-controller"));

// central error handler
router.use(require("./api/middlewares/errors"));

app.use(router);

const server = USE_TLS
  ? new require("https").Server(app, {
      cert: fs.readFileSync(process.env.SSL_CERT_FILE),
      key: fs.readFileSync(process.env.SSL_KEY_FILE),
    })
  : new require("http").Server(app);

module.exports = {
  server,
};
```

```js
// file: index.js
const { PORT } = require("./startup/environment");
const { server } = require("./server");

server.listen(PORT, () => {
  console.log(`Server Listen port: ${PORT}`);
});
```

There are cases in deployment where we can't leverage the use of Docker image layer cache because we have `.jar` dependencies in our backend. A `lib/` directory around 150mb in size. Modifying a single line in controller script will invalidate the image cache unless we declare many layers in Dockerfile. The solution for this is to put the backend scripts in a sigle directory called `src/` so any changes in scripts will only affect the specific layer.

**Updated Backend Structure:**

```
backend/
'-- lib/
|   '-- jars/
'-- node_modules/
'-- src/
|   '-- api/
|   |   '-- controllers
|   |   |   '-- x-controller.js
|   |   |   '-- z-controller.js
|   |   '-- middlewares/
|   |       '-- auth.js
|   |       '-- errors.js
|   '-- commons/
|   |   '-- errors.js
|   |   '-- utils/
|   |      '-- directories.js
|   '-- infrastructure/
|   |   '-- cache/
|   |   |   '-- cache-interface.js
|   |   |   '-- cache-memory.js
|   |   |   '-- cache-redis.js
|   |   |   '-- index.js
|   |   '-- data/
|   |   |   '-- models/
|   |   |   |   '-- devices.js
|   |   |   '-- db-client.js
|   |   '-- AAA-web-integration/
|   |       '-- models/
|   |       |   '-- timelogs.js
|   |       '-- api-client.js
|   '-- services/
|   |   '-- device/
|   |       '-- handleDeviceHeartbeat.js
|   '-- startup/
|   |   '-- environment.js
|   |   '-- integration-AAA-entrypoint.js
|   '-- index.js
|   '-- server.js
'-- package.json
'-- package-lock.json
```

## Conclusion

Code refactoring is normal, but minimizing the impact is crucial. The directory structure plays a key role in scalability and maintainability.

<center>- end -</center>
