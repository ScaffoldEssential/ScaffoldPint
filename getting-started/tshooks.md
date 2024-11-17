# tsHooks

There are several tsHooks currently being developed. \
&#x20;\
Install the below package in your root directory to be able to acccess TS Hooks&#x20;

```bash
npm i use-essential-pint
```

Starting using the hooks in any TS/JS Projects\


```typescript
import usePint from "use-essential-pint";
```

```typescript
const data = usePint(builderEndPointApi : String, solutionJson : JSON)
```

Import the hooks from the npm package\
\
`usePint()` - It runs the solution for the respective builder api endpoints and takes in two input \
&#x20; \- `builderApiEndpoint` - Smart Contract Api Endpoint\
&#x20; \- `SolutionJson` - You solution in JSON format\
\
And it returns an object with the output data and error handling variables\
\
`{`\
`"data" : "F6C5D3455C41801BC6E82901E9142DB068E17569AD15953C8C402072B8E70FAB",`\
`"isError" : "...",`\
`"isPending" : "..."`\
`}`

