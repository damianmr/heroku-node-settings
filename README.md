# Heroku Node Settings

Since Node 2, the default garbage collection settings allow the app to consume up to 1.5GB before garbage collection occurs. In Heroku this is specially problematic as it might kill your app before that limit is reached (smaller dynos do not have such amount of memory).
As Heroku provides an environment variable `$WEB_MEMORY` to let the system know what the limit is, you can start your app passing the appropriate flags so that this limit is never reached.

## Installation
```
  npm install --save heroku-node-settings
```

## Usage
If you are using the default start mechanism in Heroku (`npm start`) then you only need to change the `start` command in your `package.json` so that it uses `heroku-node-settings` instead of `node`.  

```
  {
    "name": "my-proyect",
    "version": "0.0.1",
    "description": "A web app that does not get killed by high memory consumption in Heroku.",
    "main": "server/bin/web.js",
    "repository": "https://github.com/myuser/myproyect.git",
    "scripts": {
      "test": "grunt test",
      "start": "heroku-node-settings server/bin/web.js"
    }
  }
```
Note that any arguments you pass into this command will be used when calling `node`as well.

If you're using a Procfile, be sure to include the path: `node_modules/.bin/heroku-node-settings`.

## How does it work
Essentially, the script uses the following V8 flags to start node. The values of these flags depends on the `$WEB_MEMORY` the dyno has. Check the source for the details.

- `--max_semi_space_size`: Sets how much memory can new memory allocations take (in `megabytes` ). This flag used to be called `--max_new_space_size` (see https://codereview.chromium.org/271843005).
- `--max_old_space_size`: Sets the upper limit of memory node can use. The problem is that if your app reaches this value then Node will crash (memory allocation errors will start to show in the logs).

Also note that, although these settings constrain the memory usage, there are other elements that can make the whole process use more memory that what these flags set (buffers, files, etc.).

## Acknowledgments
* I took the script from @joanniclaborde on this thread: https://github.com/nodejs/node/issues/3370#issuecomment-158521878
* I took the idea of an NPM package from Heroku-Node: https://github.com/lanetix/heroku-node
* These were very useful links:
  - https://strongloop.com/strongblog/node-js-performance-garbage-collection/
  - https://groups.google.com/forum/#!topic/v8-users/qznbdM9Bio4
  - https://groups.google.com/forum/#!topic/nodejs/sxkqPtC-3Jk
  - http://stackoverflow.com/questions/30252905/nodejs-decrease-v8-garbage-collector-memory-usage
  - http://erikcorry.blogspot.com.ar/2012/11/memory-management-flags-in-v8.html
  - http://fiznool.com/blog/2016/10/01/running-a-node-dot-js-app-in-a-low-memory-environment/
