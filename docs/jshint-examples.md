
## Usage examples

### Wildcards

In this example, running `grunt lint` will lint the project's Gruntfile as well as all JavaScript files in the `lib` and `test` directories, using the default JSHint `options` and `globals`.

```javascript
// Project configuration.
grunt.initConfig({
  lint: {
    all: ['Gruntfile.js', 'lib/*.js', 'test/*.js']
  }
});
```

With a slight modification, running `grunt lint` will also lint all JavaScript files in the `lib` and `test` directories _and all subdirectories_. See the [minimatch](https://github.com/isaacs/minimatch) module documentation for more details on wildcard patterns.

```javascript
// Project configuration.
grunt.initConfig({
  lint: {
    all: ['Gruntfile.js', 'lib/**/*.js', 'test/**/*.js']
  }
});
```

### Linting before and after concat

In this example, running `grunt lint` will lint two separate sets of files using the default JSHint `options` and `globals`: one "beforeconcat" set, and one "afterconcat" set. Running `grunt lint` will lint both sets of files all at once, because lint is a [multi task](types_of_tasks.md). This is not ideal, because `dist/output.js` may get linted before it gets created via the [concat task](task_concat.md)!

In this case, you should lint the "beforeconcat" set first, then concat, then lint the "afterconcat" set, by running `grunt lint:beforeconcat concat lint:afterconcat`.

```javascript
// Project configuration.
grunt.initConfig({
  concat: {
    dist: {
      src: ['src/foo.js', 'src/bar.js'],
      dest: 'dist/output.js'
    }
  },
  lint: {
    beforeconcat: ['src/foo.js', 'src/bar.js'],
    afterconcat: ['dist/output.js']
  }
});

// Default task.
grunt.registerTask('default', ['lint:beforeconcat', 'concat', 'lint:afterconcat']);
```

_Note: in the above example, a default [alias task](types_of_tasks.md) was created that runs the 'lint:beforeconcat', 'concat' and 'lint:afterconcat' tasks. If you didn't want this to be the default grunt task, you could give it a different name._

### Dynamic filenames

Building on the previous example, if you want to avoid duplication, you can use `<% %>` [template strings](api_template.md) in the config data. This allows you to determine strings like filenames dynamically. In this example, the `concat:dist` destination filename is generated from the `name` and `version` properties of the referenced `package.json` file through the `pkg` config property.

```javascript
// Project configuration.
grunt.initConfig({
  pkg: grunt.file.readJSON('package.json'),
  concat: {
    dist: {
      src: ['src/foo.js', 'src/bar.js'],
      dest: 'dist/<%= pkg.name %>-<%= pkg.version %>.js'
    }
  },
  lint: {
    beforeconcat: ['src/foo.js', 'src/bar.js'],
    afterconcat: ['<%= concat.dist.dest %>']
  }
});
```

### Specifying JSHint options and globals

In this example, taken from the [Sample jQuery plugin Gruntfile](https://github.com/gruntjs/grunt-init-jquery-sample/blob/master/Gruntfile.js), custom JSHint `options` and `globals` are specified. These options are explained in the [JSHint documentation](http://www.jshint.com/options/).

_Note: config `lint.options.options` and `lint.options.globals` apply to the entire project, but can be overridden with per-target options and per-file comments like `/*global exports:false*/`._

```javascript
// Project configuration.
grunt.initConfig({
  lint: {
    all: ['Gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
    options: {
      options: {
        curly: true,
        eqeqeq: true,
        immed: true,
        latedef: true,
        newcap: true,
        noarg: true,
        sub: true,
        undef: true,
        eqnull: true,
        browser: true
      },
      globals: {
        jQuery: true
      }
    }
  }
});
```

#### Per-target JSHint options and globals

To lint per-target, specify your `options` and `globals` in a target-specific `options` object. Default lint options will be overridden by target-specific options.

In this example, there are default JSHint options as well as per-target overrides:

```javascript
// Project configuration.
grunt.initConfig({
  lint: {
    // Default JSHint options.
    options: {
      options: {curly: true},
      globals: {}
    },
    all: {
      files: {
        src: ['src/*.js', 'lib/*.js']
      },
      // Just for the lint:all target.
      options: {
        options: {browser: true},
        globals: {jQuery: true}
      }
    },
    grunt: {
      files: {
        src: 'Gruntfile.js'
      },
      // Just for the lint:grunt target.
      options: {
        options: {node: true},
        globals: {task: true, config: true, file: true, log: true, template: true}
      }
    },
    tests: {
      files: {
        src: 'tests/unit/**/*.js'
      },
      // Just for the lint:tests target.
      options: {
        options: {jquery: true},
        globals: {module: true, test: true, ok: true, equal: true, deepEqual: true, QUnit: true}
      }
    }
  }
});
```

#### Using a `.jshintrc` file

To override lint options with a `.jshintrc` JSHint resource file, add a `jshintrc` setting to your `options` object. Specifying a `.jshintrc` file will override any other JSHint options set in your Gruntfile.

```javascript
// Project configuration.
grunt.initConfig({
  lint: {
    options: {
      // Default JSHint resource file
      jshintrc: '.jshintrc',
      options: {curly: true}
    },
    all: {
      files: {
        src: ['src/*.js', 'lib/*.js']
      },
    },
    tests: {
      files: {
        src: 'tests/unit/**/*.js'
      },
      options: {
        // Will override the defaults for the lint:tests target.
        jshintrc: 'tests/.jshintrc'
      }
    }
  }
});
```

The `.jshintrc` file must be valid JSON and would look something like this:

```json
{
  "curly": true,
  "eqnull": true,
  "eqeqeq": true,
  "expr": true,
  "latedef": true,
  "noarg": true,
  "onevar": true,
  "smarttabs": true,
  "trailing": true,
  "undef": true
}
```

See the [lint task source](../tasks/lint.js) for more information.