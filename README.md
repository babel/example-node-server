# Example Node Server w/ Babel

### Getting Started

First we'll install `babel-cli`.

```shell
$ npm install --save-dev babel-cli
```

Along with some [presets](http://babeljs.io/docs/plugins/#presets).

```shell
$ npm install --save-dev babel-preset-es2015 babel-preset-stage-2
```

Then create our server in `index.js`.

```shell
$ touch index.js
```
```js
import http from 'http';

http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n');
}).listen(1337, '127.0.0.1');

console.log('Server running at http://127.0.0.1:1337/');
```

Then we'll add our first `npm start` script in `package.json`.

```diff
  "scripts": {
+   "start": "babel-node index.js --presets es2015,stage-2"
  }
```

Now let's start our server.

```shell
$ npm start
```

You should now be able to visit `http://127.0.0.1:1337` and see `Hello World`.

### Watching file changes with `nodemon`

We can improve our `npm start` script with `nodemon`.

```shell
$ npm install --save-dev nodemon
```

Then we can update our `npm start` script.

```diff
  "scripts": {
-   "start": "babel-node index.js"
+   "start": "nodemon index.js --exec babel-node --presets es2015,stage-2"
  }
```

Then we'll restart our server.

```shell
$ npm start
```

You should now be able to make changes to `index.js` and our server should be
restarted automatically by `nodemon`.

Go ahead and replace `Hello World` with `Hello {{YOUR_NAME_HERE}}` while our
server is running.

If you visit `http://127.0.0.1:1337` you should see our server greeting you.

### Getting ready for production use

So we've cheated a little bit by using `babel-node`. While this is great for
getting something going, it's not a good idea to use it in production.

We should be precompiling our files, so let's do that now.

First let's move our server `index.js` file to `lib/index.js`.

```shell
$ mv index.js lib/index.js
```

And update our `npm start` script to reflect the location change.

```diff
  "scripts": {
-   "start": "nodemon index.js --exec babel-node --presets es2015,stage-2"
+   "start": "nodemon lib/index.js --exec babel-node --presets es2015,stage-2"
  }
```

Next let's add two new tasks, `npm run build` and `npm run serve`.

```diff
  "scripts": {
    "start": "nodemon lib/index.js --exec babel-node --presets es2015,stage-2",
+   "build": "babel lib -d dist --presets es2015,stage-2",
+   "serve": "node dist/index.js"
  }
```

Now we can use `npm run build` for precompiling our assets, and `npm run serve`
for starting our server in production.

```shell
$ npm run build
$ npm run serve
```

This means we can quickly restart our server without waiting for `babel` to
recompile our files.

Oh, let's not forget to add `dist` to our `.gitignore` file:

```shell
$ touch .gitignore
```

```
dist
```

This will make sure we don't accidentally commit our built files to git.

### Saving Babel options to `.babelrc`

Let's create a `.babelrc` file.

```shell
$ touch .babelrc
```

This will host any options we might want to configure `babel` with.

```json
{
  "presets": ["es2015", "stage-2"],
  "plugins": []
}
```

Now we can remove the duplicated options from our npm scripts

```diff
  "scripts": {
+   "start": "nodemon lib/index.js --exec babel-node",
+   "build": "babel lib -d dist",
    "serve": "node dist/index.js"
  }
```

### Testing the server

Finally let's make sure our server is well tested.

Let's install `mocha`.

```shell
$ npm install --save-dev mocha
```

And create our test in `test/index.js`.

```shell
$ mkdir test
$ touch test/index.js
```

```js
import http from 'http';
import assert from 'assert';

import '../lib/index.js';

describe('Example Node Server', () => {
  it('should return 200', done => {
    http.get('http://127.0.0.1:1337', res => {
      assert.equal(200, res.statusCode);
      done();
    });
  });
});
```

Next, install `babel-register` for the require hook.

```shell
$ npm install --save-dev babel-register
```

Then we can add an `npm test` script.

```diff
  "scripts": {
    "start": "nodemon lib/index.js --exec babel-node",
    "build": "babel lib -d dist",
    "serve": "node dist/index.js",
+   "test": "mocha --compilers js:babel-register"
  }
```

Now let's run our tests.

```shell
$ npm test
```

You should see the following:

```shell
Server running at http://127.0.0.1:1337/

  Example Node Server
    ✓ should return 200

  1 passing (43ms)
```

That's it!
