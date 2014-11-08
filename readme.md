# upath v0.1.4

[![Build Status](https://travis-ci.org/anodynos/upath.svg?branch=master)](https://travis-ci.org/anodynos/upath)
[![Up to date Status](https://david-dm.org/anodynos/upath.png)](https://david-dm.org/anodynos/upath.png)

A drop-in replacement / proxy to nodejs's `path` that:

  * Replaces the windows `\` with the unix `/` in all string params & results. This has significant positives - see below.

  * Adds **filename extensions** functions `addExt`, `trimExt`, `changeExt`, and `defaultExt`.

  * Add a `normalizeSafe` function to preserve any meaningful leading `./` & a `normalizeTrim` which additionally trims any useless ending `/`.

  * Plus a helper `toUnix` that simply converts `` to `/` and consolidates duplicates.

**Useful note: these docs are actually auto generated from [specs](https://github.com/anodynos/upath/blob/master/source/spec/upath-spec.coffee), running on Linux.**

## Why ?

Normal `path` doesn't convert paths to a unified format (ie `/`) before calculating paths (`normalize`, `join`), which can lead to numerous problems.
Also path joining, normalization etc on the two formats is not consistent, depending on where it runs - last checked with nodejs 0.10.32 running on Linux & Windows x64.
Running on Windows `path` yields different results.

In general, if you code your paths logic while developing on Unix/Mac and it runs on Windows, you may run into problems when using `path`.

Note that using **Unix `/` on Windows** works perfectly inside nodejs (and other languages), so there's no reason to stick to the legacy.

##### Examples / specs

Check out the different (improved) behavior to vanilla `path`:

    `upath.normalize(path)`        --returns-->

          ✓ `'c:/windows/nodejs/path'`          --->     `'c:/windows/nodejs/path'`  // equal to `path.normalize()` 
          ✓ `'c:/windows/../nodejs/path'`       --->             `'c:/nodejs/path'`  // equal to `path.normalize()` 
          ✓ `'c:\\windows\\nodejs\\path'`       --->     `'c:/windows/nodejs/path'`  // `path.normalize()` gives `'c:\windows\nodejs\path'`                                                                                                                     
          ✓ `'c:\\windows\\..\\nodejs\\path'`   --->             `'c:/nodejs/path'`  // `path.normalize()` gives `'c:\windows\..\nodejs\path'`                                                                                                                  
          ✓ `'//windows\\unix/mixed'`           --->        `'/windows/unix/mixed'`  // `path.normalize()` gives `'/windows\unix/mixed'`                                                                                                                        
          ✓ `'\\windows//unix/mixed'`           --->        `'/windows/unix/mixed'`  // `path.normalize()` gives `'\windows/unix/mixed'`                                                                                                                        
          ✓ `'////\\windows\\..\\unix/mixed/'`  --->               `'/unix/mixed/'`  // `path.normalize()` gives `'/\windows\..\unix/mixed/'`                                                                                                                   
        

Joining paths can also be a problem:

    `upath.join(paths...)`        --returns-->

          ✓ `'some/nodejs/deep', '../path'`      --->      `'some/nodejs/path'`  // equal to `path.join()` 
          ✓ `'some/nodejs\\windows', '../path'`  --->      `'some/nodejs/path'`  // `path.join()` gives `'some/path'` 
          ✓ `'some\\windows\\only', '..\\path'`  --->     `'some/windows/path'`  // `path.join()` gives `'some\windows\only/..\path'`                                                                                                                           
    

## Added functions
      

#### `upath.toUnix(path)`

Just converts all `` to `/` and consolidates duplicates, without performing any normalization.

##### Examples / specs

    `upath.toUnix(path)`        --returns-->

        ✓ `'.//windows\//unix//mixed////'`     --->      `'./windows/unix/mixed/'`
        ✓ `'..///windows\..\\unix/mixed'`      --->      `'../windows/../unix/mixed'`
      

#### `upath.normalizeSafe(path)`

Exactly like `path.normalize(path)`, but it keeps the first meaningful `./`.

Note that the unix `/` is returned everywhere, so windows `\` is always converted to unix `/`.

##### Examples / specs & how it differs from vanilla `path`

    `upath.normalizeSafe(path)`        --returns-->

        ✓ `''`                              --->                         `'.'`  // equal to `path.normalize()` 
        ✓ `'.'`                             --->                         `'.'`  // equal to `path.normalize()` 
        ✓ `'./'`                            --->                        `'./'`  // equal to `path.normalize()` 
        ✓ `'.//'`                           --->                        `'./'`  // equal to `path.normalize()` 
        ✓ `'.\\'`                           --->                        `'./'`  // `path.normalize()` gives `'.\'` 
        ✓ `'.\\//'`                         --->                        `'./'`  // `path.normalize()` gives `'.\/'` 
        ✓ `'./..'`                          --->                        `'..'`  // equal to `path.normalize()` 
        ✓ `'.//..'`                         --->                        `'..'`  // equal to `path.normalize()` 
        ✓ `'./../'`                         --->                       `'../'`  // equal to `path.normalize()` 
        ✓ `'.\\..\\'`                       --->                       `'../'`  // `path.normalize()` gives `'.\..\'` 
        ✓ `'./../dep'`                      --->                    `'../dep'`  // equal to `path.normalize()` 
        ✓ `'../dep'`                        --->                    `'../dep'`  // equal to `path.normalize()` 
        ✓ `'../path/dep'`                   --->               `'../path/dep'`  // equal to `path.normalize()` 
        ✓ `'../path/../dep'`                --->                    `'../dep'`  // equal to `path.normalize()` 
        ✓ `'dep'`                           --->                       `'dep'`  // equal to `path.normalize()` 
        ✓ `'path//dep'`                     --->                  `'path/dep'`  // equal to `path.normalize()` 
        ✓ `'./dep'`                         --->                     `'./dep'`  // `path.normalize()` gives `'dep'` 
        ✓ `'./path/dep'`                    --->                `'./path/dep'`  // `path.normalize()` gives `'path/dep'` 
        ✓ `'./path/../dep'`                 --->                     `'./dep'`  // `path.normalize()` gives `'dep'` 
        ✓ `'.//windows\\unix/mixed/'`       --->     `'./windows/unix/mixed/'`  // `path.normalize()` gives `'windows\unix/mixed/'`                                                                                                                             
        ✓ `'..//windows\\unix/mixed'`       --->     `'../windows/unix/mixed'`  // `path.normalize()` gives `'../windows\unix/mixed'`                                                                                                                           
        ✓ `'windows\\unix/mixed/'`          --->       `'windows/unix/mixed/'`  // `path.normalize()` gives `'windows\unix/mixed/'`                                                                                                                             
        ✓ `'..//windows\\..\\unix/mixed'`   --->             `'../unix/mixed'`  // `path.normalize()` gives `'../windows\..\unix/mixed'`                                                                                                                        
      

#### `upath.normalizeTrim(path)`

Exactly like `path.normalizeSafe(path)`, but it trims any useless ending `/`.

##### Examples / specs

    `upath.normalizeTrim(path)`        --returns-->

        ✓ `'./'`                         --->                        `'.'`  // `upath.normalizeSafe()` gives `'./'` 
        ✓ `'./../'`                      --->                       `'..'`  // `upath.normalizeSafe()` gives `'../'` 
        ✓ `'./../dep/'`                  --->                   `'../dep'`  // `upath.normalizeSafe()` gives `'../dep/'` 
        ✓ `'path//dep\\'`                --->                 `'path/dep'`  // `upath.normalizeSafe()` gives `'path/dep/'` 
        ✓ `'.//windows\\unix/mixed/'`    --->     `'./windows/unix/mixed'`  // `upath.normalizeSafe()` gives `'./windows/unix/mixed/'`                                                                                                                          
    

## Added functions for *filename extension* manipulation.

**Happy notes:**

  In all functions you can:

  * use both `.ext` & `ext` - the dot `.` on the extension is always adjusted correctly.

  * omit the `ext` param (pass null/undefined/empty string) and the common sense thing will happen.

  * ignore specific extensions from being considered as valid ones (eg `.min`, `.dev` `.aLongExtIsNotAnExt` etc), hence no trimming or replacement takes place on them.

       

#### `upath.addExt(filename, [ext])`

Adds `.ext` to `filename`, but only if it doesn't already have the exact extension.

##### Examples / specs

    `upath.addExt(filename, 'js')`     --returns-->

        ✓ `'myfile/addExt'`          --->          `'myfile/addExt.js'` 
        ✓ `'myfile/addExt.txt'`      --->      `'myfile/addExt.txt.js'` 
        ✓ `'myfile/addExt.js'`       --->          `'myfile/addExt.js'` 
        ✓ `'myfile/addExt.min.'`     --->     `'myfile/addExt.min..js'` 
        

It adds nothing if no `ext` param is passed.

    `upath.addExt(filename)`           --returns-->

          ✓ `'myfile/addExt'`          --->             `'myfile/addExt'` 
          ✓ `'myfile/addExt.txt'`      --->         `'myfile/addExt.txt'` 
          ✓ `'myfile/addExt.js'`       --->          `'myfile/addExt.js'` 
          ✓ `'myfile/addExt.min.'`     --->        `'myfile/addExt.min.'` 
      

#### `upath.trimExt(filename, [ignoreExts], [maxSize=7])`

Trims a filename's extension.

  * Extensions are considered to be up to `maxSize` chars long, counting the dot (defaults to 7).

  * An `Array` of `ignoreExts` (eg `['.min']`) prevents these from being considered as extension, thus are not trimmed.

##### Examples / specs

    `upath.trimExt(filename)`          --returns-->

        ✓ `'my/trimedExt.txt'`            --->                `'my/trimedExt'` 
        ✓ `'my/trimedExt'`                --->                `'my/trimedExt'` 
        ✓ `'my/trimedExt.min'`            --->                `'my/trimedExt'` 
        ✓ `'my/trimedExt.min.js'`         --->            `'my/trimedExt.min'` 
        ✓ `'../my/trimedExt.longExt'`     --->     `'../my/trimedExt.longExt'` 
        

It is ignoring `.min` & `.dev` as extensions, and considers exts with up to 8 chars.

    `upath.trimExt(filename, ['min', '.dev'], 8)`          --returns-->

          ✓ `'my/trimedExt.txt'`             --->                 `'my/trimedExt'` 
          ✓ `'my/trimedExt.min'`             --->             `'my/trimedExt.min'` 
          ✓ `'my/trimedExt.dev'`             --->             `'my/trimedExt.dev'` 
          ✓ `'../my/trimedExt.longExt'`      --->              `'../my/trimedExt'` 
          ✓ `'../my/trimedExt.longRExt'`     --->     `'../my/trimedExt.longRExt'` 
      

#### `upath.changeExt(filename, [ext], [ignoreExts], [maxSize=7])`

Changes a filename's extension to `ext`. If it has no (valid) extension, it adds it.

  * Valid extensions are considered to be up to `maxSize` chars long, counting the dot (defaults to 7).

  * An `Array` of `ignoreExts` (eg `['.min']`) prevents these from being considered as extension, thus are not changed - the new extension is added instead.

##### Examples / specs

    `upath.changeExt(filename, '.js')`  --returns-->

        ✓ `'my/module.min'`           --->               `'my/module.js'` 
        ✓ `'my/module.coffee'`        --->               `'my/module.js'` 
        ✓ `'my/module'`               --->               `'my/module.js'` 
        ✓ `'file/withDot.'`           --->            `'file/withDot.js'` 
        ✓ `'file/change.longExt'`     --->     `'file/change.longExt.js'` 
        

If no `ext` param is given, it trims the current extension (if any).

    `upath.changeExt(filename)`        --returns-->

          ✓ `'my/module.min'`           --->                  `'my/module'` 
          ✓ `'my/module.coffee'`        --->                  `'my/module'` 
          ✓ `'my/module'`               --->                  `'my/module'` 
          ✓ `'file/withDot.'`           --->               `'file/withDot'` 
          ✓ `'file/change.longExt'`     --->        `'file/change.longExt'` 
        

It is ignoring `.min` & `.dev` as extensions, and considers exts with up to 8 chars.

    `upath.changeExt(filename, 'js', ['min', '.dev'], 8)`        --returns-->

          ✓ `'my/module.coffee'`         --->                `'my/module.js'` 
          ✓ `'file/notValidExt.min'`     --->     `'file/notValidExt.min.js'` 
          ✓ `'file/notValidExt.dev'`     --->     `'file/notValidExt.dev.js'` 
          ✓ `'file/change.longExt'`      --->              `'file/change.js'` 
          ✓ `'file/change.longRExt'`     --->     `'file/change.longRExt.js'` 
      

#### `upath.defaultExt(filename, [ext], [ignoreExts], [maxSize=7])`

Adds `.ext` to `filename`, only if it doesn't already have _any_ *old* extension.

  * (Old) extensions are considered to be up to `maxSize` chars long, counting the dot (defaults to 7).

  * An `Array` of `ignoreExts` (eg `['.min']`) will force adding default `.ext` even if one of these is present.

##### Examples / specs

    `upath.defaultExt(filename, 'js')`   --returns-->

        ✓ `'fileWith/defaultExt'`             --->             `'fileWith/defaultExt.js'` 
        ✓ `'fileWith/defaultExt.js'`          --->             `'fileWith/defaultExt.js'` 
        ✓ `'fileWith/defaultExt.min'`         --->            `'fileWith/defaultExt.min'` 
        ✓ `'fileWith/defaultExt.longExt'`     --->     `'fileWith/defaultExt.longExt.js'` 
        

If no `ext` param is passed, it leaves filename intact.

    `upath.defaultExt(filename)`       --returns-->

          ✓ `'fileWith/defaultExt'`             --->                `'fileWith/defaultExt'` 
          ✓ `'fileWith/defaultExt.js'`          --->             `'fileWith/defaultExt.js'` 
          ✓ `'fileWith/defaultExt.min'`         --->            `'fileWith/defaultExt.min'` 
          ✓ `'fileWith/defaultExt.longExt'`     --->        `'fileWith/defaultExt.longExt'` 
        

It is ignoring `.min` & `.dev` as extensions, and considers exts with up to 8 chars.

    `upath.defaultExt(filename, 'js', ['min', '.dev'], 8)` --returns-->

          ✓ `'fileWith/defaultExt'`              --->              `'fileWith/defaultExt.js'` 
          ✓ `'fileWith/defaultExt.min'`          --->          `'fileWith/defaultExt.min.js'` 
          ✓ `'fileWith/defaultExt.dev'`          --->          `'fileWith/defaultExt.dev.js'` 
          ✓ `'fileWith/defaultExt.longExt'`      --->         `'fileWith/defaultExt.longExt'` 
          ✓ `'fileWith/defaultExt.longRext'`     --->     `'fileWith/defaultExt.longRext.js'` 


  87 passing (37ms)