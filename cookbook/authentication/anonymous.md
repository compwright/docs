# Anonymous authentication

Anonymous authentication can be allowed by creating a [custom strategy](../../api/authentication/strategy.md) that returns the `params` that you would like to use to identify an authenticated user.

:::: tabs :options="{ useUrlFragment: false }"
::: tab "JavaScript"
In `src/authentication.js`:

```js
const { AuthenticationBaseStrategy } = require('@feathersjs/authentication');

class AnonymousStrategy extends AuthenticationBaseStrategy {
  async authenticate(authentication, params) {
    return {
      anonymous: true
    }
  }
}

module.exports = app => {
  // ... authentication service setup
  authentication.register('anonymous', new AnonymousStrategy());
}
```
:::
::: tab "TypeScript"
```ts
import { Params } from '@feathersjs/feathers';
import { AuthenticationBaseStrategy, AuthenticationResult } from '@feathersjs/authentication';

class AnonymousStrategy extends AuthenticationBaseStrategy {
  async authenticate(authentication: AuthenticationResult, params: Params) {
    return {
      anonymous: true
    }
  }
}

export default function(app: Application) {
  // ... authentication service setup
  authentication.register('anonymous', new AnonymousStrategy());
}
```
:::
::::


Next, we create a hook called `allow-anonymous` that sets `params.authentication` if it does not exist and if `params.provider` exists (which means it is an external call) to use that `anonymous` strategy:

:::: tabs :options="{ useUrlFragment: false }"
::: tab "JavaScript"
```js
/* eslint-disable require-atomic-updates */
module.exports = function (options = {}) { // eslint-disable-line no-unused-vars
  return async context => {
    const { params } = context;

    if(params.provider && !params.authentication) {
      context.params = {
        ...params,
        authentication: {
          strategy: 'anonymous'
        }
      }
    }

    return context;
  };
};
```
:::
::: tab "TypeScript"
```ts
import { Hook, HookContext } from '@feathersjs/feathers';

export default (): Hook => {
  return async (context: HookContext) => {
    const { params } = context;

    if(params.provider && !params.authentication) {
      context.params = {
        ...params,
        authentication: {
          strategy: 'anonymous'
        }
      }
    }

    return context;
  }
}
```
:::
::::

This hook should be added __before__ the [authenticate hook](../api/authentication/hook.md) wherever anonymous authentication should be allowed:

```js
all: [ allowAnonymous(), authenticate('jwt', 'anonymous') ],
```

If an anonymous user now accesses the service externally, the service call will succeed and have `params.anonymous` set to `true`.
