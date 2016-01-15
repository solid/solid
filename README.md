# Solid

[![](https://img.shields.io/badge/project-Solid-7C4DFF.svg?style=flat-square)](https://github.com/solid/solid)

![Solid Logo](https://avatars3.githubusercontent.com/u/14262490?v=3&s=200)

> Re-decentralizing the web

Solid (derived from "**so**cial **li**nked **d**ata") is a proposed set of
conventions and tools for building *decentralized social applications* based on
[Linked Data](http://www.w3.org/DesignIssues/LinkedData.html) principles. Solid
is modular and extensible. It relies as much as possible on existing
[W3C](http://www.w3.org/) standards and protocols.

Specifically, Solid is:

* A tech stack -- a set of complementary
  [standards](https://github.com/solid/solid-spec#standards-used) and
  [data formats/vocabularies](https://github.com/solid/vocab) that together
  provide capabilities that are currently available only through centralized
  social media services (think Facebook/Twitter/LinkedIn/many others), such as
  *identity*, *authentication and login*, *authorization and permission lists*,
  *contact management*, *messaging and notifications*, *feed aggregation and
  subscription*, *comments and discussions*, and more.
* A **[Specifications document](https://github.com/solid/solid-spec)**
  that describes a REST API that extends those existing
  standards, contains design notes on the individual components used, and is
  intended as a guide for developers who plan to build servers or applications.
* A set of [servers](https://github.com/solid/solid-platform#servers) that
  implement this specification.
* A [test suite](https://github.com/solid/solid-tests) for testing and validating
  Solid implementations.
* An ecosystem of [social apps](https://github.com/solid/solid-apps),
  [identity providers](https://github.com/solid/solid-idp-list) and helper
  libraries (such as [solid.js](https://github.com/solid/solid.js)) that run on
  the Solid platform.
* A community providing documentation, discussion (see the
  [solid gitter channel](https://gitter.im/solid/solid)),
  [tutorials](https://github.com/solid/solid#tutorials) and
  [talks/presentations](https://github.com/solid/talks).

## Solid Project Workflow

To contribute to Solid development, and to bring up issues or feature requests,
please use the following workflow:

1. Have a question or a feature request or a concern about the Solid framework,
  or on one of its servers? **Open an issue on
  [solid/solid](https://github.com/solid)** (this repo here).

2. Have an issue with the *Solid spec* specifically? **Open an issue on
  [solid/solid](https://github.com/solid) anyway.** And then, as a result of
  discussion, if it's agreed that it is actually a Spec issue, it will be moved
  to `solid-spec`.

3. The individual [solid/solid](https://github.com/solid) issues can coordinate
  and track component/dependent issues on the various affected Solid servers,
  apps, and so on.

## Project directory

- Useful links
  - [Specification](https://github.com/solid/solid-spec)
  - [Talks](https://github.com/solid/talks)
  - Tutorials
    - [Intro to Solid Tutorial](https://github.com/solid/solid-tutorial-intro)
      introductory Solid tutorial using the
      [solid.js](https://github.com/solid/solid.js) library
    - [Solid Angular Apps Tutorial](https://github.com/solid/solid-tutorial-angular):
      a set of example apps built on the Solid platform using AngularJS.


- Implementing
  - [Sign-up/Login application](https://github.com/solid/solid-signup)
  - [Solid Server Test suite](https://github.com/solid/solid-tests)

- Community
  - [Server implementations](https://github.com/solid/solid-platform)
  - [Identity providers](https://github.com/solid/solid-idps)
  - [Applications](https://github.com/solid/solid-apps)
  - Join our conversations on [gitter](https://gitter.im/solid/solid)
