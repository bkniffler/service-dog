<div align="center">
  <a href="https://github.com/bkniffler/service-dog">
    <img alt="alpacka" src="https://raw.githubusercontent.com/bkniffler/service-dog/master/assets/logo.png" height="250px" />
  </a>
</div>
<div align="center">
  <strong>Teach your service-dog whatever skills it needs and let it handle your requests.</strong>
  <br />
  <br />
  <a href="https://travis-ci.org/bkniffler/service-dog">
    <img src="https://img.shields.io/travis/bkniffler/service-dog.svg?style=flat-square" alt="Build Status">
  </a>
  <a href="https://codecov.io/github/bkniffler/service-dog">
    <img src="https://img.shields.io/codecov/c/github/bkniffler/service-dog.svg?style=flat-square" alt="Coverage Status">
  </a>
  <a href="https://github.com/bkniffler/service-dog">
    <img src="http://img.shields.io/npm/v/service-dog.svg?style=flat-square" alt="Version">
  </a>
  <a href="https://github.com/bkniffler/service-dog">
    <img src="https://img.shields.io/badge/language-typescript-blue.svg?style=flat-square" alt="Language">
  </a>
  <a href="https://github.com/bkniffler/service-dog/master/LICENSE">
    <img src="https://img.shields.io/github/license/bkniffler/service-dog.svg?style=flat-square" alt="License">
  </a>
  <a href="https://github.com/bkniffler/service-dog">
    <img src="https://flat.badgen.net/bundlephobia/minzip/service-dog" alt="License">
  </a>
  <br />
  <br />
</div>

# service-dog

Create flexible and extandable data flows by encapsulating operations into skills, making an ordered chain that can handle whatever request you throw at it. As soon as you send an action, service-dog will run through all of its skills, mutating the action's value along until it is done. Each skill can also hook into the return chain to modify the value on its way back. The executional interface to service-dog is based on promises, though internally, due to performance and flexibility, service-dog uses callbacks. Thus, it can easily handle async stuff like http requests, image transformation, etc.

Service-Dog is based a lot on the idea of middlewares (made popular by expressjs) but is completely agnostic to what kind of operations it handles. It is also inspired by redux, though only on the action/middleware part. It excels in cases where you'd work with class overwriting and/or hooks to allow extending some part of functionality. A classic example is a database with different adapters and plugins (like soft-delete, auditing, time-stamping), a server that can handle requests, an authentication/authorization system, an API client. Basically anything that has a strong focus on how data flows.

It can also help you check your flow by providing a tracker that will fire on start and completion of your chain and each time a skill is entered.

This library works on node and in the browser, has no dependencies except for tslib (no dependencies, 2kb gzipped) and is tree shackable.

## Docs

- [Documentation](https://bkniffler.github.io/service-dog/)
- [Typedoc](https://bkniffler.github.io/service-dog/typedoc)
- [CodeSandbox Playground](https://codesandbox.io/s/pp3zwnxk7m)

## Install

```
yarn add service-dog
npm i service-dog
```

### CDN

A browser version is available on https://cdn.jsdelivr.net/npm/service-dog
