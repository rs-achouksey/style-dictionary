# API

### buildAllPlatforms 
> StyleDictionary.buildAllPlatforms() ⇒ [<code>style-dictionary</code>](#module_style-dictionary)




The only top-level method that needs to be called
to build the Style Dictionary.

**Example**  
```js
const StyleDictionary = require('style-dictionary').extend('config.json');
StyleDictionary.buildAllPlatforms();
```

* * *

### buildPlatform 
> StyleDictionary.buildPlatform(platform) ⇒ [<code>style-dictionary</code>](#module_style-dictionary)




Takes a platform and performs all transforms to
the properties object (non-mutative) then
builds all the files and performs any actions. This is useful if you only want to
build the artifacts of one platform to speed up the build process.

This method is also used internally in [buildAllPlatforms](#buildAllPlatforms) to
build each platform defined in the config.


| Param | Type | Description |
| --- | --- | --- |
| platform | <code>String</code> | Name of the platform you want to build. |

**Example**  
```js
StyleDictionary.buildPlatform('web');
```
```bash
$ style-dictionary build --platform web
```

* * *

### cleanAllPlatforms 
> StyleDictionary.cleanAllPlatforms() ⇒ [<code>style-dictionary</code>](#module_style-dictionary)




Does the reverse of [buildAllPlatforms](#buildAllPlatforms) by
performing a clean on each platform. This removes all the files
defined in the platform and calls the undo method on any actions.


* * *

### cleanPlatform 
> StyleDictionary.cleanPlatform(platform) ⇒ [<code>style-dictionary</code>](#module_style-dictionary)




Takes a platform and performs all transforms to
the properties object (non-mutative) then
cleans all the files and perfoms the undo method of any [actions](actions.md).


| Param | Type |
| --- | --- |
| platform | <code>String</code> | 


* * *

### exportPlatform 
> StyleDictionary.exportPlatform(platform) ⇒ <code>Object</code>




Exports a properties object with applied
platform transforms.

This is useful if you want to use a style
dictionary in JS build tools like webpack.


| Param | Type | Description |
| --- | --- | --- |
| platform | <code>String</code> | The platform to be exported. Must be defined on the style dictionary. |


* * *

### extend 
> StyleDictionary.extend(config) ⇒ [<code>style-dictionary</code>](#module_style-dictionary)




Create a Style Dictionary


| Param | Type | Description |
| --- | --- | --- |
| config | [<code>Config</code>](#Config) | Configuration options to build your style dictionary. If you pass a string, it will be used as a path to a JSON config file. You can also pass an object with the configuration. |

**Example**  
```js
const StyleDictionary = require('style-dictionary').extend('config.json');

const StyleDictionary = require('style-dictionary').extend({
  source: ['properties/*.json'],
  platforms: {
    scss: {
      transformGroup: 'scss',
      buildPath: 'build/',
      files: [{
        destination: 'variables.scss',
        format: 'scss/variables'
      }]
    }
    // ...
  }
});
```

* * *

### registerAction 
> StyleDictionary.registerAction(action) ⇒ [<code>style-dictionary</code>](#module_style-dictionary)




Adds a custom action to the style property builder. Custom
actions can do whatever you need, such as: copying files,
base64'ing files, running other build scripts, etc.
After you register a custom action, you then use that
action in a platform your config.json

Actions run after the files in a platform are generated so you
can perform operations on files generated by the style dictionary.
Actions are run sequentially, if you write synchronous code then
it will block other actions, or if you use asynchronous code like Promises
it will not block.


| Param | Type | Description |
| --- | --- | --- |
| action | <code>Object</code> |  |
| action.name | <code>String</code> | The name of the action |
| action.do | <code>function</code> | The action in the form of a function. |
| [action.undo] | <code>function</code> | A function that undoes the action. |

**Example**  
```js
StyleDictionary.registerAction({
  name: 'copy_assets',
  do: function(dictionary, config) {
    console.log('Copying assets directory');
    fs.copySync('assets', config.buildPath + 'assets');
  },
  undo: function(dictionary, config) {
    console.log('Cleaning assets directory');
    fs.removeSync(config.buildPath + 'assets');
  }
});
```

* * *

### registerFormat 
> StyleDictionary.registerFormat(format) ⇒ [<code>style-dictionary</code>](#module_style-dictionary)




Add a custom format to the style dictionary


| Param | Type | Description |
| --- | --- | --- |
| format | <code>Object</code> |  |
| format.name | <code>String</code> | Name of the format to be referenced in your config.json |
| format.formatter | <code>function</code> | Function to perform the format. Takes 2 arguments, `dictionary` and `config` Must return a string. |

**Example**  
```js
StyleDictionary.registerFormat({
  name: 'json',
  formatter: function(dictionary, config) {
    return JSON.stringify(dictionary.properties, null, 2);
  }
})
```

* * *

### registerTemplate 
> StyleDictionary.registerTemplate(template) ⇒ [<code>style-dictionary</code>](#module_style-dictionary)




Add a custom template to the Style Dictionary


| Param | Type | Description |
| --- | --- | --- |
| template | <code>Object</code> |  |
| template.name | <code>String</code> | The name of your template. You will refer to this in your config.json file. |
| template.template | <code>String</code> | Path to your lodash template |

**Example**  
```js
StyleDictionary.registerTemplate({
  name: 'Swift/colors',
  template: __dirname + '/templates/swift/colors.template'
});
```

* * *

### registerTransform 
> StyleDictionary.registerTransform(transform) ⇒ [<code>style-dictionary</code>](#module_style-dictionary)




Add a custom transform to the Style Dictionary
Transforms can manipulate a property's name, value, or attributes


| Param | Type | Description |
| --- | --- | --- |
| transform | <code>Object</code> | Transform object |
| transform.type | <code>String</code> | Type of transform, can be: name, attribute, or value |
| transform.name | <code>String</code> | Name of the transformer so a transformGroup can call a list of transforms. |
| [transform.matcher] | <code>function</code> | Matcher function, return boolean if transform should be applied. If you omit the matcher function, it will match all properties. |
| transform.transformer | <code>function</code> | Performs a transform on a property object, should return a string or object depending on the type. Will only update certain properties so you can't mess up property objects on accident. |

**Example**  
```js
StyleDictionary.registerTransform({
  name: 'time/seconds',
  type: 'value',
  matcher: function(prop) {
    return prop.attributes.category === 'time';
  },
  transformer: function(prop) {
    // Note the use of prop.original.value,
    // before any transforms are performed, the build system
    // clones the original property to the 'original' attribute.
    return (parseInt(prop.original.value) / 1000).toString() + 's';
  }
});
```

* * *

### registerTransformGroup 
> StyleDictionary.registerTransformGroup(transformGroup) ⇒ [<code>style-dictionary</code>](#module_style-dictionary)




Add a custom transformGroup to the Style Dictionary, which is a
group of transforms.


| Param | Type | Description |
| --- | --- | --- |
| transformGroup | <code>Object</code> |  |
| transformGroup.name | <code>String</code> | Name of the transform group that will be referenced in config.json |
| transformGroup.transforms | <code>Array.&lt;String&gt;</code> | Array of strings that reference the name of transforms to be applied in order. Transforms must be defined and match the name or there will be an error at build time. |

**Example**  
```js
StyleDictionary.registerTransformGroup({
  name: 'Swift',
  transforms: [
    'attribute/cti',
    'size/pt',
    'name/cti'
  ]
});
```

* * *
