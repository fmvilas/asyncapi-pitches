<h2>On This Page</h2>

<!-- TOC depthFrom:2 depthTo:6 updateOnSave:true -->

- [Summary](#summary)
  - [Problem Overview](#problem-overview)
  - [Solution Overview](#solution-overview)
  - [Positive Side-Effects](#positive-side-effects)
- [Problem Details](#problem-details)
  - [Generating strong-typed data models is a rather complex task](#generating-strong-typed-data-models-is-a-rather-complex-task)
  - [Complex and unmaintainable template files](#complex-and-unmaintainable-template-files)
  - [Creating and maintaining data model generators for many languages is an arduous task](#creating-and-maintaining-data-model-generators-for-many-languages-is-an-arduous-task)
- [Solution guidelines](#solution-guidelines)
  - [My proposed "solution"](#my-proposed-solution)
  - [These examples are suggestions](#these-examples-are-suggestions)
- [Boundaries](#boundaries)
    - [Don’t do](#dont-do)
    - [Watch out for](#watch-out-for)
- [Long-Term Vision](#long-term-vision)

<!-- /TOC -->

## Summary

### Problem Overview

1. Generating strong-typed data models is a rather complex task.
1. Complex and unmaintainable template files.
1. Creating and maintaining data model generators for many languages is an arduous task.

### Solution Overview

1. Design an SDK to generate strong-typed data models from AsyncAPI / JSON Schema.
1. The SDK must be suitable for Nunjucks as well as for React.
1. The SDK must allow for the extensibility of existing data model definitions.

### Positive Side-Effects

1. Less code fragmentation and duplication across templates. 

## Problem Details

### Generating strong-typed data models is a rather complex task

As of today, generating data models is a purely manual task that every template developer should take care of. There are lots of things to take into account when creating a generic solution even when it's for a specific programming language. To name a few:

* Map data types from JSON Schema to the selected programming language.
* Make sure variable names are valid and conforming to the standards of the language.
* Branching logic, i.e., creating some lines of code depending on the constraints of a property. For instance, validating that a number is between the specified minimum and maximum values before it's assigned on a setter method.

### Complex and unmaintainable template files

The complexity described above also creates some bad side-effects, like the [difficulty to read and maintain a data model template](https://github.com/asyncapi/java-spring-cloud-stream-template/blob/master/partials/java-class). Using React may improve the readability but that will leave out the developers who prefer Nunjucks.

### Creating and maintaining data model generators for many languages is an arduous task

If creating and maintaining a data model generator for a single programming language is hard, imagine doing the same for many programming languages. And keeping them updated and in-sync in terms of features is another challenge. E.g., making sure all of them support annotations, different visibility levels, etc.

## Solution guidelines

Since we want to solve these problems for Nunjucks and React templates, the solution goes through an SDK that both frameworks would use. It may be used behind the scenes by providing filters in Nunjucks and components in React. In any case, there must be a framework-agnostic solution.

**For now, the solution is to invest some time in thinking and designing that SDK.** Let me outline a few things I found out during my research:

* It must be framework-agnostic. We'll use it ourselves but, in practice, anyone should be able to use it in their tools.
* It is not enough to generate code. We need to somehow understand how this code is composed, e.g., where `import` or `require` statements start and end, what's a method definition, what's a class attribute, how annotations work, and more.
* We should be able to "alter" an existing template. E.g., add custom annotations, custom methods, etc.

### My proposed "solution"

Let's start with an example. Assuming we have a schema like the following:

```js
{
  type: 'object',
  properties: {
    displayName: {
      type: 'string'
    },
    email: {
      type: 'string',
      format: 'email'
    },
    createdAt: {
      type: 'string',
      format: 'date-time'
    }
  }
}
```

...and this code using the SDK:

```js
const { generateModelFor, Constants, buildMethod, buildGetters, buildSetters, typeNameFor, varNameFor, annotate } = require('@asyncapi/generator-sdk')
const schema = new Schema(UserSignedUp)

return generateModelFor('java', schema, {
  package: 'com.asyncapi',
  dependencies: ['com.fasterxml.jackson.annotation.JsonFormat'],
  annotations: {
    properties: (property, propertyName) => {
      if (property.format() !== 'date-time') return property

      annotate(property, 'JsonFormat', [{
        shape: 'JsonFormat.Shape.STRING',
        pattern: '"dd-MM-yyyy hh:mm:ss"',
      }])
    },
    methods: (method) => {
      const { name, visibility, isStatic, returnType, arguments } = method
      annotate(method, 'SomeAnnotation', [{
        someAnnotationKey: '"someAnnotationValue"',
      }])
    }
  },
  methods: {
    hashCode: null, // Removes hashCode method from the generated result
    findEvenOdd: buildMethod({
      name: 'findEvenOdd',
      visibility: Constants.Visibility.PUBLIC,
      static: true,
      returnType: 'String',
      parameters: [{
        name: 'num',
        type: 'int',
        // default: 0,
      }],
      body: '// method implementation here...'
    }),
    // getters
    ...buildGetters(schema),
    // setters
    ...buildSetters(schema),
  },
})
```

The following should be generated:

```java
package com.asyncapi;

import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.annotation.JsonProperty;
import java.util.Date;
import java.text.DateFormat;

@JsonInclude(JsonInclude.Include.NON_NULL)
public class UserSignedUp {

  @JsonProperty("displayName")
  private String displayName;
  
  @JsonProperty("email")
  private String email;

  @JsonProperty("createdAt")
  @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "dd-MM-yyyy hh:mm:ss")
  private Date createdAt;

  constructor (String displayName, String email, Date createdAt) {
    this.displayName = displayName;
    this.email = email;
    this.createdAt = createdAt;
  }

  @SomeAnnotation(SomeAnnotationKey = "SomeAnnotationValue")
  public String getDisplayName () {
    return displayName;
  }
  
  @SomeAnnotation(SomeAnnotationKey = "SomeAnnotationValue")
  public String getEmail () {
    return email;
  }
  
  @SomeAnnotation(SomeAnnotationKey = "SomeAnnotationValue")
  public Date getCreatedAt () {
    return createdAt;
  }

  @SomeAnnotation(SomeAnnotationKey = "SomeAnnotationValue")
  public void setDisplayName (String newDisplayName) {
    displayName = newDisplayName;
  }
  
  @SomeAnnotation(SomeAnnotationKey = "SomeAnnotationValue")
  public void setEmail (String newEmail) {
    email = newEmail;
  }
  
  @SomeAnnotation(SomeAnnotationKey = "SomeAnnotationValue")
  public void setCreatedAt (String newCreatedAt) {
    return DateFormat.parse(newCreatedAt);
  }

  @SomeAnnotation(SomeAnnotationKey = "SomeAnnotationValue")
  public static void findEvenOdd (int num) {
    // method implementation here...  
  }
}
```

Afterward, this SDK can power Nunjucks filters and React components. See the following React example:

```jsx
import { Model, Constants } from 'generator-react-sdk'
const schema = new Schema(UserSignedUp)

return (
  <Model
    language="java"
    schema={schema}
    annotateProperties={prop => {
      if (prop.format() !== 'date-time') return prop

      return annotate(prop, 'JsonFormat', [{
        shape: 'JsonFormat.Shape.STRING',
        pattern: 'dd-MM-yyyy hh:mm:ss',
      }])
    }}
    annotateMethods={method => {
      const { name, visibility, isStatic, returnType, arguments } = method
      annotate(method, 'SomeAnnotation', [{
        someAnnotationKey: '"someAnnotationValue"',
      }])
    }}
    methods={{
      hashCode: null, // Removes hashCode method from the generated result
      findEvenOdd: {
        visibility: Constants.Visibility.PUBLIC,
        static: true,
        returnType: 'String',
        parameters: [{
          name: 'num',
          type: 'int',
          // default: 0,
        }],
        body: '// method implementation here...'
      }
    }}
    buildGetters={true}
    buildSetters={true}
  />
)
```

### These examples are suggestions

This pitch/bet is actually about ideating and designing the SDK (excluding the React one). Let's make sure it works with any kind of programming language (object-oriented, functional, etc.)

A good starting list of languages to test against are:

* Java
* JavaScript / Node.js
* Python
* C#
* [GraphQL data type](https://graphql.org/learn/schema/)
* C++
* [Haskell](http://learnyouahaskell.com/making-our-own-types-and-typeclasses)

And a good starting list of artifacts that programming languages may offer:

* Packages
* Imports
* Interfaces
* Classes
  * Inheritance (may be possible to inherit from multiple classes)
* Attributes
* Methods
  * Getters
  * Setters
* Annotations
* Decorators
* Visibility
  * Public
  * Private
  * Protected
  * There may be others...
* Basic Types
* Functions
* Parameters (functions, methods, etc.)


## Boundaries

#### Don’t do

* **Please don't write a single line of code for the SDK yet.** It's ok to have proofs of concepts in your machine if that helps you. Remember, the goal is to get a basement from where we'll build a great SDK in the future.
* Don't go too far trying to define all the possibilities a programming language can offer. Let's stick to the API surface. For instance, we're not interested in defining the inners of the body of a function. A simple string would suffice.

#### Watch out for

* Java Annotations and Python Decorators are not the same: https://stackoverflow.com/questions/15347136/is-a-python-decorator-the-same-as-java-annotation-or-java-with-aspects


## Long-Term Vision

In the short term, the idea behind this SDK is to power the Generator templates, making it super easy to create new ones. However, long-term applications are also interesting. A declarative way of generating code will also help us with:

* **SDK generation.** We may end up providing an AsyncAPI-first SDK (not to confuse with the Generator SDK) for multiple programming languages.
* **Parser generation.** Since we're able to generate code in any language, it would be easier for us to automatically generate some parts of the AsyncAPI parsers that we'll write for other languages.

That said, focus on the short-term goals for now. We don't want to end up creating a super generic solution.

Happy ~coding~ designing! :blush: