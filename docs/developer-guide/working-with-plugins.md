---
title: Working with Plugins
layout: doc
---
<!-- Note: No pull requests accepted for this file. See README.md in the root directory for details. -->

# Working with Plugins

Each plugin is an npm module with a name in the format of `eslint-plugin-<plugin-name>`, such as `eslint-plugin-jquery`. You can also use scoped packages in the format of `@<scope>/eslint-plugin-<plugin-name>` such as `@jquery/eslint-plugin-jquery`.

每个插件是一个命名格式为 `eslint-plugin-<plugin-name>` 的 npm 模块，比如 `eslint-plugin-jquery`。你也可以用这样的格式 `@<scope>/eslint-plugin-<plugin-name>` 限定在包作用域下，比如 `@jquery/eslint-plugin-jquery`。

## Create a Plugin

The easiest way to start creating a plugin is to use the [Yeoman generator](https://npmjs.com/package/generator-eslint). The generator will guide you through setting up the skeleton of a plugin.

创建一个插件最简单的方式是使用 [Yeoman generator](https://npmjs.com/package/generator-eslint)。它将引导你完成插件框架的设置。

### Rules in Plugins

Plugins can expose additional rules for use in ESLint. To do so, the plugin must export a `rules` object containing a key-value mapping of rule ID to rule. The rule ID does not have to follow any naming convention (so it can just be `dollar-sign`, for instance).

在 ESLint 中，插件可以暴露额外的规则以供使用。为此，插件必须输出一个 `rules`对象，包含规则 ID 和对应规则的一个键值对。这个规则 ID 不需要遵循任何命名规范（所以，比如，它可以是 `dollar-sign`）。

```js
module.exports = {
    rules: {
        "dollar-sign": {
            create: function (context) {
                // rule implementation ...
            }
        }
    }
};
```

To use the rule in ESLint, you would use the unprefixed plugin name, followed by a slash, followed by the rule name. So if this plugin were named `eslint-plugin-myplugin`, then in your configuration you'd refer to the rule by the name `myplugin/dollar-sign`. Example: `"rules": {"myplugin/dollar-sign": 2}`.

如果要在 ESLint 中使用插件中的规则，你可以使用不带前缀的插件名，后跟一个 `/`，然后是规则名。所以如果这个插件是 `eslint-plugin-myplugin`，那么在你的配置中你可以使用 `myplugin/dollar-sign` 来引用其中的规则。示例：`"rules": {"myplugin/dollar-sign": "error"}`。

### Environments in Plugins

Plugins can expose additional environments for use in ESLint. To do so, the plugin must export an `environments` object. The keys of the `environments` object are the names of the different environments provided and the values are the environment settings. For example:

插件可以暴露额外的环境以在 ESLint 中使用。为此，插件必须输出一个 `environments` 对象。`environments` 对象的 key 是不同环境提供的名字，值是不同环境的设置。例如：

```js
module.exports = {
    environments: {
        jquery: {
            globals: {
                $: false
            }
        }
    }
};
```

There's a `jquery` environment defined in this plugin. To use the environment in ESLint, you would use the unprefixed plugin name, followed by a slash, followed by the environment name. So if this plugin were named `eslint-plugin-myplugin`, then you would set the environment in your configuration to be `"myplugin/jquery"`.

在这个插件中，定义了一个 `jquery` 环境。为了在 ESLint 中使用这个环境，你可以使用不带前缀的插件名，后跟一个 `/`，然后是环境名。所以如果这个插件是 `eslint-plugin-myplugin`，那么在你的配置中设置环境为 `myplugin/jquery`。

Plugin environments can define the following objects:

插件环境定义以下对象：

1. `globals` - acts the same `globals` in a configuration file. The keys are the names of the globals and the values are `true` to allow the global to be overwritten and `false` to disallow.
1. `globals` - 同配置文件中的 `globals` 一样。key 是全局变量的名字，值为 `true`允许全局变量被覆盖，`false` 不允许覆盖。
1. `parserOptions` - acts the same as `parserOptions` in a configuration file.
1. `parserOptions` - 同配置文件中的 `parserOptions` 一样。

### Processors in Plugins

You can also create plugins that would tell ESLint how to process files other than JavaScript. In order to create a processor, object that is exported from your module has to conform to the following interface:

你也可以创建插件告诉 ESLint 如何处理 JavaScript 之外的文件。为了创建一个处理器，从你的模块中输出的对象必须符合以下接口：

```js
processors: {

    // assign to the file extension you want (.js, .jsx, .html, etc.)
    ".ext": {
        // takes text of the file and filename
        preprocess: function(text, filename) {
            // here, you can strip out any non-JS content
            // and split into multiple strings to lint

            return [string];  // return an array of strings to lint
        },

        // takes a Message[][] and filename
        postprocess: function(messages, filename) {
            // `messages` argument contains two-dimensional array of Message objects
            // where each top-level array item contains array of lint messages related
            // to the text that was returned in array from preprocess() method

            // you need to return a one-dimensional array of the messages you want to keep
            return [Message];
        }
    }
}
```

The `preprocess` method takes the file contents and filename as arguments, and returns an array of strings to lint. The strings will be linted separately but still be registered to the filename. It's up to the plugin to decide if it needs to return just one part, or multiple pieces. For example in the case of processing `.html` files, you might want to return just one item in the array by combining all scripts, but for `.md` file where each JavaScript block might be independent, you can return multiple items.

`preprocess` 将文件内容和文件名称作为参数，返回一个要检查的字符串数组。这些字符串将被分别检查，但仍要注册到文件名。决定它需要返回的只是一部分还是多个块，取决于这个插件。例如，在处理`.html`文件时，通过合并所有的脚本，你可能想要返回数组中的一项，但是对于`.md`文件，每个 JavaScript 块可能是独立的，你可以返回多个项。

The `postprocess` method takes a two-dimensional array of arrays of lint messages and the filename. Each item in the input
array corresponds to the part that was returned from the `preprocess` method. The `postprocess` method must adjust the location of all errors and aggregate them into a single flat array and return it.

`postprocess` 方法需要一个二维数组作为参数，用于检查消息和文件名。传入数组中的每一项对应着从 `preprocess` 方法返回的部分。`postprocess`方法必须调整所有错误的位置并将他们汇集到一个的扁平的数组中，然后返回该数组。

You can have both rules and processors in a single plugin. You can also have multiple processors in one plugin.
To support multiple extensions, add each one to the `processors` element and point them to the same object.

你可以在一个插件中同时有规则和处理器。你也可以一个插件中有多个处理器。
为了支持多个扩展，将每一个处理器添加到 `processors` 元素，然后将它们指向同一个对象。

### Configs in Plugins

You can bundle configurations inside a plugin. This can be useful when you want to provide not just code style, but also some custom rules to support it. You can specify configurations under `configs` key. Please note that when exposing configurations, you have to name each one, and there is no default. So your users will have to specify the name of the configuration they want to use.

你可以在插件中包含配置。但你想提供不止代码风格，还有一些自定义规则时，这会非常有用。你可以在 `configs` 键下指定配置。请注意，当暴露配置时，你需要对它们进行命名，而且是没有默认的。所以，你的用户将需要指定他们想使用的配置的名称。

```js
configs: {
    myConfig: {
        env: ["browser"],
        rules: {
            semi: 2,
            "myPlugin/my-rule": 2,
            "eslint-plugin-myPlugin/another-rule": 2
        }
    }
}
```

**Note:** Please note that configuration will not automatically attach your rules and you have to specify your plugin name and any rules you want to enable that are part of the plugin. Any plugin rules must be prefixed with the short or long plugin name. See [Configuring Plugins](../user-guide/configuring#configuring-plugins)

**注意：**配置不会自动附加你的规则，你必须指定你的插件名和任何你想使用的插件中的规则。任何插件中的规则必须带有插件名或其简写前缀。查看 [Configuring Plugins](../user-guide/configuring#configuring-plugins)。

### Peer Dependency

To make clear that the plugin requires ESLint to work correctly you have to declare ESLint as a `peerDependency` in your `package.json`.
The plugin support was introduced in ESLint version `0.8.0`. Ensure the `peerDependency` points to ESLint `0.8.0` or later.

为了明确插件需要 ESLint 才能正常运行，你必须在你的 `package.json` 中声明将 ESLint 作为一个 `peerDependency`。对插件的支持在 ESLint `0.8.0` 版本中被引入。要确保 `peerDependency` 指向 ESLint `0.8.0` 或之后的版本。

```json
{
    "peerDependencies": {
        "eslint": ">=0.8.0"
    }
}
```

### Testing

You can test the rules of your plugin [the same way as bundled ESLint rules](working-with-rules) using RuleTester.

你可以使用[`ESLintTester`](https://github.com/eslint/eslint-tester)测试你插件中的规则[同测试 ESLint 规则一样](working-with-rules)。

Example:

示例：

```js
"use strict";

var rule = require("../../../lib/rules/custom-plugin-rule"),
    RuleTester = require("eslint").RuleTester;

var ruleTester = new RuleTester();
ruleTester.run("custom-plugin-rule", rule, {
    valid: [
        "var validVariable = true",
    ],

    invalid: [
        {
            code: "var invalidVariable = true",
            errors: [ { message: "Unexpected invalid variable." } ]
        },
        {
            code: "var invalidVariable = true",
            errors: [ { message: /^Unexpected.+variable/ } ]
        }
    ]
});
```

The `RuleTester` constructor accepts an optional object argument, which can be used to specify defaults for your test cases. For example, if all of your test cases use ES2015, you can set it as a default:

`RuleTester` 构造函数接收一个对象参数，用来指定你测试用例的默认配置。比如，如果伱的测试用例都是使用的 ES2015，你可以设置它为一个默认的的配置。

```js
const ruleTester = new RuleTester({ parserOptions: { ecmaVersion: 2015 } });
```

The `RuleTester#run()` method is used to run the tests. It should be passed the following arguments:

`RuleTester#run()` 方法用来运行测试用例。需要传入一下参数：

* The name of the rule (string)
* 规则名 (字符串)
* The rule object itself (see ["working with rules"](./working-with-rules))
* 规则对象 (查看 ["working with rules"](./working-with-rules))
* An object containing `valid` and `invalid` properties, each of which is an array containing test cases.
* 一个包含 `valid` 和 `invalid` 属性的对象，其值是一个包含测试用例的数组。

A test case is an object with the following properties:

测试用例个对象，包含以下属性：

* `code` (string, required): The source code that the rule should be run on
* `code` (string, required)：规则要验证的源码。
* `options` (array, optional): The options passed to the rule. The rule severity should not be included in this list.
* `options` (array, optional)：这个选项要传递给规则。不要包括规则级别。
* `filename` (string, optional): The filename for the given case (useful for rules that make assertions about filenames)
* `filename` (string, optional)：给定的测试用例的文件名 (只在规则对文件名进行断言时才有用)

In addition to the properties above, invalid test cases can also have the following properties:

除了上述的属性外，invalid 测试用例还可以有以下属性：

* `errors` (number or array, required): Asserts some properties of the errors that the rule is expected to produce when run on this code. If this is a number, asserts the number of errors produced. Otherwise, this should be a list of objects, each containing information about a single reported error. The following properties can be used for an error (all are optional):
* `errors` (number or array, required): 断言该规则在运行该处代码时产生的错误的一些属性。如果是个数组，则断言产生的错误数量。否则，应该是个对象列表，每个对象包含单个报告的错误。一个错误可以包含以下属性（都是可选的）：
    * `message` (string/regexp): The message for the error
    * `message` (string/regexp): 产生的错误的消息
    * `type` (string): The type of the reported AST node
    * `type` (string): 报告的 AST 节点的类型
    * `line` (number): The 1-based line number of the reported location
    * `line` (number): 报告的位置的行号（从1开始）
    * `column` (number): The 0-based column number of the reported location
    * `column` (number): 报告的位置的列号（从0开始）
    * `endLine` (number): The 1-based line number of the end of the reported location
    * `endLine` (number): 报告的位置的结束行号（从1开始）
    * `endColumn` (number): The 0-based column number of the end of the reported location
    * `endColumn` (number): 报告的位置的结束列号（从0开始）
* `output` (string, optional): Asserts the output that will be produced when using this rule for a single pass of autofixing (e.g. with the `--fix` command line flag). If this is `null`, asserts that none of the reported problems suggest autofixes.
* `output` (string, optional): 断言在使用该规则时产生的输出 (如，使用命令行标记 `--fix`)。如果是 `null`，断言没有建议自动修复的报告。

Any additional properties of a test case will be passed directly to the linter as config options. For example, a test case can have a `parserOptions` property to configure parser behavior.

测试用例的其他任何属性都将作为配置选项直接传给 linter 。例如，测试用例可以有一个 `parserOptions` 属性来配置解析器的行为。

#### Customizing RuleTester

To create tests for each valid and invalid case, `RuleTester` internally uses `describe` and `it` methods from the Mocha test framework when it is available. If you use another test framework, you can override `RuleTester.describe` and `RuleTester.it` to make `RuleTester` compatible with it and have proper individual tests and feedback.

为了给每个有效或无效的用例创建测试用例，`RuleTester` 在内部使用 Mocha 测试框架的 `describe` 和 `it` 方法。如果你使用了另一个测试框架，你可以覆盖 `RuleTester.describe` 和 `RuleTester.it` 使 `RuleTester` 与它兼容，并有适当的测试和反馈。

Example:

示例：

```js
"use strict";

var RuleTester = require("eslint").RuleTester;
var test = require("my-test-runner");

RuleTester.describe = function(text, method) {
    RuleTester.it.title = text;
    return method.apply(this);
};

RuleTester.it = function(text, method) {
    test(RuleTester.it.title + ": " + text, method);
};

// then use RuleTester as documented
```


## Share Plugins

In order to make your plugin available to the community you have to publish it on npm.

为了让你的插件在社区中可用，你需要将它发布到 npm。

Recommended keywords:

推荐的关键字：

* `eslint`
* `eslint`
* `eslintplugin`
* `eslintplugin`

Add these keywords into your `package.json` file to make it easy for others to find.

将这些关键字添加到你的 `package.json` 中，会让其他人更容易找到它。

## Further Reading

* [npm Developer Guide](https://docs.npmjs.com/misc/developers)

### Working with Custom Parsers

If you want to use your own parser and provide additional capabilities for your rules, you can specify your own custom parser. By default, the ESLint parser will use its parse method that takes in the source code as a first parameter and additional optional parameters as a second parameter to create an AST. You can specify a `parse` configuration to use your own custom parser. If a `parseForESLint` method is exposed, this method will be used to parse. Otherwise, the parser will use the `parse` method. `parseForESLint` behaves like `parse` and takes in the the source code and optional ESLint configurations. When `parseForESLint` is called, the method should return an object that contains the required property `ast` and an optional `services` property. `ast` should contain the AST. The `services` property contains the parser-dependent services. The value of the service property is available to rules as `context.parserServices`

如果你想使用你自己的解析器，并为你的规则提供额外的功能，那么你可以指定你自定义的解析器了。默认情况下，ESLint 解析器使用它的源码中的解析方法作为第一个参数，其他的可选参数作为第二个参数来创建 AST。你可以指定一个 `parse` 配置来使用你的自定义解析器。如果暴露了 `parseForESLint` 方法，该方法将被用来做解析使用。否则，该解析器将使用 `parse` 方法。`parseForESLint` 类似于 `parse`，也接受源码和可选的 ESLint 配置。当调用 `parseForESLint` 时，该方法应该返回一个包含必须属性的 `ast` 和可选属性 `services` 的对象。`ast` 应该包含对应的 AST。`services` 属性应该包含解析器依赖的服务。规则对于的 service 属性的值为 `context.parserServices`。

If no parseForESLint function is found, the parser will use the default parse method with the source code and the parser options. You can find a ESLint parser project [here](https://github.com/eslint/typescript-eslint-parser).

如果没有发现 `parseForESLint` 方法，解析器将使用带有源码和解析选项的默认的解析方法。具体查看 [ESLint 解析器](https://github.com/eslint/typescript-eslint-parser)。

    {

        "parser": './path/to/awesome-custom-parser.js'
    }

```javascript
var espree = require("espree");
// awesome-custom-parser.js
exports.parseForESLint = function(code, options) {
    return {
        ast: espree.parse(code, options),
        services: {
            foo: function() {
                console.log("foo");
            }
        }
    };
};

```


