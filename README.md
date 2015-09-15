# typeScript-enum-export-bug

I *think* this is an atom-typescript error as I've tried compiling this from typescript@next directly, and it seems to compile fine.

The error relates to enums, and exports using the new typescript node resolution (with the typings property in the package.json).

The bug can be reproduced through the following scenario: (Sadly I can't attach a .zip, so ask me if you want the contents directly for easy import and analysis):

Three projects: ```a```, ```b```, ```c```.

```b``` depends on ```a```
```c``` depends on ```b```

```a``` exports an enum through its index.ts:

```TypeScript
export enum E{}
```

```b``` then exports a function that returns ```E``` through its index.ts:

```TypeScript
import {E} from 'a'
export function f():E {
  return null
}
```

```c``` then consumes ```f``` through its index.ts:

```TypeScript
import {E} from 'a'
import {f} from 'b'

let e:E = f()
```

There is now an error for ```e```:

```
Type 'E' is not assignable to type 'E'.
```

All three packages have a package.json that looks like this:
```
{
  "name": "b",
  "version": "1",
  "main": "lib/index.js",
  "typings": "lib/index.d.ts",
  "dependencies": {
    "a": "1"
  }
}
```

And a tsconfig.json that looks like:
```
{
    "compilerOptions": {
        "declaration": true,
        "module": "commonjs",
        "rootDir": "./src",
        "target": "es5",
        "outDir": "./lib"
    },
    "files": [
        "src/index.ts"
    ]
}
```

Now, what's especially interesting, is if you have the rootDir and outDir the same (of package ```a```), then the error doesn't occur.

Furthermore, this defect occurs if ```f``` were to export another symbol (e.g. an interface) which references an exported enum somewhere within it's structure or hierarchy (which was how I encountered this bug, and it took me some time to track it down to this specific scenario)

This only seems to affect enums, not other types.
