bunglespecs
===========

This repository holds package description files (specs) for bungle.

# File format

## Basics

Package descriptions are saved in JSON format and are compatible to the
CommonJS `package.json` format used by npm. All bungle specific configuration
is stored under the key `bungle`. A minimal package description consists only
of a URI that tells bungle from where to download the package sources. 
For example, jquery-1.9.1 can be described by:

    {
        "bungle": {
        "uri": "http://code.jquery.com/jquery-1.9.1.js"
    }

Bungle supports checking the integrity of the sources it downloads, and
therefore you can specify the expected SHA1 hash of the file that is being
downloaded:

    {
        "bungle": {
            "uri": "http://code.jquery.com/jquery-1.9.1.js",
            "sha1": "9257afd2d46c3a189ec0d40a45722701d47e9ca5"
        }
    }

If the packages sources consist of multiple files, you have to tell bungle with
file should be loaded if the user depends on the package name. This is
accomplished with the `main` key. Consider an example package that contains two
javascript modules (`foo.js` and `bar.js`) and is described by `foobar.json`:

    {
        "bungle": {
            "uri": "https://example.com/foobar.zip",
            "sha1": "94d14359b54ab7f7bd4e8c8h58b05c40ebda6f3d",
            "main": "foo.js"
        }
    }

When you require `foobar` in your source code you will get what is returned by
the module `foo.js`.

    define(['foobar'], function(foobar) {
        // the variable foobar contains what is returned by foo.js
    });

If you want to reference `bar.js` from your source code you can use:

    define(['foobar/bar'], function(bar) {
        // ...
    });

## Depending on other packages

Most packages depend on others and you can tell bungle in your specs about it
with the `dependencies` key:

    {
        "bungle": {
            "uri": "https://example.com/foobar.zip",
            "sha1": "26d345c2bd78f6c5ad8a38cdfh35a46ccf06fad7",
            "dependencies": [ "jquery" ],
            "main": "foo.js"
        }
    }

The above description tells bungle that it has to make sure that `jquery` is
installed when it is installing `foobar`.

## Making foreign code compatible to requirejs

When packaging code that was created by a third party without requirejs in
mind, it is often necessary to apply small patches to the package sources. The
philosophy behind bungle is to keep the upstream packages as they were created
by the original authors and apply the patches at install time. This can be
accomplished easily with install hooks:

    {
        "bungle": {
            "uri": "https://example.com/foobar.zip",
            "sha1": "26d345c2bd78f6c5ad8a38cdfh35a46ccf06fad7",
            "dependencies": [ "pagedown" ],
            "installHooks": [
                [ "wrapDefine", "foo.patched.js", "foo.js", "pagedown as Markdown" ]
            ],
            "main": "foo.patched.js"
        }
    }

Consider the case where the package foobar and its source file `foo.js` are
unaware of requirejs and are using the Markdown object from the pagedown
package. To make `foo.js` compatible with requirejs it is necessary to wrap its
contents in a define call. The define call has to depend on pagedown and make
the pagedown module available to the `foo.js` code as the variable Markdown.
The install hook in the above example creates `foo.patched.js`:

    define(['pagedown'], function(Markdown) {
        // original content of foo.js
    });

Another commonly used hook is a CommonJS wrapper which can be defined by

    [ "wrapDefineCommonJS", "bar.patched.js", "bar.js" ]

and creates the following patched content

    define(function(require, exports, module) {
        // original content of bar.js
    });

