# Working Memory State

This is where we can store some state information that can be used in your lambda (or other applications)

## Use Case: Who am I and where I belong

Our lambda middleware uses this state information to create some state information for us.

As of right now, whenever we call our lambda, we must pass in a `by-id` and a `client-id`.

Based on these two variables, lambda will do validations so we can make the following calls

### `getMe()`

returns `UserVo`

#### Usage Example

```typescript
import { getModelWorkingMemState } from "@cscore/cs-singleton";
import { User } from "@cscore/cs-package";

export const fn = (): any => {
    const workingState = getModelWorkingMemState();
    const me: User = workingState.getMe();
    const authorId = me.getId();
    console.log("🚀 ~ authorId:", authorId)
}
```

### `getProperty(string)`

`params`

The following will get automatically populated

| name     | description           |
| -------- | --------------------- |
| clientId | returns the client id |

But, you may add your own properties to the working memory state and the function returns whatever value that gets stored in the property

#### Usage Example

```typescript
import { getModelWorkingMemState } from "@cscore/cs-singleton";


export const fn = (): any => {
    const workingState = getModelWorkingMemState();
    const clientId = workingState.getStateProperty('clientId');
    console.log("🚀 ~ clientId:", clientId)
}
```
