# Language description

Waves Contracts language is:

* Lazy
* Strong typed
* Statically typed
* Expression-based.

Operations and constructs available are:

* Binary operations:`>=`, `>`, `<`, `<=`, `+`, `-`, `&&`, `||`, `*`, `%`, `/`, `==`, `!=`
* Unary operations `-`, `!`
* Constants declaration via `let`
* `IF-THEN-ELSE` clause
* Accessing fields of any of instances of predifined sctructures via `.`
* Accessing list element by index `[]`
* Calls to predefined functions via `()`

## Available data types

* `Long`
* `String`
* `Boolean`
* `ByteVector`
* `Unit` - is analogous to method which is declared in some languages as `void`.
* `Option[T]`- implements as Option `T | Unit`
* `List[T]`
* Predefined non-recursive data structure like `Transaction`, `Block`etc
* `Nothing`- "bottom type", no instance of this type can exist

### Structures

* `Address(ByteVector)`
* `Alias(String)`
* `DataType.Long`
* `DataType.Boolean`
* `DataType.ByteArray`
* `DataType.String`

Structure types can be instantiated by calling their constructor, which has the same name as the type itself, and passing the necessary parameters:
```
let addr = Address(base58'3My3KZgFQ3CrVHgz6vGRt8687sH4oAA1qp8')
let alias = Alias("alicia")
```

### Option\[T\]

`Option[T]` can be represented as

* `Some(t)` for any `t` of type `T`
* `None`
* `extract`:`Option[T] => T`,`isDefined`:`Option[T] => Boolean`defined over`Option[_]`

It is higher-kind data type indicating a possibility of absence of value of inner type. This rather popular construct from functional programming languages\(Haskell, Scala, Clojure\) nowdays gets included in majority of standard libraries for many popular languages like Java, C\#, etc.


### List\[T\]

User can't create `List[T]` instances but input data can contains some `List[T]` fields.
Now all of transactions contain field `proofs: List[ByteVector]` and MassTransfer transactions contain field `transfers: List[Transfer]`.

To access lists element you can use syntax `list[index]` with the first element at index 0.

To determinate count of lists elements you can use function `size`:
* `size `: `List[T] => Long`
This is also true for `DataType.ByteArray`:
* `size `: `DataType.ByteArray => Long`

### ByteVector
Standart ByteVector type 

* `size `: `ByteVector => Long`
* `take`: `ByteVector`, `Long` => `ByteVector`
* `drop`: `ByteVector`, `Long` => `ByteVector`
* `dropRight`: `ByteVector`, `Long` => `ByteVector`
* `takeRight`: `ByteVector`, `Long` => `ByteVector`

### Long

* `fraction(value: LONG, numerator: LONG, denominator: LONG) => LONG`

### String
Standart string type

* `size `: `String => Long`
* `take`: `String`, `Long` => `String`
* `drop`: `String`, `Long` => `String`
* `takeRight`: `String`, `Long` => `String`
* `dropRight`: `String`, `Long` => `String`

# Examples

Here's a complete multisig 2 of 3 example. Keep in mind all`let`s could actually be inlined, they exist only for the sake of readability.

```
let alicePubKey  = base58'B1Yz7fH1bJ2gVDjyJnuyKNTdMFARkKEpV'
let bobPubKey    = base58'7hghYeWtiekfebgAcuCg9ai2NXbRreNzc'
let cooperPubKey = base58'BVqYXrapgJP9atQccdBPAgJPwHDKkh6A8'

let aliceSigned  = if(sigVerify(tx.bodyBytes, tx.proofs[0], alicePubKey  )) then 1 else 0
let bobSigned    = if(sigVerify(tx.bodyBytes, tx.proofs[1], bobPubKey    )) then 1 else 0
let cooperSigned = if(sigVerify(tx.bodyBytes, tx.proofs[2], cooperPubKey )) then 1 else 0

aliceSigned + bobSigned + cooperSigned >= 2
```

