---
title: TypeScript
description: integereleven's approach to TypeScript
toc_min_heading_level: 2
toc_max_heading_level: 4
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CodeBlock from '@theme/CodeBlock';

## Introduction {#intro}

### Terminology {#terms}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL
NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in
[RFC 2119][rfc2119].

## Source file overview

### Encoding

Source files must be encoded in UTF-8.

### Whitespace

With the exception of the line terminator sequence, the ASCII *SPACE* character (`_`, `0x20`) must be the only whitespace character in a source file.

### Escape sequences

The special escape sequence for any character that has one (`\'`, `\"`, `\\`, `\b`, `\f`, `\n`, `\r`, `\t`, `\v`) must be used rather than the corresponding numeric escape (e.g. `\x0a`, `\u000a`, `\u{a}`). Legacy octal escapes (e.g. `\085`, `\012`) must not be used.

### Non-ASCII characters

The remaining non-ASCII characters must use the actual Unicode character (e.g. `∞`). The equivalent hex or Unicode escapes for non-printable characters may be used as long as there is a comment explaining the character and its use.

:::danger Bad non-ASCII usage

This example highlights the issues with escaping printable characters. `units` is unclear with even with a comment and `output` would be unreadable to most developers.

```ts
const units = '\u03bcs';  //  Greek mu, appending 's'

const output = `\ufeff${content}`;
```

:::

:::tip Good non-ASCII usage

This example shows small changes can remove unnecessary vagueness.

```ts
const units = 'μs';

const output = `\ufeff${content}`; // byte order mark
```

For cases, where you re-use non-printable characters within the same file, you should name the interpolated value descriptive of the non-printable character.

```ts
const byteOrderMark = '\ufeff';

const output = `${byteOrderMark}${content}`;
```

You may abbreviate the name, as long as you provide an explanatory comment.

```ts
const bom = '\ufeff'; // byte order mark

const output = `${bom}${content}`;
```

:::

### Structure

Source files **may** consist of the following, **in order**:

1. JSDoc header with the following:
  1. JSDoc `@copyright` tag with copyright information, **required**.
  2. Either of the following, required:
    1. JSDoc `@file` tag describing the file for source files.
    2. A module description followed by the JSDoc `@module` tag for module entry file.
2. Imports, if present.
3. The file's implementation.

A single blank new line **must** separate each present section.

### JSDoc header

#### Copyright information

Copyright information must include the following:

- The copyright start year.
- The current year.
- The copyright holder.
- The phrase "All rights reserved".
- The license the source file is released under.

```ts title="Typical copyright tag in the JSDoc header of a source file"
/**
//highlight-start
 * @copyright 2020-2024 integereleven. All rights reserved. MIT license.
//highlight-end
 * ...
 */
```


#### Source file descriptor

The descriptor for a source file should include a description of what the exported symbols are and their purpose.

```ts title="Example of a source file descriptor"
/**
 * ...
//highlight-start
 * @file Exports the Foo, Bar, and Baz functions, which do nothing but provide example functionality.
//highlight-end
 */
```

```ts title="An example of a directory entry-file descriptor"
/**
 * ...
//highlight-start
 * @file Re-exports the publicly available types.
//highlight-end
 */
```
:::note Directory entry files
Directory entry files are not module entry files, but generally have the same name `mod.ts`. Most directories have them and they act as a roll-up of the public API within the directory to simplify development and keep import statements clean.
:::

#### Module-entry file descriptor

```ts title="An example of a module-entry file descriptor"
/**
 * ...
//highlight-start
 * This module provides the testing functions, {@link foo}, {@link bar},
 * and {@link baz}.
 *
 * These functions are dumb and do absolutely nothing, but are there for testing.
 *
 * You can use the {@link foo} function to do foo-things.
 * 
 * ```ts
 * import { assertEquals } from '@std/assert';
 * import { foo } from './mod.ts';
 * 
 * const a = foo();
 *
 * assertEquals(a, 'foo', 'a is not foo');
 * ```
 *
 * @module
//highlight-end
 */
```

### Imports

In order:

#### Variants

In order:

##### Module

module import

TypeScript imports

```ts
import * as foo from './foo.ts';
```

##### Named

destructured import

TypeScript imports

```ts
import { Foo } from './foo.ts';
```

Within named imports, import types, in order:

###### Regular

Imports only value symbols.

```ts
import { Foo, FooEnum, FOO_CONST } from './foo.ts';

const foo = new Foo(FooEnum.Foo);

foo.foo === FOO_CONST;
```

###### Mixed

Imports both value and type symbols.

```ts
import { Foo, FooEnum, FOO_CONST, type IFoo, type FooType } from './foo.ts';

const foo = new Foo(FooEnum.Foo);
const fint: IFoo = {
  foo: 'foo',
};
const f: FooType<IFoo> = { foo: fint };

foo.foo === FOO_CONST;
```

###### Type

Imports only type symbols.

```ts
import type { FooEnum, IFoo, FooType } from './foo.ts';

export function getFoo(variant: FooEnum): FooType<IFoo> {
  const foo = variant === 0 ? 'bar' : 'foo';
	
  return {
    foo: { foo },	
  };
}

getFoo(1);
```

##### Default

Only used to import default exports from external, 3rd party code when required.
integereleven does not use default exports and does not use 3rd party code outside the standard libraries, so this usage should be very rare.

```ts
import foo from './foo.ts';
```

##### Side-effect

Only used to import modules for their side-effect when loaded.

```ts
import './foo.ts';
```

Each variant section should be separated by a single empty new line.

```ts
import * as foo from './foo.ts';

import { Bar } from './foo.ts';

import Baz from './foo.ts';

import './foo.ts';
```

#### Locations

With in import variants, locations, in order:

##### Parent files

Parent files, furthest away, then alphabetically by path name.

```ts
import { baz } from '../../baz/mod.ts';
import { bar } from '../../bar.ts';
import { quux } from '../internal/quux.ts';
import { foo } from '../foo.ts'
```

`baz` import is 4 steps away. `bar` and `quux` are both 3 steps away. The "b" in "bar" comes before the "i" in "internal". `foo` is 2 steps away.

##### Child files

Child files, furthest away, then alphabetically by path name.

```ts
import { baz } from './util/baz/mod.ts';
import { quux } from './internal/quux.ts';
import { bar } from './util/bar.ts';
```

`baz` import is 4 steps away. `quux` and `bar` are both 3 steps away. The "i" in "internal" comes before the "u" in "util".

##### Sibling files

Sibling files, alphabetically by path name.

```ts
import { bar } from './bar.ts';
import { baz } from './util.ts';
import { quux } from './quux.ts';
```

All siblings are 2 steps away. The "b" in "bar" comes before the "u" in "util", which comes before the "q" in "quux".

#### Import paths
Paths may be relative, beginning with `.` or `..`, or absolute, e.g. `path/to/file.ts`.

Relative paths are relative to the directory of the file referencing the relative path. Absolute paths are relative to the current working directory or base directory.

It is preferred to use relative paths when referring to files within the same project.

It is recommended to keep the number of parent steps to three steps (`../../../`).

##### Sub-module imports

Some modules will contain sub-modules, and in some cases, a sub-module may need a symbol from another sub-module. As they are within the same project, it is tempting to navigate to the file containing the original symbol export. 

Importing a symbol across sub-module boundaries must be done at the sub-module entry file.

```
//higlight-start
sub_mod_a/
|__ mod.ts (module entry file)
//higlight-end
|__ src/
| 	|__ mod.ts (directory entry file)
|   |__ types/
| 		  |__ types.ts
| 		  |__ mod.ts (directory entry file)
//higlight-start
sub_mod_b/
|__ mod.ts (module entry file)
//higlight-end
|__ src/
		|__ mod.ts (directory entry file)
		|__ my_class.ts
	  |__ types/
			  |__ types.ts
			  |__ mod.ts (directory entry file)
```

In the example project above, `sub_mod_a` and `sub_mod_b`, being within the same project, are capable of accessing anything within each other’s source files. However, sub-modules within a project must treat each other as foreign codebases only permitted to access each other’s module entry file.

#### Module imports

Both module (namespace) imports and named imports may be used.

Named imports are preferred for symbols used frequently in a file, or for symbols with concise names.

You may alias named imports using `as` in these scenarios. 

- The exported symbol name is not concise and aliasing it will improve clarity.
- The exported symbol has a generated or exceedingly long name.
- To avoid name collisions with other exported symbol names.

```ts
import { foo } from './my/foo.ts';
import { foo as otherFoo } from './their/foo.ts';
import { a as bar} from './bar.ts';
import { NAHD6743HGFDHJfgs as Random } from './generated.ts';
import { doesAThingAndThenAnotherThingToReturn42RegardlessOfInput as get42 } from './what.ts';
```

Module imports are preferred when using many different symbols from large APIs. Despite using the `*` character, module imports are not "wildcard" imports comparable in other languages. Module imports provide a name to all exports of a module, with each exported symbol from that module becoming a property on the module name. This may aid in code clarity for symbols with common names, without having to declare aliases. 

:::warning Okay, but not great

Though this example isn’t terrible when it comes to clarity, it is a lot of overhead when it comes to cleanliness and using additional features of each API.

```ts
import { Bar as CommandBar, Item as CommandBarItem } from './command_bar.ts';
import { Bar as NavBar, Item as NavBarItem } from './nav_bar.ts';

const cmdItems: CommandBarItem[] = [];
const cmdBar = new CommandBar(cmdItems);

const navItems: NavBarItem[] = [];
const navBar = new NavBar(navItems);
```

:::

:::tip Better and preferred

This example has a bit more clarity that the previous example and is much easier to maintain, as using additional features of each API is as easy as accessing the module symbols (`CommandBar` and `NavBar`).

```ts
import * as CommandBar from './command_bar.ts';
import * as NavBar from './nav_bar.ts';

const cmdItems: CommandBar.Item[] = [];
const cmdBar = new Command.Bar(cmdItems);

const navItems: NavBar.Item[] = [];
const navBar = new Nav.Bar(navItems);
```

:::

:::danger Unnecessary module import

Importing module name does not improve code clarity.

```ts
import testing from '@std/testing/bdd';
import assert from '@std/assert';

const foo = 'foo';

testing.describe('foo', () => {	
	testing.it('should bar', () => {
		assert.assert(true);
		assert.assertEquals(foo, 'foo');
	});
});
```

:::

:::tip Better and preferred

Exported symbol names are concise and common.

```ts
import { describe, it } from '@std/testing/bdd';
import { assert, assertEquals } from '@std/assert';

const foo = 'foo';

describe('foo', () => {	
	it('should bar', () => {
		assert(true);
		assertEquals(foo, 'foo');
	});
});
```

:::

### Exports

Only named exports **must** be used.

Default exports do not have a canonical name, increasing maintenance and clarity difficulty while providing little benefit to maintainers. Default export encourage developers to encapsulate exports in a container in an attempt to namespace the exports. This creates ambiguity with the exported value where developers may attempt to use it as a value or type.

:::danger No canonical name

The following example highlights the lack of a canonical name for default exports.

```ts
import Foo from './foo';
import Bar from './foo';
```

:::

:::danger Maintenance difficulty

The following examples highlight difficulty in debugging when using default exports.

```ts title="./foo.ts"
const foo = 'bar';
export default foo;
```

Trying to import a named export that doesn’t exist throws an error. The following results in a `TS2614` error. `Module '"./foo.ts"' has no exported member 'foo'.`.

```ts title="./bar.ts"
import { foo } from './foo.ts';
```

```ts title="./baz.ts"
import baz from './foo.ts';
```

`baz` in `baz.ts` is equal to `foo` in foo.ts.

:::

Developers can achieve a namespace simply with the file scope provided by the module itself, which is required in this style guide.

Named exports have the benefit of erroring when importing a symbol that hasn’t been declared.

```ts
export const SOME_CONST = 'some_const';
export function thisWillHelp() { }
export class FooBar {

}
```

#### Visibility

There is no support for restricting visibility for exported symbols in TypeScript, so symbols that are to be used outside the module, and those that developers may come into contact with, must only be exported. It is **preferred** that the surface of public APIs of a module are minimal.

What must be exported:

- Public API symbols
- Any type that meets the following criteria:
  - A parameter of a public API method or function.
  - The return value of a public method or function.
  - The value type of any public property, constant, or variable.
  - For the above listed:
    - The type of any property of a parametric object, or element of a parametric array, regardless of the depth.
    - Any type in any function type of a parametric function.

#### Mutability

Exports must be final and immutable. `export let` is not allowed.

Mutable exports are difficult to support as they can be hard to understand and debug. This is exacerbated when it is re-exported across multiple modules.

If the need to provide external accessibility and mutable binding, it should be done with explicit getter functions.

:::danger Do not use `export let`.

```ts
let index = 0;
export let id = `id-${index}`;

setTimeout(() => id = `id-${++index}`, 1000);
```

:::

:::tip Explicit getter

```ts
let index = 0;
let id = `id-${index}`;

setTimeout(() => id = `id-${++index}`, 1000);

export function getId() {
  return id;
}

```

:::

There is a common pattern of conditionally exporting a value. You must first do the conditional check. All exports must be final.

```ts
function selectApi() {
  const os = Deno.build.os;

  if (os === 'windows') {
    return windowsStuff();
  } else {
    return otherStuff();
  }
}

export const api = selectApi();
```

#### Container classes

Container classes are classes that contain static methods and/or properties, or singleton instance of a class with methods and/or properties, for the sake of creating a namespace.

Container classes must not be used. Instead rely on the file scope and module namespace and export individual constants and functions.

### Import/export type

#### Import type

The import type statement (`import type {...}` or `import { type ...}`) **may** be used to import a symbol only as a type. For values, use regular imports.

Import type helps speed of development by improving compilation time, which produces quick iteration loops. It also helps increase stability and reduce bugs in productions.

```ts
import type { Foo } from './foo.ts';

export function getFooValue(foo: Foo): number {
  return foo.valueOf();
}
```

Because we are not working with the value `Foo` itself here, we only need to import the type.

```ts
import { Foo } from './foo.ts';

export function getFooValue(index: number): number {
  const foo = new Foo(index);

  return foo.valueOf();
}
```

Because we are creating an instance of `Foo` in the module, we need to import the value as well.

#### Export type

The export type statement (`export type`) may be used when re-exporting a type.

This is useful in file-by-file transpilation. 

The export type statement must not be used to avoid exporting a value for an API. Splitting out type and value into separate symbols is less error prone and communicates the intent more clearly.

##### No `namespace`

The TypeScript `namespace` statement must not be used. You may semantically namespace code using files.

Code must use ES6 module syntax and never use the `require` statement.

## Language features

[rfc2119]: https://datatracker.ietf.org/doc/html/rfc2119 "Key words for use in RFCs to Indicate Requirement Levels"
