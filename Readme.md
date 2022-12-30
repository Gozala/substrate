# substrate language

Substrate is an idea of the programming language which is a lisp dialect inspired by [scheme], [clojure] with static types and inference inspired by [elm].

## Mandatory naming

Most languages tend to make naming optional which leads to functions and or types with parameters who's order you have to remember. Many teams (I have worked with) adopt best practice of defining functions with one (or sometimes two) primary arguments and rest as named options. However even such teams have to deal with third party libraries which don't follow such a practice and often there is a temptation to add one more positional parameter.

Substrate simply adopts "at most one anonymous parameter" practice everywhere which is arguably a good tradeoff. In practice it also means that functions are more like unix pipes as they take single input either in form of anonymous parameter or a record with several named parameters.

```clojure
(-> name "hello "name"!")
```

> ℹ️ You can combine named strings together using string interpolation, simply by omitting white space between string literal and string reference

If you want to pass multiple arguments you have to use record of named attributes

```clojure
(-> {first last} "hello "first" "last"!")
```

In fact function with anonymous parameter is a sugar for the following

```clojure
(-> {: name} "hello "name"!")
```

## Protocols

Language adopts idea of a [namespace from clojure] coupled with [module from racket] as a way to declare definitions for in some context

```clojure
(with { :id :substrate.codes/core/0.1/ }
  (type (Result {ok error})
    :ok ok
    :error error))
```

Example above defines `Result` type of `substrate.codes/core/0.1/` protocol. Which is equivalent of

```clojure
(type (Result {ok error})
  :substrate.codes/core/0.1/ok ok
  :substrate.codes/core/0.1/error error)
```

This affects data representation, which is convenient in cases where different protocols may have conflicting definitions or when you want to implement multiple protocols.

### Variants

```clojure
(type (List item)
  :empty {}
  :pair {:head item :tail (List item)))
```

Which is syntactic sugar for

```clojure
(type (List {: item})
  :empty {}
  :pair {:head item :tail (List {: item })))
```

### Records

```clojure
(record Point
  :x Int
  :y Int
)

(record Line :start Point :end Point)
```

Which is syntactic sugar for a variant type

```clojure
(define Point {
  :x Int
  :y Int
})
```

## Attributes

Attributes are labeled values, which are represented as a records with a single field matching the label

```clojure
{ :message "hello" }
```

Sometimes you want attributes that are just labels without any values, those are created as

```clojure
{ :yes }
```

And are just shortcut to an attribute with unit value

```clojure
{ :yes {} }
```

It is possible to define a types for the attribute so it can be referred to in other contexts

```clojure
(define Message {:message Text})
```

### Records

Records are simply set of attributes, they also could be defined as a type so they could be referred to

```clojure
(define Visible
  (field :visible Bit))

(define Position
  (record
    :x Int
    :y Int))

(define View {
  ...Position
  ...Visible
})
```

### Variants

Variants represent an attribute out of the set of attributes

```clojure
(define Visibility
  (variant
    :hidden
    :visible))
```

Variants can also carry payloads

```clojure
(define Event
  (variant
    :PageLoad
    :PageUnload
    :KeyPress Text
    :Paste Text
    :Click { :x Int :y Int }))
```

#### Recursive

```clojure
(record data
  :left (Tree data)
  :right (Tree data))


(variant data
  :leaf data
  :node (Node data))


(define Result
  (variant {ok error}
    :ok ok
    :error error))
```

### Data Types

#### Bits

```clojure
0.
1.
```

#### Natural numbers

```clojure
0
123
0456
123,456,789
```

### Integers

```clojure
+0
-123
+0456
-123,456,789
```

### Fractions

```clojure
+123.456
-456.7890
+0.001
123,456.789
```

### Text

```clojure
"Hello world!"
```

You can also assign text to specific binding.

```clojure
(define name "Alice")
```

You can combine text using interpolation syntax, which is similar to [JS template literals]

```clojure
(define greeting "Hello"name"!")
```

> ℹ️ When text fragments have no spaces between them they are interpreted as concatenation

You can use interpolation syntax to concatenate two text bindings as well by joining them with empty string literal `""`

```clojure
(define greet "Hello")
(define audience "Bob")

(define message ""greet""audience)
```

### Unit

[Unit type] is has only one value (and thus can hold no information). Record with no attributes is such value, so you could define unit type as follows

```clojure
(define Unit {})
```

### Never

Is type who's value can not exist. You could not create a value for the variant with no options, so you could define never type as follows

```clojure
(define Never)
```

> In scientific literature it is often referred to as [bottom type]

# 🚧 Would be nice to use `[]` in arrays

## Examples

```clojure
(let [record {:x 1 :y 2}] record)
; (the {:x Int :y Int} {:x 1 :y 2})

(let [variant {:left record}] variant)
; (the {:left {:x Int :y Int}} {:left {:x 1 :y 2}})


(lambda n (+ n 1))

;; type annotate above function
(the (-> Number Number)
     (=> n (+ n 1)))

(define inc (=> n (+ n 1)))
```

[scheme]: https://en.wikipedia.org/wiki/Scheme_(programming_language)
[clojure]: https://clojure.org/
[elm]: https://elm-lang.org/
[namespace from clojure]: https://clojure.org/reference/namespaces
[module from racket]: https://beautifulracket.com/explainer/modules.html
[js template literals]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals
[bottom type]: https://en.wikipedia.org/wiki/Bottom_type
[unit type]: https://en.wikipedia.org/wiki/Unit_type
