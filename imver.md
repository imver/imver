# Immutable Versioning (draft)

## Summary

An Immutable Version number is a version number with a format

ivNUMBER(-LABEL) - for example: iv2019.01.01-beta.3

- ‘iv’ identifies the version number as being an Immutable Version number and declares it follows this specification
- NUMBER is a numeric string with any number of optional ‘.’ (dot) characters for human readability. Dots are allowed anywhere in the NUMBER part except first and last character and are considered whitespace - they do not carry information.
- NUMBER with dots removed monotonically increases with every new version
  \* LABEL can be any string of alphanumeric characters, dots or hyphens. Labels contain meta information for humans, such as nature of the release, source package version for language bindings, etc., and are ignored by package managers

In Immutable Versioning

\* Breaking changes in public API or behaviour are strictly not allowed

- Public API calls can be deprecated (in language or documentation), meaning they are no longer supported and issues in them are unlikely to get fixed
- Package management software provides a mechanism to mark packages as deprecated when issues are discovered
- Package management software supports at least two installation modes
  - Normal mode which always installs the most recent version of packages
  - Strict mode which installs the requested version, even if it is deprecated

## Why

Dependency management is one of the most difficult and common problems of software development based on open source software packages. As systems grow, they depend on more and more packages and it gets harder and harder to keep the dependencies up to date, while staying compatible with their APIs and behaviours.

The cause of incompatibilities is in dependencies breaking their original "contract" as they evolve. The APIs change in a way that removes calls, requires more input, provides less output or changes side-effects. Most versioning schemes address this by assigning meanings to portions of the version number. In [Semantic Versioning](https://semver.org) a change of the first part - the major version - signifies a break in compatibility requiring work to upgrade. The version number becomes an agreed way for the package maintainer to signify the intended effect of the update on compatibility.

Conversely, consumers of the packages can usually declare a compatibility mask, that allows them to lock a portion of the version number to a specific number, allowing automatic updates as long as the changes are small enough.

Choosing this mask is a balancing act between receiving automatic patches for longer and assuming too much forward compatibility, where an upgrade assumed to be safe based on its version number breaks compatibility anyway. The ability to mask versions and keep a portion fixed also implies a promise of long-term maintenance of (at least) all the version lines that are meant to be compatible ("major" versions), but sometimes even more granular than that. This leads to a branching of revision history causing a maintenance nightmare for package maintainers trying to fix issues across all supported version lines.

## How

Immutable versioning proposes a solution to these problems by requiring full backwards compatibility of the public APIs and behaviours (side effects) for the lifetime of the package.

This is a strong requirement but it allows safe upgrades and lets the maintainer focus their efforts solely on the latest version of their package. Any break in compatibility is considered an issue to be fixed.

The version number itself is used as metadata to convey information to humans, namely, the ordering of versions. For two versions it is always decidable which of them is the newer one. Authors can chose a versioning scheme to also convey other information, such as the versions age (implying confidence). A recommendation is to use any subset of a date formatted YYYY.MM.DD.HH.MM.SS as the version number.

In order to work as a system, Immutable Versioning also puts a limited set of requirements on maintainers and package management software, and sets maintenance expectations for packages.

Packages can still evolve in backwards compatible ways by:

- Adding new API calls with entirely new behaviour or modified existing behaviour
- Relaxing input requirements (e.g. removing required fields in input arguments. This depends on language and type system) of public APIs
- Expanding return values (e.g. adding extra members to a return collection. This depends on language and type system) of public APIs
- Improving performance of existing APIs
- Deprecating API calls (in documentation and/or by means of the programming language)

Packages versioned with Immutable Versioning are NOT allowed to:

- Remove existing public APIs (supported or deprecated)
  Tighten input requirements (e.g. require extra function arguments) of public APIs
- Restricting return values (e.g. remove members from a return collection) of public APIs
  Add or remove side effects of public APIs
- In order to facilitate safe automatic upgrades, it is necessary for package management systems to allow:
  \* Marking packages as deprecated, signifying there is a known issue with them
- Consumers to decide whether to install any supported (not deprecated) version later than the one requested or exactly the version requested (for unsupervised installation)

Adopting Immutable Versioning sets the following maintenance expectations on the package:

* Issues affecting supported API calls will be fixed and a new version will be released. All version affected by the issue will be deprecated.
* Issues only affecting deprecated API calls may NOT be fixed (as there is a supported version that can be upgraded to).
* Issues only affecting deprecated package versions may NOT be fixed.
* This allows maintainers to focus their effort on the latest versions of the package and effectively ignore deprecated APIs and old versions.

## Immutable Versioning Specification (ImVer)

The key words “MUST”, “MUST NOT”, “REQUIRED”, “SHALL”, “SHALL NOT”, “SHOULD”, “SHOULD NOT”, “RECOMMENDED”, “MAY”, and “OPTIONAL” in this document are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

1. Software using Immutable Versioning MUST declare a public API either in code or in documentation.
1. An Immutable Version number MUST take the form ivNUMBER, where NUMBER is a string of numeric characters or dots (full stops), MUST NOT begin with a zero and MUST NOT contain two or more consecutive dots.
1. The number, with dots removed, MUST increase numerically for subsequently released versions. It is RECOMMENDED to use any subset of a calendar date in the format YYYY.MM.DD.HH.MM.SS as the NUMBER using the UTC time zone. For example: iv2019.03.29
1. An Immutable Version number MAY contain an additional label, following the number part, and separated from it by a hyphen (‘-’). The label MUST be a string of ASCII alphanumeric characters, numbers, dots or hyphens in any order. The label MUST NOT be an empty string. The label is intended for additional metadata and MUST be ignored by package management software. For example: iv2019.03.29-rc.1
1. Once an immutably versioned package has been released the contents of the version MUST NOT be modified. Any modifications MUST be released as a new version.
1. The public APIs which were released MUST NOT be removed and their behaviours MUST NOT change in subsequent releases.
1. In subsequent releases, the public APIs MUST NOT require additional inputs, return fewer outputs or new cases for the consumer to handle, have additional side-effects or remove some expected side-effects. The exact nature of breaking changes depends on language and type system.
1. Subsequent releases MAY add additional public APIs, require fewer inputs than before and return additional outputs or reduce the set of cases for the consumer to handle in a way that will be ignored by the consumer. The exact nature of safe changes depends on language and type system.
1. Individual public APIs MAY be deprecated, in code or in documentation. APIs which were not deprecated are considered supported.
1. Issues in supported public APIs SHOULD be fixed and released as a new version.
1. Issues in deprecated public APIs MAY be fixed. If an issue in a deprecated API isn’t fixed an alternative supported API(s) SHOULD be recommended.
1. Package management software using Immutable Versioning MUST provide a mechanism to deprecate individual package versions retrospectively. Deprecated package versions MUST NOT be removed.
1. Package versions with known issues MUST be deprecated after the issue has been fixed and a fixed version has been released.
1. Package management software using Immutable Versioning MUST provide means to install either the exact specified version of the package, or any supported (not deprecated) version later than the one specified. The latter mode SHOULD be the default behaviour. A ‘conservative’ mode MAY be provided, which installs the first supported version later than the one specified.
1. Package management software SHOULD provide means to resolve an Immutable Version number to a package content fingerprint. Package management software SHOULD provide means to refer to packages by content fingerprint and to resolve a package content fingerprint to all Immutable Versions it was released under.

## About

Immutable Versioning is inspired by [”Spec-ulation” by Rich Hickey](https://www.youtube.com/watch?v=oyLBGkS5ICk) from Clojure Conj 2016 and authored by Viktor Charypar, Tech Director at [Red Badger](https://red-badger.com). The format of this document and the formal specification is based on the [semantic versioning](https://semver.org) specification.

If you wish to leave feedback, open a [GitHub issue](https://github.com/imver/imver/issues).

## License

This work is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)
](https://creativecommons.org/licenses/by/4.0/).
