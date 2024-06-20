# Rust Development Guidelines at the Laboratory for Web Algorithmics

This file details the guidelines for developing Rust code at the Laboratory for
Web Algorithmics. It is a document _in fieri_, and will be updated as
necessary.  

These guidelines extend the [Rust API
Guidelines](https://rust-lang.github.io/api-guidelines/about.html).

## Tools

- Code should be `clippy`-clean.

- Use `rustfmt` with standard options to format the code. Formatting should
  be enabled as a save action in the editor to reduce the number of spurious
  difference in commits due to spacing and formatting.

- To release new versions:
  - run `cargo c --all-targets` with every feature enabled;
  - run `cargo +nightly fuzz build` if necessary;
  - run `clippy` and `rustfmt` on the code;
  - run `cargo doc` and check the generated docs;
  - run tests with the `slow_tests` feature, if available;
  - bump the version number;
  - run [`cargo semver-checks`](https://crates.io/crates/cargo-semver-checks);
  - update the change log;
  - commit the changes;
  - add on GitHub a new titleless release with a new tag given by the version number, and
    a message given by the entry of the change log;
  - publish the crate (in case of a crate with procedural macros,
    first publish the procedural macros, then test again the main
    crate, and finally publish the main crate).

## Naming Conventions

- <https://github.com/rust-lang/rfcs/blob/master/text/0344-conventions-galore.md>

- As in the Rust standard libraries, traits should use the indefinite/imperative
  form of a verb, whereas implementing structures should use a qualified agent
  associated with the verb: for example, the trait `Read` is implemented by
  `BufReader`. See “Trait Naming” in the link above.

- Modules are directories containing a `mod.rs` file. They should have plural
  names for countables, as in `traits`, `impls`, `utils`, `tests`, and singular
  names for uncountables, such as `fuzz`. Module names should be (very) short.

- Modules that contain a single trait or structure should not be public, and
  the trait or structure should be re-exported by the module that contains it.
  Otherwise, the module should be public and should contain documentation about
  its content.
  
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
simulates them implicitly using `Option` or special values. In functions and
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

  Possible reasons for using type parameters instead of `impl Trait` include:
  - the type parameter is used in the return type of the function or method;
  - the type parameter is used in the body of the function or method.

- Type parameters and `impl Trait` parameters should minimize trait bounds. This
  is a concern similar to
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

- There are a few standard preferences in argument types:
  - always prefer receiving an `IntoIterator` rather than an `Iterator`;
  - always prefer receiving a `AsRef<str>` rather than a `String`, `&str`, or
      `&String`;
  - always prefer receiving a `AsRef<Path>` rather than a `Path`, `&Path`, or
      `&PathBuf`;
  - always prefer receiving a reference to a slice rather than a more specific
    data structure like a vector.

## Tests

- Test functions should be named `test_` followed first by a brief description
  of the tested structure, and then of the specific feature tested, e.g.,
  `test_rank9_empty`.

- Assertions in tests must sport first the actual value, and then the expected
  value. For example,

  ```rust
  assert_eq!(bit_read.read_gamma(), 2);
  ```

- Unit tests that need to access the private parts of a structure should be
  added directly at the end the source file as

  ```rust
  #[cfg(test)]
  mod tests {
      use super::*;
  
      #[test]
      fn test_structure_behavior() -> anyhow::Result<()> {
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

## Errors

- <https://github.com/rust-lang/rfcs/blob/master/text/0236-error-conventions.md>

## Documentation

- <https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md>

- Functions and methods should be documented [in this
  way](https://doc.rust-lang.org/rust-by-example/meta/doc.html).

- Function arguments need not be documented explicitly if the meaning is
  evident from the naming; otherwise, they should be documented as follows:

  ```rust
  /// # Arguments
  ///
  /// * `a` - The first argument, and note that lines should be max 80 characters
  /// long to facilitate reading.
  ///
  /// * `b` - The second argument. Note the empty line above.
  ///
  /// * `c` - The third argument. Note the empty line below.
  ```

- As discussed in the reference above, links should be written in reference
  style, and the references should be placed at the end of the documentation
  block.

- The main crate documentation must be placed in a `README.md` file that
  is included by `lib.rs` using `#![doc = include_str!("../README.md")]`.
  All links in the `README.md` file should be written in reference style, using
  absolute URLs, possibly pointing to the latest version of the crate on
  `docs.rs` for the crate, as in

  ```md
  [`MemCase`]: <https://docs.rs/epserde/latest/epserde/deser/mem_case/struct.MemCase.html>
  ```

- Each project should sport a change log named `CHANGELOG.md` with the
  following sample format:

  ```md
  # Change Log

  ## [0.0.1] - 1970-01-02

  ### New

  * New feature, and note that lines should be max 80 characters long to
    facilitate reading.
  
  * Other new feature.
  
  ### Changed

  * Something changed (not new, not an improvement, not a fix).
  
  ### Improved

  * Improvement.
  
  ### Fixed

  * Bug fix.


  ## [0.0.0] - 1970-01-01

  ### New

  * New feature.
  
  ### Changed

  * Something changed (not new, not an improvement, not a fix).
  
  ### Improved

  * Improvement.
  
  ### Fixed

  * Bug fix.

```
