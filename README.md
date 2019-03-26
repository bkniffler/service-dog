# service-dog

Teach your service-dogs whatever skill it needs and let it handle your requests.

As soon as you send an action, service-dog will run through all of its skills, mutating the action's value along until it is done. Each skill can also hook into the return chain to modify the value again. The executional interface to service-dog is based on promises, though internally, due to performance and flexibility, service-dog uses callbacks. Thus, it can easily handle async stuff like http requests, image transformation, etc.

Service-dog is based a lot on the idea of middlewares (made popular by expressjs) but is completely agnostic to what kind of operations it handles.

## Example

Just a basic example of chaining skills.

### Basic

```ts
const dog = new ServiceDog();
// Pickup
dog.train('pickup', (type, payload, flow) => {
  if (type === 'throw') {
    flow({ ...payload, isInMouth: true });
  } else {
    flow(payload);
  }
});
// Bring back
dog.train('bring-back', (type, payload, flow) => {
  if (type === 'throw' && payload.isInMouth) {
    flow({ ...payload, position: 'next-to-human' });
  } else {
    flow(payload);
  }
});
// Let fall
dog.train('letfall', (type, payload, flow) => {
  if (
    type === 'throw' &&
    payload.isInMouth &&
    payload.position === 'next-to-human'
  ) {
    flow({ ...payload, isInMouth: false, drooled: drooled });
  } else {
    flow(payload);
  }
});
// Perform an action on your dog
const result = await dog.send<any>('throw', {
  name: 'Stick #1',
  drooled: false,
  position: 'far-away'
});
console.log(result.drooled); // => true;
console.log(result.position); // => 'next-to-human';
```

### Complex operations

```ts
const dog = new ServiceDog();
dog.train('bring-back', async (type, payload, flow) => {
  // Access the context
  const humanNotPatient = flow.get('bad-day');
  if (type === 'throw' && payload.isInMouth) {
    if (humanNotPatient) {
      // Break the forward chain and start going backwards immediately
      flow.return({ ...payload, position: 'next-to-human' });
    } else if (!humanNotPatient) {
      // Start a new action and wait
      await flow.send('play', { with: payload.name });
      // Continue your chain
      flow({ ...payload, position: 'next-to-human' });
    } else {
      // Restart with a different action
      flow.restart('play', { lostInterestInStick: true });
    }
  } else {
    flow(payload);
  }
});
const result = await dog.send<any>(
  'throw',
  {
    name: 'Stick #1',
    drooled: false,
    position: 'far-away'
  },
  {
    'bad-day': true // set the context
  }
);
console.log(result.drooled); // => true;
console.log(result.position); // => 'next-to-human';
```
