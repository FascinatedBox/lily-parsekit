Parsekit engine schema
======================

The core of parsekit is an engine that outputs an enum of the following:

```
enum JsObject {
    JsBoolean(Boolean)
    JsInteger(Integer)
    JsString(String)
    JsList(List[JsObject])
    JsHash(Hash[String, JsObject])
}
```

You'll `import engine` to get this lightweight Json-like enum. Afterward, you
can use the `as_*` functions to get the contents of values. The `as_*` functions
will return either that value, or a false-equivalent default.

The output of the block parser is a json blob that follows the schema defined
below.

## Notes

* This schema carries the implicit assumption that the outgoing representation
  is for a single file. The schema may change to include scopes in references
  when that is no longer the case.

* The schema makes reference to class methods being static, a feature not yet
  implemented in the interpreter.

## Directives

### Type

```
{
    "class": "<name>",
    "children": [ type member... ] | absent
    "value": "<default value>" | absent
    "is_optarg": true | absent
    "is_vararg": true | absent
}
```

Note that in the case of functions, the return type is always present since
a `Function` will always return at least `Unit`. Similar to Lily's internals,
the return type of a function is the first value, instead of the last.

In the case of optional arguments, `"is_optarg"` is always present but
`"value"` may not be (ex: `define x(fn: Function(*Integer))`).

## Var

```
{
    "name": "<name>"
    "qualifier": "protected" | "private" | "public" | absent
    "type": type directive
}
```

Global vars will never have a qualifier set, as they're always global.

## Function

```
{
    "name": "<name>"

    "args": [
        {
            "name": "<argument name>",
            "type": type directive
        } ...
    ]

    "doc": "<docstring>"

    "generics": [
        "<generic name>"...
    ] non-empty if present

    "is_ctor": true | absent
    "is_static": true | absent
    "output": type directive
    "qualifier": "protected" | "private" | "public" | absent
}
```

`generics` lists every generic that is available to this function. Since Lily
expects generics to follow a rigid order (`"A, B, C"`), this ends up being a
list of letters.

If the function is a class member, the default action is to automatically have
the argument `"self"` with a type that `self` would have. This action has not
happened if `"is_static"` is absent.

If the function is a class constructor, then the return type is the type that
`self` would have within the function. The function also has the special name of
`"<new>"` that Lily uses to prevent constructors from being explicitly called.

The output field is never absent. If the function declaration didn't specify a
return value, then a type directive with class `"Unit"` and no children is used.

Similar to the var directive, `qualifer` is only absent of the function provided
is global.

## Class

```
{
    "name": "<name>"

    "doc": "<docstring>"

    "fields": [
        "LILY_FOREIGN_HEADER",
        "<layout member>"...
    ] | absent

    "functions": [
        function directive...
    ] | absent

    "generics": [
        "<generic name>"
    ] non-empty if present

    "kind": "builtin" | "class" | "native"
    "parent": "<parent name>" | absent

    "properties": [
        var directive...
    ] | absent
}
```

There are three general kinds of classes that are represented here:

* native classes will always `"members"` present, even if it is empty. This is
  the only kind of class that can inherit from another.

* foreign classes that need a wrapper struct will always have `"fields"` set.
  `"LILY_FOREIGN_HEADER"` is always present and always the first member in the
  list. Layout members should each be a proper struct field member with a `';'`
  at the end.

* builtin classes (made by the builtin package) don't have members or fields
  set.

## Variant

```
{
    "name": "<name>"

    "args": [
        {
            "name": ""
            "type": type directive
        }...
    ] | absent
}
```

Right now, `"name"` is always an empty string, because the interpreter does not
yet allow variant arguments to have names.

## Enums

```
{
    "name": "<name>"

    "doc": "<docstring>"
    "is_scoped": true | false

    "functions": [
        function directive...
    ]

    "generics": [
        "<generic name>"
    ] non-empty if present

    "kind": "enum"

    "variants": [
        variant directive...
    ]
}
```

The `"is_scoped"` field maps to whether this enum was made by `scoped enum` or
simply `enum`.

Variants is always present (and should always be occupied by an enum with at
least two variants to select between).

## Module

```
{
    "name": "<name>"

    "containers": [
        (class directive |
         enum directive)...
    ] | absent

    "doc": "<docstring>"

    "functions": [
        function directive...
    ] | absent

    "kind": "module"

    "vars": [
        var directive...
    ] | absent

    "modules": [
        module directive...
    ] | absent
}
```

This is the root of the tree, returned once parsing is done.
