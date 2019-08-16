
# F# RFC FS-1003 - nameof Operator

The design suggestion [nameof operator](https://github.com/fsharp/fslang-suggestions/issues/252) has been marked "approved in principle".
This RFC covers the detailed proposal for this suggestion.

[Discussion thread](https://github.com/fsharp/FSharpLangDesign/issues/48)

* [x] Approved in principle
* [x] Details: [Resolved to Preview](https://github.com/fsharp/FSharpLangDesign/issues/48)
* [x] Implementation: [Complete to Preview](https://github.com/Microsoft/visualfsharp/pull/6325)

### Introduction

It's often useful to obtain the simple (unqualified) string name of a variable, type, or member. Today users end up writing something like this:

```fsharp
let add x y =
    if x < 0 then raise (ArgumentOutOfRangeException("x"))
    x + y
```

This RFC allows user code to extract the name of a declaration, like:

```fsharp
let add x y =
    if x < 0 then raise (ArgumentOutOfRangeException(nameof(x))
    x + y
```

### Naming 

The name of the operator is `nameof`. It is an intrinsic in FSharp.Core

### Scope of use

- can be used with method parameters

- can be used with local variables

- can be used with local (nested) functions

- can be used with local curried functions

- can be used with local tupled functions

- can be used with provided symbol names, i.e. generated by type providers

- can be used from inside a local function (needs to be let rec)

- can be used with member names

- can be used with static member names

- can be used with static property names

- can be used with names that are quoted in <code>``</code>

- can be used with names of operators like `+`, `|>`, `typeof`, `nameof`, ...

- can be used with generic functions/types

- can be used in attributes

- can be used with namespaces, `nameof(System.Diagnostics)`

- can be used with type names, `nameof(System.String)`

- can be used with module names, `nameof(FSharp.Collections.List)`

- can be used with operators, `nameof(+)`.  In this case, the construct evaluates to the compiled name of the operator.

Other considerations:

- the use of `nameof` is allowed in quotations, the substitution is still made at compile-time

- the text `nameof` may resolves to something user-defined (this avoids a breaking change)

- `nameof` may not used like a first-class function value, `let f = nameof ;; f x`

- `nameof` may not be used with pipe operator, `x |> nameof`

### Names of instance members 

Names of members must be static or come from an instance. So the following code that attempts to get the name of an instance property with an instance of its containing class is not valid:

```fsharp
type C() =
    member __.M = ()
    
nameof C.M // Error!
```

But the following are valid::

```fsharp
type C() =
    static member M = ()
    member __.M2 = ()
    
nameof C.M // Yay :)

let c = C()
nameof c.M2 // Yay :)
```

NOTE: this is being reconsidered, see unresolved issues below.

### Names of overloaded members require a type annotation:

When selecting an overloaded member, a type annotation may be necessary to select a member:
```fsharp

type MethodGroupNameOfTests() =
    member this.MethodGroup() = ()    
    member this.MethodGroup(i:int) = ()

    member this.MethodGroup1(i:int, f:float, s:string) = 0
    member this.MethodGroup1(f:float, l:int64) = "foo"
    member this.MethodGroup1(u:unit -> unit -> int, h: unit) : unit = ()

    member this.``single argument method group name lookup`` () =
        let b = nameof(this.MethodGroup)
        Assert.AreEqual("MethodGroup",b)

    member this.``multiple argument method group name lookup`` () =
        let b = nameof(this.MethodGroup1 : (float * int64 -> _))
        Assert.AreEqual("MethodGroup1",b)
```

### Names of operators reveal the compiled name of the operator

For example:
```fsharp
nameof(+)   // gives "op_Addition"
```

NOTE: this is being reconsidered, see unresolved issues below.

### Names of generic type instantiations are permitted

The `nameof` construct can be used with generic type instantiations, e.g.

```fsharp
type C2<'T> = A | B
type C3<'T>() = class end

nameof(C2)      // gives "C2"
nameof(C2<int>) // gives "C2"
nameof(C2<_>)   // gives "C2"
nameof(C3)      // gives "C3"
nameof(C3<_>)   // gives "C3"
```

### `nameof` on generic type parameters is not permitted

In the preview release, the `nameof` construct can't be used with type arguments, `let f<'t> (x : 't) = nameof 't`.  This is because syntactcially the argument to `nameof` is an expression, and not a type.

NOTE: this was by design for the preview, but is being reconsidered, see unresolved issues below.

### `nameof` of a function using `RequireExplicitTypeArgumentsAttribute` may require explicit type paramaters

F# functions labelled with the `RequireExplicitTypeArguments` require the use of a ummy type instantiation:

```fsharp
nameof(typeof<_>)
```

NOTE: this attribute is used very rarely.  It is possible this restriction may be lifted in future language revisions.

### Performance considerations

This substitution should be done at compile time, no performance impact expected

## Alternatives

See also unresolved issues below.

#### Alternative: use a keyword not a library intrinsic

This would have been a breaking change.


## Unresolved issues (in preview)

* [ ] `nameof` on an operator gives the compiled name of the operator

* [ ] Resolving instance members requires an artificial instance object.

* [ ] `nameof` may not be used with the name of a generic type parameter.

* [ ] Resolving overloaded members requires a type instantiation.
 