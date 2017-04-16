# api documentation for  [gulp-concat-css (v2.3.0)](https://github.com/mariocasciaro/gulp-concat-css)  [![npm package](https://img.shields.io/npm/v/npmdoc-gulp-concat-css.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-gulp-concat-css) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-gulp-concat-css.svg)](https://travis-ci.org/npmdoc/node-npmdoc-gulp-concat-css)
#### Concatenate css files, rebasing urls and inlining @import

[![NPM](https://nodei.co/npm/gulp-concat-css.png?downloads=true&downloadRank=true&stars=true)](https://www.npmjs.com/package/gulp-concat-css)

[![apidoc](https://npmdoc.github.io/node-npmdoc-gulp-concat-css/build/screenCapture.buildCi.browser.%252Ftmp%252Fbuild%252Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-gulp-concat-css/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-gulp-concat-css/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-gulp-concat-css/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Mario Casciaro"
    },
    "bugs": {
        "url": "https://github.com/mariocasciaro/gulp-concat-css/issues"
    },
    "dependencies": {
        "gulp-util": "~3.0.1",
        "lodash.defaults": "^3.0.0",
        "parse-import": "^2.0.0",
        "rework": "~1.0.0",
        "rework-import": "^2.0.0",
        "rework-plugin-url": "^1.0.1",
        "through2": "~1.1.1"
    },
    "description": "Concatenate css files, rebasing urls and inlining @import",
    "devDependencies": {
        "chai": "^1.10.0",
        "mocha": "^2.1.0"
    },
    "directories": {},
    "dist": {
        "shasum": "4c1586121a8411ff4b2dc44fcfa4dc740e8fe1b6",
        "tarball": "https://registry.npmjs.org/gulp-concat-css/-/gulp-concat-css-2.3.0.tgz"
    },
    "gitHead": "c4171769510e6921b7c33610acfb6fd6a42237be",
    "homepage": "https://github.com/mariocasciaro/gulp-concat-css",
    "keywords": [
        "gulpplugin",
        "concat",
        "css",
        "import",
        "merge",
        "inline"
    ],
    "licenses": [
        {
            "type": "MIT",
            "url": "https://github.com/mariocasciaro/gulp-concat-css/blob/master/LICENSE"
        }
    ],
    "maintainers": [
        {
            "name": "mariocasciaro"
        }
    ],
    "name": "gulp-concat-css",
    "optionalDependencies": {},
    "repository": {
        "type": "git",
        "url": "git://github.com/mariocasciaro/gulp-concat-css.git"
    },
    "scripts": {
        "test": "node_modules/mocha/bin/mocha test/*.js --reporter spec"
    },
    "version": "2.3.0"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module gulp-concat-css](#apidoc.module.gulp-concat-css)
1.  [function <span class="apidocSignatureSpan"></span>gulp-concat-css (destFile, options)](#apidoc.element.gulp-concat-css.gulp-concat-css)
1.  [function <span class="apidocSignatureSpan">gulp-concat-css.</span>toString ()](#apidoc.element.gulp-concat-css.toString)



# <a name="apidoc.module.gulp-concat-css"></a>[module gulp-concat-css](#apidoc.module.gulp-concat-css)

#### <a name="apidoc.element.gulp-concat-css.gulp-concat-css"></a>[function <span class="apidocSignatureSpan"></span>gulp-concat-css (destFile, options)](#apidoc.element.gulp-concat-css.gulp-concat-css)
- description and source-code
```javascript
gulp-concat-css = function (destFile, options) {
  var buffer = [];
  var firstFile, commonBase;
  var destDir = path.dirname(destFile);
  var urlImportRules = [];
  options = defaults({}, options, {
    inlineImports: true,
    rebaseUrls: true,
    includePaths: []
  });

  return through.obj(function(file, enc, cb) {
    var processedCss;

    if (file.isStream()) {
      this.emit('error', new gutil.PluginError('gulp-concat-css', 'Streaming not supported'));
      return cb();
    }

    if(!firstFile) {
      firstFile = file;
      commonBase = file.base;
    }

    function urlPlugin(file) {
      return reworkUrl(function(url) {
        if(isUrl(url) || isDataURI(url) || path.extname(url) === '.css' || path.resolve(url) === url) {
          return url;
        }

        var resourceAbsUrl = path.relative(commonBase, path.resolve(path.dirname(file), url));
        resourceAbsUrl = path.relative(destDir, resourceAbsUrl);
        //not all systems use forward slash as path separator
        //this is required by urls.
        if(path.sep === '\\'){
          //replace with forward slash
          resourceAbsUrl = resourceAbsUrl.replace(/\\/g, '/');
        }
        return resourceAbsUrl;
      });
    }


    function collectImportUrls(styles) {
      var outRules = [];
      styles.rules.forEach(function(rule) {
        if(rule.type !== 'import') {
          return outRules.push(rule);
        }

        var importData = parseImport('@import ' + rule.import + ';');
        var importPath = importData && importData[0].path;
        if(isUrl(importPath) || !options.inlineImports) {
          return urlImportRules.push(rule);
        }
        return outRules.push(rule);
      });
      styles.rules = outRules;
    }


    function processNestedImport(contents) {
      var rew = rework(contents,{source:this.source});//find the css file has syntax errors
      if(options.rebaseUrls) {
        rew = rew.use(urlPlugin(this.source));
      }
      rew = rew.use(collectImportUrls);
      return rew.toString();
    }

    try {
      processedCss = rework(String(file.contents||""),{source:file.path});//find the css file has syntax errors
      if(options.rebaseUrls) {
        processedCss = processedCss.use(urlPlugin(file.path));
      }

      processedCss = processedCss.use(collectImportUrls);

      if(options.inlineImports) {
        processedCss = processedCss.use(reworkImport({
          path: [
            '.',
            path.dirname(file.path)
          ].concat(options.includePaths),
          transform: processNestedImport
        }))
          .toString();
      }

      processedCss = processedCss.toString();
    } catch(err) {
      this.emit('error', new gutil.PluginError('gulp-concat-css', err));
      return cb();
    }

    buffer.push(processedCss);
    cb();
  }, function(cb) {
    if(!firstFile) {
      return cb();
    }

    var contents = urlImportRules.map(function(rule) {
      return '@import ' + rule.import + ';';
    }).concat(buffer).join(gutil.linefeed);

    var concatenatedFile = new gutil.File({
      base: firstFile.base,
      cwd: firstFile.cwd,
      path: path.join(firstFile.base, destFile),
      contents: new Buffer(contents)
    });
    this.push(concatenatedFile);
    cb();
  });
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.gulp-concat-css.toString"></a>[function <span class="apidocSignatureSpan">gulp-concat-css.</span>toString ()](#apidoc.element.gulp-concat-css.toString)
- description and source-code
```javascript
toString = function () {
    return toString;
}
```
- example usage
```shell
...

function processNestedImport(contents) {
  var rew = rework(contents,{source:this.source});//find the css file has syntax errors
  if(options.rebaseUrls) {
    rew = rew.use(urlPlugin(this.source));
  }
  rew = rew.use(collectImportUrls);
  return rew.toString();
}

try {
  processedCss = rework(String(file.contents||""),{source:file.path});//find the css file has syntax errors
  if(options.rebaseUrls) {
    processedCss = processedCss.use(urlPlugin(file.path));
  }
...
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
