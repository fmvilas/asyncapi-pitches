# React as a Generator engine

Make Generator support React as a template engine, along with Nunjucks.

## Problem

The current templating system —based on Nunjucks— has some really important drawbacks:

1. It's hard (or impossible) to create reusable "partials" across templates. Ultimately, this leads to repetition, especially those of the same kind. For instance, POJO (Plain Old Java Objects) definitions are common across different templates. And this is something that's going to happen more and more as we create different templates for the same programming language.
1. It gets really painful to debug a template when an error happens inside a macro. Even with the `--debug` flag turned on. That's a problem of Nunjucks itself and has been there for quite some time now.
1. Even though Nunjucks provides lots of built-in filters, it's very common to find ourselves having to create a custom filter for basic stuff like checking the type of a variable, e.g., `typeof variable === 'object'` or running basic JS logic.
1. It can get really hard to read a complex template. See [this example](https://github.com/asyncapi/java-spring-cloud-stream-template/blob/master/partials/java-class).
1. All the points above become super important if we want to increase the number of templates. Template authoring needs to be a smooth process, not a painful one.

## Solution

In general, the solution goes through adding React as a template engine to the Generator. When the template declares that it's using React as its template engine, the Generator will render files using React instead of Nunjucks.

There a slight change in the "rendering paradigm" though. In the case of React, we don't "render the file" but instead we "import" it, "process" it, and get the result as a string that has to be written to the destination file. This will allow us to have top-level metadata on each file, like in the following example:

```js
// File: $$schema$$.java

...
import { generateCamelCaseSchemaName } from '@asyncapi/generator/sdk'

export default function SchemaFile({ schema }) {
  return (
    <File name={generateCamelCaseSchemaName(schema)}>
      ... (your Java code here)
    </File>
  )
}
```

In the example above, we have a top-level `<File>` component that allows us to add meta information about the file itself, like its name, permissions, and more. Even more, we can tell Generator not to render this file by simply returning `null` in the `SchemaFile` function:

```js
// File: $$schema$$.java

...
import { generateCamelCaseSchemaName } from '@asyncapi/generator/sdk'

export default function SchemaFile({ schema }) {
  if (schema.type !== 'object') return null // <== NOTICE THIS LINE
  
  return (
    <File name={generateCamelCaseSchemaName(schema)}>
      ... (your Java code here)
    </File>
  )
}
```

In the example above, this file will only be rendered if the type of the schema is `object`. This will allow us to eventually get rid of file name template syntax like `$$schema$$`, `$$everySchema$$`, and `$$objectSchema$$`. Also, the usage of `<File name="...">` will eventually allow us to get rid of all the file name templates.

## Solution elements

* Renderer: the code in charge of rendering a React component to a file. It will be used from the Generator.
* `<File>` component: the component in charge of defining a file's metadata and content. As metadata, we should have `name` and leave `permissions` as a nice to have.
* `<Text>` component: this would be used for rendering plain text, e.g., code. Should offer props to set indentation and its type. For instance, `<Text indentation={1} indentationType="tabs">` to indent the whole text with 1 tab.

## Rabbit holes & challenges

1. Compiling React JSX and ES7 `import` syntax may be challenging. Watch out. We may have to introduce a step to precompile ES7 and JSX syntax before the renderer `require`s the files.
1. We may have to create our own React renderer with its own DOM model and reconciler. Here's a great source of inspiration: https://github.com/chentsulin/awesome-react-renderer.

## Out of bounds

1. Removing the usage of file name templates.
1. Creating an SDK for the Generator.