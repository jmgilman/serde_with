# De/Serialize Transformations Available

This page lists the transformations implemented in this crate and supported by `serde_as`.

1. [Big Array support (Rust 1.51+)](#big-array-support-rust-151)
2. [Borrow from the input for `Cow` type](#borrow-from-the-input-for-cow-type)
3. [`Bytes` with more efficiency](#bytes-with-more-efficiency)
4. [Bytes / `Vec<u8>` to hex string](#bytes--vecu8-to-hex-string)
5. [Convert to an intermediate type using `Into`](#convert-to-an-intermediate-type-using-into)
6. [Convert to an intermediate type using `TryInto`](#convert-to-an-intermediate-type-using-tryinto)
7. [`Default` from `null`](#default-from-null)
8. [De/Serialize with `FromStr` and `Display`](#deserialize-with-fromstr-and-display)
9. [`Duration` as seconds](#duration-as-seconds)
10. [Ignore deserialization errors](#ignore-deserialization-errors)
11. [`Maps` to `Vec` of tuples](#maps-to-vec-of-tuples)
12. [`NaiveDateTime` like UTC timestamp](#naivedatetime-like-utc-timestamp)
13. [`None` as empty `String`](#none-as-empty-string)
14. [One or many elements into `Vec`](#one-or-many-elements-into-vec)
15. [Pick first successful deserialization](#pick-first-successful-deserialization)
16. [Timestamps as seconds since UNIX epoch](#timestamps-as-seconds-since-unix-epoch)
17. [Value into JSON String](#value-into-json-string)
18. [`Vec` of tuples to `Maps`](#vec-of-tuples-to-maps)

## Big Array support (Rust 1.51+)

Support for arrays of arbitrary size.

```ignore
// Rust
#[serde_as(as = "[[_; 64]; 33]")]
value: [[u8; 64]; 33],

// JSON
"value": [[0,0,0,0,0,...], [0,0,0,...], ...],
```

## Borrow from the input for `Cow` type

The types `Cow<'_, str>`, `Cow<'_, [u8]>`, or `Cow<'_, [u8; N]>` (Rust 1.51+) can borrow from the input, avoiding extra copies.

```ignore
// Rust
#[serde_as(as = "BorrowCow")]
value: Cow<'a, str>,

// JSON
"value": "foobar",
```

## `Bytes` with more efficiency

[`Bytes`]

More efficient serialization for byte slices and similar.

```ignore
// Rust
#[serde_as(as = "Bytes")]
value: Vec<u8>,

// JSON
"value": [0, 1, 2, 3, ...],
```

## Bytes / `Vec<u8>` to hex string

[`Hex`]

Requires the `hex` feature.

```ignore
// Rust
#[serde_as(as = "serde_with::hex::Hex")]
value: Vec<u8>,

// JSON
"value": "deadbeef",
```

## Convert to an intermediate type using `Into`

[`FromInto`]

```ignore
// Rust
#[serde_as(as = "FromInto<(u8, u8, u8)>")]
value: Rgb,

impl From<(u8, u8, u8)> for Rgb { ... }
impl From<Rgb> for (u8, u8, u8) { ... }

// JSON
"value": [128, 64, 32],
```

## Convert to an intermediate type using `TryInto`

[`TryFromInto`]

```ignore
// Rust
#[serde_as(as = "TryFromInto<i8>")]
value: u8,

// JSON
"value": 127,
```

## `Default` from `null`

[`DefaultOnNull`]

```ignore
// Rust
#[serde_as(as = "DefaultOnNull")]
value: u32,
#[serde_as(as = "DefaultOnNull<DisplayFromStr>")]
value2: u32,

// JSON
"value": 123,
"value2": "999",

// Deserializes null into the Default value, i.e.,
null => 0
```

## De/Serialize with `FromStr` and `Display`

Useful if a type implements `FromStr` / `Display` but not `Deserialize` / `Serialize`.

[`DisplayFromStr`]

```ignore
// Rust
#[serde_as(as = "serde_with::DisplayFromStr")]
value: u128,
#[serde_as(as = "serde_with::DisplayFromStr")]
mime: mime::Mime,

// JSON
"value": "340282366920938463463374607431768211455",
"mime": "text/*",
```

## `Duration` as seconds

[`DurationSeconds`]

```ignore
// Rust
#[serde_as(as = "serde_with::DurationSeconds<u64>")]
value: Duration,

// JSON
"value": 86400,
```

[`DurationSecondsWithFrac`] supports subsecond precision:

```ignore
// Rust
#[serde_as(as = "serde_with::DurationSecondsWithFrac<f64>")]
value: Duration,

// JSON
"value": 1.234,
```

Different serialization formats are possible:

```ignore
// Rust
#[serde_as(as = "serde_with::DurationSecondsWithFrac<String>")]
value: Duration,

// JSON
"value": "1.234",
```

The same conversions are also implemented for [`chrono::Duration`] with the `chrono` feature.

## Ignore deserialization errors

Check the documentation for [`DefaultOnError`].

## `Maps` to `Vec` of tuples

```ignore
// Rust
#[serde_as(as = "Vec<(_, _)>")]
value: HashMap<String, u32>, // also works with BTreeMap

// JSON
"value": [
    ["hello", 1],
    ["world", 2]
],
```

The [inverse operation](#vec-of-tuples-to-maps) is also available.

## `NaiveDateTime` like UTC timestamp

Requires the `chrono` feature.

```ignore
// Rust
#[serde_as(as = "chrono::DateTime<chrono::Utc>")]
value: chrono::NaiveDateTime,

// JSON
"value": "1994-11-05T08:15:30Z",
                             ^ Pretend DateTime is UTC
```

## `None` as empty `String`

[`NoneAsEmptyString`]

```ignore
// Rust
#[serde_as(as = "serde_with::NoneAsEmptyString")]
value: Option<String>,

// JSON
"value": "", // converts to None

"value": "Hello World!", // converts to Some
```

## One or many elements into `Vec`

[`OneOrMany`]

```ignore
// Rust
#[serde_as(as = "serde_with::OneOrMany<_>")]
value: Vec<String>,

// JSON
"value": "", // Deserializes single elements

"value": ["Hello", "World!"], // or lists of many
```

## Pick first successful deserialization

[`PickFirst`]

```ignore
// Rust
#[serde_as(as = "serde_with::PickFirst<(_, serde_with::DisplayFromStr)>")]
value: u32,

// JSON
// serialize into
"value": 666,
// deserialize from either
"value": 666,
"value": "666",
```

## Timestamps as seconds since UNIX epoch

[`TimestampSeconds`]

```ignore
// Rust
#[serde_as(as = "serde_with::TimestampSeconds<i64>")]
value: SystemTime,

// JSON
"value": 86400,
```

[`TimestampSecondsWithFrac`] supports subsecond precision:

```ignore
// Rust
#[serde_as(as = "serde_with::TimestampSecondsWithFrac<f64>")]
value: SystemTime,

// JSON
"value": 1.234,
```

Different serialization formats are possible:

```ignore
// Rust
#[serde_as(as = "serde_with::TimestampSecondsWithFrac<String>")]
value: SystemTime,

// JSON
"value": "1.234",
```

The same conversions are also implemented for [`chrono::DateTime<Utc>`] and [`chrono::DateTime<Local>`] with the `chrono` feature.

## Value into JSON String

Some JSON APIs are weird and return a JSON encoded string in a JSON response

[`JsonString`]

Requires the `json` feature.

```ignore
// Rust
#[derive(Deserialize, Serialize)]
struct OtherStruct {
    value: usize,
}

#[serde_as(as = "serde_with::json::JsonString")]
value: OtherStruct,

// JSON
"value": "{\"value\":5}",
```

## `Vec` of tuples to `Maps`

```ignore
// Rust
#[serde_as(as = "HashMap<_, _>")] // also works with BTreeMap
value: Vec<(String, u32)>,

// JSON
"value": {
    "hello": 1,
    "world": 2
},
```

This operation is also available for other sequence types.
This includes `BinaryHeap<(K, V)>`, `BTreeSet<(K, V)>`, `HashSet<(K, V)>`, `LinkedList<(K, V)>`, `VecDeque<(K, V)>`, `Option<(K, V)>` and `[(K, V); N]` for all sizes of N.

The [inverse operation](#maps-to-vec-of-tuples) is also available.

[`Bytes`]: crate::Bytes
[`chrono::DateTime<Local>`]: chrono_crate::DateTime
[`chrono::DateTime<Utc>`]: chrono_crate::DateTime
[`chrono::Duration`]: https://docs.rs/chrono/latest/chrono/struct.Duration.html
[`DefaultOnError`]: crate::DefaultOnError
[`DefaultOnNull`]: crate::DefaultOnNull
[`DisplayFromStr`]: crate::DisplayFromStr
[`DurationSeconds`]: crate::DurationSeconds
[`DurationSecondsWithFrac`]: crate::DurationSecondsWithFrac
[`FromInto`]: crate::FromInto
[`Hex`]: crate::hex::Hex
[`JsonString`]: crate::json::JsonString
[`NoneAsEmptyString`]: crate::NoneAsEmptyString
[`OneOrMany`]: crate::OneOrMany
[`PickFirst`]: crate::PickFirst
[`TryFromInto`]: crate::TryFromInto
