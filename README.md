# gulp-istanbul [![NPM version][npm-image]][npm-url] [![Build Status][travis-image]][travis-url] [![Dependency Status][depstat-image]][depstat-url]

基于 gulp-istanbul@1.1.3 版本修改，增加覆盖率上报逻辑代码，并利用 istanbul 库，注入到各个经过 gulp 处理的 js 文件，主要修改内容如下：

1. 增加可选参数 `relativePathPrefix` ，用于在相对路径的前面加上特定的前缀，比如 `code/`，其中可以选填某些已配置好的 git 相关的参数，比如 `store/${project_name}/${branch}/code`，参数如下：
   1. `${commit_hash}`，会替换成 commit 的 hash 值
   2. `${version}`，会替换成 git 仓库的 version
   3. `${branch}`，会替换成当前分支名
   4. `${last_commit_datetime}`，会替换成上次提交的时间
   5. `${remote}`，会替换成远程仓库的地址
   6. `${project_name}`，会替换成项目名
2. 增加可选参数 `incrementCoverageDir`，表示生成增量代码覆盖率时，增量增量代码的生效路径，比如 `src`，表示只有 `src` 下的文件变化才会被计算增量覆盖率，如果不设置，则表示所有文件都会被计算增量覆盖率
3. 增加可选参数 `coverageVariable`，表示覆盖率数据在全局对象下面的变量名，默认是 `__coverage__`
4. 增加可选参数 `reportURL`，覆盖率上报地址
5. 增加可选参数 `autoReportInterval`，覆盖率自动上报的时间间隔（ms）

发布：`不需要打包`

[Istanbul][istanbul] unit test coverage plugin for [gulp][gulp].

Works on top of any Node.js unit test framework.

## Installation

```shell
npm install --save-dev gulp-istanbul
```

## Example

In your `gulpfile.js`:

#### Node.js testing

```javascript
var istanbul = require("gulp-istanbul");
// We'll use mocha in this example, but any test framework will work
var mocha = require("gulp-mocha");

gulp.task("pre-test", function () {
  return (
    gulp
      .src(["lib/**/*.js"])
      // Covering files
      .pipe(istanbul())
      // Force `require` to return covered files
      .pipe(istanbul.hookRequire())
  );
});

gulp.task("test", ["pre-test"], function () {
  return (
    gulp
      .src(["test/*.js"])
      .pipe(mocha())
      // Creating the reports after tests ran
      .pipe(istanbul.writeReports())
      // Enforce a coverage of at least 90%
      .pipe(istanbul.enforceThresholds({ thresholds: { global: 90 } }))
  );
});
```

**Note:** Version 4.x.x of `gulp-mocha` is not supported (see issue [#115](https://github.com/SBoudrias/gulp-istanbul/issues/115) for details). In this example, you should use `gulp-mocha` version 3.0.1 for the time being.

#### Browser testing

For browser testing, you'll need to write the files covered by istanbul in a directory from where you'll serve these files to the browser running the test. You'll also need a way to extract the value of the [coverage variable](#coveragevariable) after the test have runned in the browser.

Browser testing is hard. If you're not sure what to do, then I suggest you take a look at [Karma test runner](http://karma-runner.github.io) - it has built-in coverage using Istanbul.

```javascript
var istanbul = require("gulp-istanbul");

gulp.task("pre-test", function () {
  return (
    gulp
      .src(["lib/**/*.js"])
      // Covering files
      .pipe(istanbul())
      // Write the covered files to a temporary directory
      .pipe(gulp.dest("test-tmp/"))
  );
});

gulp.task("test", ["pre-test"], function () {
  // Make sure your tests files are requiring files from the
  // test-tmp/ directory
  return (
    gulp
      .src(["test/*.js"])
      .pipe(testFramework())
      // Creating the reports after tests ran
      .pipe(istanbul.writeReports())
  );
});
```

#### Source Maps

gulp-istanbul supports [gulp-sourcemaps][gulp-sourcemaps] when instrumenting:

```javascript
gulp.task("pre-test", function () {
  return (
    gulp
      .src(["lib/**/*.js"])
      // optionally load existing source maps
      .pipe(sourcemaps.init())
      // Covering files
      .pipe(istanbul())
      .pipe(sourcemaps.write("."))
      // Write the covered files to a temporary directory
      .pipe(gulp.dest("test-tmp/"))
  );
});
```

## API

### istanbul(opt)

Instrument files passed in the stream.

#### opt

Type: `Object` (optional)

```js
{
  coverageVariable: 'someVariable',
  ...other Instrumeter options...
}
```

##### coverageVariable

Type: `String` (optional)
Default: `'$$cov_' + new Date().getTime() + '$$'`

The global variable istanbul uses to store coverage

See also:

- [istanbul coverageVariable][istanbul-coverage-variable]
- [SanboxedModule][sandboxed-module-coverage-variable]

##### includeUntested

Type: `Boolean` (optional)
Default: `false`

Flag to include test coverage of files that aren't `require`d by any tests

See also:

- [istanbul "0% coverage" issue](https://github.com/gotwarlost/istanbul/issues/112)

##### instrumenter

Type: `Instrumenter` (optional)
Default: `istanbul.Instrumenter`

Custom Instrumenter to be used instead of the default istanbul one.

```js
var isparta = require("isparta");
var istanbul = require("gulp-istanbul");

gulp.src("lib/**.js").pipe(
  istanbul({
    // supports es6
    instrumenter: isparta.Instrumenter,
  })
);
```

See also:

- [isparta](https://github.com/douglasduteil/isparta)

##### Other Istanbul Instrumenter options

See:

- [istanbul Instrumenter documentation][istanbul-coverage-variable]

### istanbul.hookRequire()

Overwrite `require` so it returns the covered files. The method take an optional [option object](https://gotwarlost.github.io/istanbul/public/apidocs/classes/Hook.html#method_hookRequire).

Always use this option if you're running tests in Node.js

### istanbul.summarizeCoverage(opt)

get coverage summary details

#### opt

Type: `Object` (optional)

```js
{
  coverageVariable: "someVariable";
}
```

##### coverageVariable

Type: `String` (optional)
Default: `'$$cov_' + new Date().getTime() + '$$'`

The global variable istanbul uses to store coverage

See also:

- [istanbul coverageVariable][istanbul-coverage-variable]
- [SanboxedModule][sandboxed-module-coverage-variable]

#### returns

Type: `Object`

```js
{
  lines: { total: 4, covered: 2, skipped: 0, pct: 50 },
  statements: { total: 4, covered: 2, skipped: 0, pct: 50 },
  functions: { total: 2, covered: 0, skipped: 0, pct: 0 },
  branches: { total: 0, covered: 0, skipped: 0, pct: 100 }
}
```

See also:

- [istanbul utils.summarizeCoverage()][istanbul-summarize-coverage]

### istanbul.writeReports(opt)

Create the reports on stream end.

#### opt

Type: `Object` (optional)

```js
{
  dir: './coverage',
  reporters: [ 'lcov', 'json', 'text', 'text-summary', CustomReport ],
  reportOpts: { dir: './coverage' },
  coverageVariable: 'someVariable'
}
```

You can pass individual configuration to a reporter.

```js
{
  dir: './coverage',
  reporters: [ 'lcovonly', 'json', 'text', 'text-summary', CustomReport ],
  reportOpts: {
    lcov: {dir: 'lcovonly', file: 'lcov.info'},
    json: {dir: 'json', file: 'converage.json'}
  },
  coverageVariable: 'someVariable'
}
```

##### dir

Type: `String` (optional)
Default: `./coverage`

The folder in which the reports are to be outputted.

##### reporters

Type: `Array` (optional)
Default: `[ 'lcov', 'json', 'text', 'text-summary' ]`

The list of available reporters:

- `clover`
- `cobertura`
- `html`
- `json`
- `lcov`
- `lcovonly`
- `none`
- `teamcity`
- `text`
- `text-summary`

You can also specify one or more custom reporter objects as items in the array. These will be automatically registered with istanbul.

See also `require('istanbul').Report.getReportList()`

##### reportOpts

Type: `Object` (optional)

```js
{
  dir: "./coverage";
}
```

You can also configure separate directory for each report.

```js
{
  html: {
        dir: './coverage/html',
        watermarks: {
            statements: [ 50, 80 ],
            lines: [ 50, 80 ],
            functions: [ 50, 80],
            branches: [ 50, 80 ]
        }
    },
    lcov: {dir: './coverage/lcov'},
    lcovonly: {dir: './coverage/lcovonly'},
    json: {dir: './coverage/json'},
}
```

`watermarks` can be used to confgure the color of the HTML report.
Default colors are.. RED: below 50% coverage, YELLOW: 50-80% coverage, GREEN: above 80%

##### coverageVariable

Type: `String` (optional)
Default: `'$$cov_' + new Date().getTime() + '$$'`

The global variable istanbul uses to store coverage

See also:

- [istanbul coverageVariable][istanbul-coverage-variable]
- [SanboxedModule][sandboxed-module-coverage-variable]

### istanbul.enforceThresholds(opt)

Checks coverage against minimum acceptable thresholds. Fails the build if any of the thresholds are not met.

#### opt

Type: `Object` (optional)

```js
{
  coverageVariable: 'someVariable',
  thresholds: {
    global: 60,
    each: -10
  }
}
```

##### coverageVariable

Type: `String` (optional)
Default: `'$$cov_' + new Date().getTime() + '$$'`

The global variable istanbul uses to store coverage

##### thresholds

Type: `Object` (required)

Minimum acceptable coverage thresholds. Any coverage values lower than the specified threshold will fail the build.

Each threshold value can be:

- A positive number - used as a percentage
- A negative number - used as the maximum amount of coverage gaps
- A falsey value will skip the coverage

Thresholds can be specified across all files (`global`) or per file (`each`):

```
{
  global: 80,
  each: 60
}
```

You can also specify a value for each metric:

```
{
  global: {
    statements: 80,
    branches: 90,
    lines: 70,
    functions: -10
  }
  each: {
    statements: 100,
    branches: 70,
    lines: -20
  }
}
```

#### emits

A plugin error in the stream if the coverage fails

## License

[MIT License](http://en.wikipedia.org/wiki/MIT_License) (c) Simon Boudrias - 2013

[istanbul]: http://gotwarlost.github.io/istanbul/
[gulp]: https://github.com/gulpjs/gulp
[gulp-sourcemaps]: https://github.com/floridoo/gulp-sourcemaps
[npm-url]: https://npmjs.org/package/gulp-istanbul
[npm-image]: https://badge.fury.io/js/gulp-istanbul.svg
[travis-url]: http://travis-ci.org/SBoudrias/gulp-istanbul
[travis-image]: https://secure.travis-ci.org/SBoudrias/gulp-istanbul.svg?branch=master
[depstat-url]: https://david-dm.org/SBoudrias/gulp-istanbul
[depstat-image]: https://david-dm.org/SBoudrias/gulp-istanbul.svg
[istanbul-coverage-variable]: http://gotwarlost.github.io/istanbul/public/apidocs/classes/Instrumenter.html
[istanbul-summarize-coverage]: http://gotwarlost.github.io/istanbul/public/apidocs/classes/ObjectUtils.html#method_summarizeCoverage
[sandboxed-module-coverage-variable]: https://github.com/felixge/node-sandboxed-module/blob/master/lib/sandboxed_module.js#L240
