# almostjs
__ALMOsT__ is an **A**gi**L**e **MO**del **T**ransformation framework for JavaScript

[![NPM Version][npm-image]][npm-url]
[![Build][travis-image]][travis-url]
[![Build][appveyor-image]][appveyor-url]
[![Test Coverage][coveralls-image]][coveralls-url]
[![MIT licensed][license-image]][license-url]

ALMOsT tries to reduce the complexity involved in MDD (Model Driven Development) and in particular Model Transformations.
With ALMOsT it is possible to start writing new transformations by understanding just few simple concepts.

## Design Principles

ALMOsT is build uped these design principles:
- __No installation__
  It is possible to use the framework instantly, with no installations.
- __No new language__
  It is possible to start using the framework by just knowing _JavaScript_.
- __Fast start-up__
  It is possible to create a minimum viable model editor and model transformation in a very short time.
- __Parallel development__
  It is possible to work in team on different aspects of the same sprint, e.g., by adding a new concept to the model editor, and the corresponding generative rules to the model to model transformation and model to text transformation in parallel.
- __Customized generation__
  While ALMOsT provides a predefined set of output "formats", it is possible to customize the generated output following the necessity of the developer.

## Connected Projects

ALMOsT is developed in modules. The two underlying components are:

[![NPM Version][core-npm-image]][core-npm-url] [__almost-core__](https://github.com/B3rn475/almostjs-core)

[![NPM Version][extend-npm-image]][extend-npm-url] [__almost-extend__](https://github.com/B3rn475/almostjs-extend)

ALMOsT also gives you a set of plugins which can be installed separately:

[![NPM Version][joint-npm-image]][joint-npm-url] [__almost-joint__](https://github.com/B3rn475/almostjs-joint)
[![NPM Version][trace-npm-image]][trace-npm-url] [__almost-trace__](https://www.npmjs.com/package/almost-trace)

## Main Concepts

Here are the few concepts needed to write a model transformation using ALMOst.

### Model

The input, an many times the output, of a model transformation is a model.

```JSON
{
  "elements": [
    ...
  ],
  "relations": [
    ...
  ]
}
```

The only requirements set by ALMOsT are:
- A valid __Model__ is a JSON object
- A valid __Model__ has a field called `elements`, which is an __Array__.
  Anything item of this array will be considered as an __Element__ in the model.
- A valid __Model__ has a field called `relations`, which is an __Array__.
  Anything item of this array will be considered as a __Relationship__ between elements in the model.

No other requirements are mandated in order to start using ALMOsT.

### Transformation Rules

Rules are the building block of a transformation.

```JavaScript
createRule(
  /* Activation Function */
	function (model) { /* Condition */ },
  /* Action Function */
	function (model) { /* Action */ }
)
```

They are composed of two functions:
- The __Activation Function__ which decides if the rules is going to be enabled on the specific input.
- The __Action Function__ which, given the current input, generates a partial output.

There are 3 types of __Rules__:
- __Model Rules__ which are invokes once on the entire model
  ```JavaScript
  createRule(
    function (model) { /* Condition */ },
    function (model) { /* Action */ }
  )
  ```
- __Element Rules__ which are invokes once on each element in the model (i.e. the items of the `elements` array)
  ```JavaScript
  createRule(
    function (element, model) { /* Condition */ },
    function (element, model) { /* Action */ }
  )
  ```
- __Relation Rules__ which are invokes once on each relationship in the model (i.e. the items of the `relations` array)
  ```JavaScript
  createRule(
    function (relation, model) { /* Condition */ },
    function (relation, model) { /* Action */ }
  )
  ```

N.B. there is no guarantees on the execution order of transformation rules.

### Reducer

All the partial results, generated by the transformation rules, are merged togheter by means of a reducer.

ALMOsT Provides 3 default reduction policies:
- __Model To Model__ `m2m`, in this case the rules output is going to be merged as if it was a partial model (i.e. it contains two fields named `elements` and `relations` which content is going to be concatenated)
- __Model To Text__ `m2t`, in this case the rules output is going to be merged as if it was a partial "file system".
  Each partial result is an `Object` in which each field is an `Object` itself describing a __folder__ or a __file__:
  - __Files__ are `Object`s having 2 fields:
    - `name` a `String` which contains the name of the file
    - `content` a `String` which contains the name of the file
    Only one rules should generate a specific file
  - __Folders__ are `Object`s having 3 fields (:
    - `name` a `String` which contains the name of the folder
    - `isFolder` always set to `true`
    - `children` an `Array` containing `String`s, each one referencing a the key of another filed in the to "file system like" `Object`
    Multiple rules can generate the same folder, but only one rule should set `name` and `isFolder`, the elements of `children` will be concatenated.
- __Model To ALMOsT__ `m2a` the same as `m2m`, but Elements and Relations should follow the extra requirements defined in [A bit or restrictions for extra power](#a-bit-or-restrictions-for-extra-power).

[__almost-core__](https://github.com/B3rn475/almostjs-core)

### A full transformation

```JavaScript
var createTransformer = require('almost').createTransformer,
    createRule = require('almost').createRule;

var transform = createTransformer({
    model: [
      createRule(
        function (model) { /* Condition */ },
        function (model) { /* Action */ }
      )
    ],
    element: [
      createRule(
        function (element, model) { /* Condition */ },
        function (element, model) { /* Action */ }
      )
    ],
    relation: [
      createRule(
        function (relation, model) { /* Condition */ },
        function (relation, model) { /* Action */ }
      )
    ]
}, 'm2m');

var inputModel = {
  elements: [
    ...
  ],
  relations: [
    ...
  ],
}

var output = transform(input);
```

### A bit or restrictions for extra power

While there are minimal restrictions imposed on the [Model](#model), in particular the structure of an __Element__ or a __Relationship__, ALMOsT has a suggested structure to follow.

See the documentation of [__almost-extend__](https://github.com/B3rn475/almostjs-extend).

Following this structure not just enables the use of __almost-extend__, but also enables the use of the `m2a` reducer, which enables the developer to generate the same __Element__ in multiple rules.
Elements generated in different rules are matched by `id` and their `attributes`, and `matadata`, are merged using the `mergeOrSingle` policty defined in [__almost-core__](https://github.com/B3rn475/almostjs-core).
Under this policy `Array` attributes are concatenated, while `Object` attributes are reducersively merged with the same policy.

## The better way to learn is by looking at examples

- [ICWE2018-Tutorial](https://github.com/B3rn475/ICWE2018-Tutorial/tree/master/client/js) contains a simple example written during a Tutorial at [ICWE 2018](https://icwe2018.webengineering.org/).
- [IFMLEdit.org](https://github.com/B3rn475/IFMLEdit.org/tree/master/client/js) contains multiple examples of transformations.

## Related Publications

- [ALMOsT.js: An Agile Model to Model and Model to Text Transformation Framework](https://link.springer.com/chapter/10.1007/978-3-319-60131-1_5)
- [Tools, semantics and work-flows for web and mobile model driven development](https://www.politesi.polimi.it/handle/10589/144846)

[npm-image]: https://img.shields.io/npm/v/almost.svg
[npm-url]: https://npmjs.org/package/almost
[travis-image]: https://img.shields.io/travis/B3rn475/almostjs/master.svg
[travis-url]: https://travis-ci.org/B3rn475/almostjs
[appveyor-image]: https://ci.appveyor.com/api/projects/status/github/B3rn475/almostjs?svg=true
[appveyor-url]: https://ci.appveyor.com/project/B3rn475/almostjs
[coveralls-image]: https://img.shields.io/coveralls/B3rn475/almostjs/master.svg
[coveralls-url]: https://coveralls.io/r/B3rn475/almostjs?branch=master
[core-npm-image]: https://img.shields.io/npm/v/almost-core.svg
[core-npm-url]: https://npmjs.org/package/almost-core
[extend-npm-image]: https://img.shields.io/npm/v/almost-extend.svg
[extend-npm-url]: https://npmjs.org/package/almost-extend
[joint-npm-image]: https://img.shields.io/npm/v/almost-joint.svg
[joint-npm-url]: https://npmjs.org/package/almost-joint
[trace-npm-image]: https://img.shields.io/npm/v/almost-trace.svg
[trace-npm-url]: https://npmjs.com/package/almost-trace
[license-image]: https://img.shields.io/badge/license-MIT-blue.svg
[license-url]: https://raw.githubusercontent.com/B3rn475/almostjs/master/LICENSE
