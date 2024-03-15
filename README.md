# Rust Development Guidelines at the Laboratory for Web Algorithmics

This file details the guidelines for developing Rust code at the Laboratory for
Web Algorithmics. It is a document _in fieri_, and will be updated as
necessary.  

These guidelines extend the [Rust API
Guidelines](https://rust-lang.github.io/api-guidelines/about.html).

## Naming Conventions

- As in the Rust standard libraries, traits should use the indefinite/imperative
  form of a verb, whereas implementing structures should use a qualified agent
  associated with the verb: for example, the trait `Read` is implemented by
  `BufReader`.

- Modules: TBD

## Structures

- In structures, the first declared fields should be the immutable ones (e.g.,
those that do not change value in the lifetime of the structure) followed by the
mutable ones. In each section, fields should be ordered from the most
general/important to the least general/important. In particular, chains of
dependence should be reflected by the field order.

- `impl` blocks should minimize the trait bounds of the type parameters. This is
  a concern similar to
  [C-STRUCT-BOUNDS](https://rust-lang.github.io/api-guidelines/future-proofing.html#c-struct-bounds).
  For example,

  ```rust
  impl<S: Read + Seek> Foo<S> {
      pub fn bar(seek: S) -> std::io::Result<u64> {
          seek.stream_position()
      }
  }
  ```

  should be

  ```rust
  impl<S: Seek> Foo<S> {
      pub fn bar(seek: S) -> std::io::Result<u64> {
          seek.stream_position()
      }
  }

## Methods

- Rust does not allow for optional parameters with default values, but often one
simulates them implicitly using `Option` or special values. In function and
methods, the first parameters should be the compulsory ones, followed
by the optional ones. In each section, arguments should be ordered from the most
general/important to the least general/important. In particular, chains of
dependence should be reflected by the argument order.

- Prefer `impl Trait` over type parameters whenever possible.
  For example,

```rust
pub fn doit<P: AsRef<Path>>(a: P) {

}
```

is better written as

```rust
pub fn doit(a: impl AsRef<Path>) {

}
```

- Possible reasons for using type parameters instead of `impl Trait` include:
  - the type parameter is used in the return type of the function or method;
  - the type parameter is used in the body of the function or method.

- Type parameters and `impl Trait` parameters trait bounds. This is
  a concern similar to
  [C-STRUCT-BOUNDS](https://rust-lang.github.io/api-guidelines/future-proofing.html#c-struct-bounds).
  For example,

  ```rust
  pub fn bar(seek: impl (Read + Seek)) -> std::io::Result<u64> {
      seek.stream_position()
  }
  ```

  should be

  ```rust
  pub fn bar(seek: impl Seek) -> std::io::Result<u64> {
      seek.stream_position()
  }

## Tests

- Small, short unit tests showcasing the behavior of a structure should
be added directly in the source file as

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test() -> anyhow::Result<()> {
    }
}
```

- Unit-test functions should be named `test_` followed by a brief description of
the tested structure, or the specific feature tested.

- Longer, end-to-end, and possibly slow tests should be placed in a separate
file named `test_*` in the `tests` directory.

- Very slow tests should be gated with the feature `slow_tests`. Ideally, `cargo
  test` should not take more than a few seconds to run.

## Logging

- All binaries and tests using logging (e.g., a `ProgressLogger`) must configure
  an `env_logger` with

```rust
    env_logger::builder()
        .filter_level(log::LevelFilter::Info)
        .try_init()?;
```

or

```rust
    env_logger::builder()
        .is_test(true)
        .filter_level(log::LevelFilter::Info)
        .try_init()?;
```

for tests.
