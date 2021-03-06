<div align="center">
  <a href="https://github.com/bkniffler/flowzilla">
    <img alt="flowzilla" src="https://raw.githubusercontent.com/bkniffler/flowzilla/master/assets/logo.png" height="250px" />
  </a>
</div>
<div align="center">
  <strong>Create flexible and extandable data flows by encapsulating operations into skills, making an ordered chain that can handle whatever request you throw at it.</strong>
  <br />
  <br />
  <a href="https://travis-ci.org/bkniffler/flowzilla">
    <img src="https://img.shields.io/travis/bkniffler/flowzilla.svg?style=flat-square" alt="Build Status">
  </a>
  <a href="https://codecov.io/github/bkniffler/flowzilla">
    <img src="https://img.shields.io/codecov/c/github/bkniffler/flowzilla.svg?style=flat-square" alt="Coverage Status">
  </a>
  <a href="https://github.com/bkniffler/flowzilla">
    <img src="http://img.shields.io/npm/v/flowzilla.svg?style=flat-square" alt="Version">
  </a>
  <a href="https://github.com/bkniffler/flowzilla">
    <img src="https://img.shields.io/badge/language-typescript-blue.svg?style=flat-square" alt="Language">
  </a>
  <a href="https://github.com/bkniffler/flowzilla/master/LICENSE">
    <img src="https://img.shields.io/github/license/bkniffler/flowzilla.svg?style=flat-square" alt="License">
  </a>
  <a href="https://github.com/bkniffler/flowzilla">
    <img src="https://flat.badgen.net/bundlephobia/minzip/flowzilla" alt="License">
  </a>
  <a href="https://github.com/bkniffler/flowzilla">
    <img src="https://img.shields.io/david/bkniffler/flowzilla.svg?style=flat-square" alt="Dependencies">
  </a>
  <br />
  <br />
</div>

# flowzilla

Create flexible and extandable data flows by encapsulating operations into skills, making an ordered chain that can handle whatever request you throw at it. As soon as you send an action, flowzilla will run through all of its skills, mutating the action's value along until it is done. Each skill can also hook into the return chain to modify the value on its way back. The executional interface to flowzilla is based on promises, though internally, due to performance and flexibility, flowzilla uses callbacks. Thus, it can easily handle async stuff like http requests, image transformation, etc.

Flowzilla is based a lot on the idea of middlewares (made popular by expressjs) but is completely agnostic to what kind of operations it handles. It is also inspired by redux, though only on the action/middleware part. It excels in cases where you'd work with class overwriting and/or hooks to allow extending some part of functionality. A classic example is a database with different adapters and plugins (like soft-delete, auditing, time-stamping), a server that can handle requests, an authentication/authorization system, an API client. Basically anything that has a strong focus on how data flows.

It can also help you check your flow by providing a tracker that will fire on start and completion of your chain and each time a skill is entered.

This library works on node and in the browser, has no dependencies except for tslib (no dependencies, 2kb gzipped) and is tree shackable.

## Get started

https://codepen.io/bkniffler/pen/eoNRWo?editors=0012

```js
const { Flowzilla } = flowzilla;

// Declare the retry skill
function retrySkill(type, value, flow) {
  // Get retries from context
  const currentRetries = flow.get('retries', 0);
  // Compare with maxRetries from context
  if (currentRetries < flow.get('maxRetries', 0)) {
    // Call reset on fail, providing the current retries as new context
    flow.catch(err => flow.reset(type, value, { retries: currentRetries + 1 }));
  }
  // Continue normally
  flow(value);
}

// Declare the fetch skill
async function fetchSkill(type, value, flow) {
  // Check current type
  if (type === 'fetch') {
    // Fetch using browser' fetch & return the json response
    flow.return(await fetch(value).then(response => response.json()));
  } else {
    flow(value);
  }
}

// Define your own class, inheriting Flowzilla to define your own methods
class MyHTTPClient extends Flowzilla {
  constructor() {
    super();
    // Add both skills from above
    this.addSkill('retry', retrySkill);
    this.addSkill('fetch', fetchSkill);
  }
  // Define a fetch method that calls flowzilla.run with 'fetch' action type, a url and maxRetries as context
  fetch(url) {
    return this.run('fetch', url, {
      maxRetries: 3
    });
  }
}

// Try it!
async function work() {
  const client = new MyHTTPClient();
  console.log(
    await client.fetch('https://jsonplaceholder.typicode.com/todos/1')
  );
}

work();
```

# Table of Contents

- [Install](#install)
  - Yarn/NPM
  - CDN
- [API Documentation](#api-documentation)
  - [Flowzilla](#flowzilla)
  - [Skill](#skill)
- [Guides](#guides)
  - [Context](#context)
  - [Error Handling](#error-handling)
- [Examples](#examples)
  - [HTTP Client](#http-client)
  - Database Example
  - Event emitting
- [Performance](#performance)

## Install

### Yarn/NPM

```
yarn add flowzilla
npm i flowzilla
```

### CDN

A browser version is available on https://cdn.jsdelivr.net/npm/flowzilla

# API Documentation

Flowzillas' structure is fairly simple. There is a Flowzilla class, which you add Skill functions to. Then you can run your flows. Read more about all API arguments below.

## Flowzilla

```js
const flowzilla = new Flowzilla();
flowzilla.addSkill((type, value, flow) => {
  if (type === 'append') {
    value.push(1);
  }
  flow(value);
});
flowzilla
  .run('append', [0])
  .then(result => console.log('Result', result))
  .catch(err => console.error(err));
```

### Methods

#### constructor

Initiate a new instance, optionally providing a name.

- `const flowzilla = new Flowzilla(name?: string)`

#### flowzilla.addSkill

Add a skill to flowzilla. Read more about skills in the #Skill section. You can control the order of your skills by either adding them in the according order or by providing the position.

- `flowzilla.addSkill(skill: Skill): void`
- `flowzilla.addSkill(name: string, skill: Skill): void`
- `flowzilla.addSkill(name: string, skill: Skill, position: 'AFTER'|'BEFORE'|'START'|'END', anchor?: any | any[]): void`

```js
// Add a skill with named function
function firstSkill(type, value, flow) {
  flow(value);
}
flowzilla.addSkill(firstSkill);

// Simple skill that does nothing
const simpleSkill = (type, value, flow) => flow(value);
// Add a skill with name
flowzilla.addSkill('skill1337', simpleSkill);
// Add a skill at start
flowzilla.addSkill(simpleSkill, 'START');
// Add a skill after firstSkill
flowzilla.addSkill(simpleSkill, 'AFTER', firstSkill);
// Add a skill before skill1337
flowzilla.addSkill(simpleSkill, 'BEFORE', 'skill1337');
// Add a skill before firstSkill and skill1337
flowzilla.addSkill(simpleSkill, 'BEFORE', [firstSkill, 'skill1337']);
// Add multiple skills
flowzilla.addSkill([simpleSkill, simpleSkill]);
```

#### flowzilla.removeSkill

Remove a skill from flowzilla, either by name or by the function.

- `flowzilla.removeSkill(skill: Skill | string): void`

#### flowzilla.skill(s)Count

Number of skills currently in flowzilla instance.

- `flowzilla.skill(s)Count: number`

#### flowzilla.run

Dispatch an action into flowzilla, optionally providing an initialValue and a context.

- `flowzilla.run<T>(type: string, initialValue?: any, context?: any): Promise<T>`

```js
flowzilla
  .run('fetch', { id: '123' }, { accessToken: 'BEARER 123' })
  .then(result => console.log('Result', result))
  .catch(err => console.error(err));
```

#### flowzilla.runSync

Dispatch an action into flowzilla, optionally providing an initialValue and a context. Will return whatever first skill returns. Handy for adding change-listeners.

- `flowzilla.runSync<T>(type: string, initialValue: any, context: any): Promise<T>`

```js
flowzilla.addSkill(function calc(type, value, flow) {
  if (type === 'multiply') {
    return value.reduce((sum, n) => sum * n, 1);
  }
});
const value = flowzilla.runSync('multiply', [2, 2]); // => 4;
```

## Skill

A skill is only just a function

- `(type: string, value: any, flow: Flow): void`

```js
const calculator = (type, value, flow) => {
  if (type === 'add') {
    flow.return(value[0] + value[1]);
  } else if (type === 'multiply') {
    flow.return(value[0] * value[1]);
  } else {
    // Do nothing
    flow(value);
  }
};
```

### Arguments

#### type

Type is a string that is defined when calling Flowzilla.run.

#### value

The value can be anything, and it can be altered in each skill.

#### flow

`flow` exposes multiple functions to control your data flow, read more in the section below.

### Flow

- `flow(newValue)` will continue to next skill (if any) or return (if none)
- `flow.return(finalValue)` will force to return with specified value instead of proceeding to next
- `flow.run('new-action', value): Promise` will start a new flow and await its value before continuing
- `flow.reset('new-action', value)` will stop the current flow and start a new one
- `flow.catch((err, previousErrorHandler) => void)` will add an error handler for subsequent flows
- `flow.get('key', defaultValue)` get value from context
- `flow.set('key', value)` set value in context

# Guides

## Context

You can get and set context inside of skills using `flow.set(key: string, value: any)` and `flow.get(key: string, defaultValue?: any)`.
Check below for an example.

## Error Handling

You can catch any errors by using `flowzilla.run(...).catch(err => void)`, but you can also catch subsequent skills from inside a skill using `flow.catch((err, previousErrorHandler) => void)`.

Here is a simple retry mechanism using error handlers and context.

```js
const flowzilla = new Flowzilla();
flowzilla.addSkill('retry', (type, value, flow) => {
  const maxRetries = flow.get('maxRetries', 0);
  const currentRetries = flow.get('retries', maxRetries);
  if (currentRetries < 3) {
    // Call reset on fail, providing the current retries as new context
    flow.catch(err => flow.reset(type, value, { retries: currentRetries + 1 }));
  }
  flow(value);
});
flowzilla.addSkill('error', async (type, value, flow) => {
  if (type === 'fetch') {
    flow.return(await fetch(value).then(response => response.json()));
  }
});
flowzilla
  // Provide context
  .run('fetch', 'https://jsonplaceholder.typicode.com/todos/1', {
    maxRetries: 3
  })
  .then(result => console.log('Result', result))
  .catch(err => console.error(err));
```

# Examples

## HTTP Client

Checkout [Codepen Playground](https://codepen.io/bkniffler/pen/eoNRWo?editors=0012)

# Performance

The overhead of running flowzilla compared to using callbacks or a chain of promises is very low.
Checkout the (simple) benchmark at: https://github.com/bkniffler/flowzilla/blob/master/src/tests/benchmark.test.ts

```bash
# Macbook Pro 13
callback x 761 ops/sec ±1.80% (74 runs sampled)
promise x 750 ops/sec ±0.99% (76 runs sampled)
flowzilla x 758 ops/sec ±0.98% (74 runs sampled)
```
