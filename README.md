# @wpo/zod-openapi

Converts a [Zod](https://github.com/colinhacks/zod) schema to an OpenAPI `SchemaObject` as defined by [openapi3-ts](https://www.npmjs.com/package/openapi3-ts)

Forked from [@anatine/zod-plugins](https://github.com/anatine/zod-plugins)

Changed in this fork:

- Uses [defu](https://github.com/unjs/defu) instead of `ts-deepmerge`
- Ships CJS and ESM bundles


## Installation

Both openapi3-ts and zod are peer dependencies instead of dependant packages.
While `zod` is necessary for operation, `openapi3-ts` is for type-casting.

```shell
npm install openapi3-ts zod @wpo/zod-openapi
```

----

## Usage

### Take any Zod schema and convert it to an OpenAPI JSON object

```typescript
import { generateSchema } from '@wpo/zod-openapi';
const aZodSchema = z.object({
  uid: z.string().nonempty(),
  firstName: z.string().min(2),
  lastName: z.string().optional(),
  email: z.string().email(),
  phoneNumber: z.string().min(10).optional(),
})
const myOpenApiSchema = generateSchema(aZodSchema);
// ...
```

This will generate an OpenAPI schema for `myOpenApiSchema`

```json
{
  "type": "object",
  "properties": {
    "uid": {
      "type": "string",
      "minLength": 1
    },
    "firstName": {
      "type": "string",
      "minLength": 2
    },
    "lastName": {
      "type": "string"
    },
    "email": {
      "type": "string",
      "format": "email"
    },
    "phoneNumber": {
      "type": "string",
      "minLength": 10
    }
  },
  "required": [
    "uid",
    "firstName",
    "email"
  ]
}
```

### Extend a Zod schema with additional OpenAPI schema via a function wrapper

```typescript
import { extendApi, generateSchema } from '@wpo/zod-openapi';
const aZodExtendedSchema = extendApi(
      z.object({
        uid: extendApi(z.string().nonempty(), {
          title: 'Unique ID',
          description: 'A UUID generated by the server',
        }),
        firstName: z.string().min(2),
        lastName: z.string().optional(),
        email: z.string().email(),
        phoneNumber: extendApi(z.string().min(10), {
          description: 'US Phone numbers only',
          example: '555-555-5555',
        }),
      }),
      {
        title: 'User',
        description: 'A user schema',
      }
    );
const myOpenApiSchema = generateSchema(aZodExtendedSchema);
// ...
```

... or via extension of the Zod schema:

```typescript
import { extendApi, generateSchema, extendZodWithOpenApi } from '@wpo/zod-openapi';
import {z} from 'zod';

extendZodWithOpenApi(z);

const aZodExtendedSchema = 
      z.object({
        uid: z.string().nonempty().openapi({
          title: 'Unique ID',
          description: 'A UUID generated by the server',
        }),
        firstName: z.string().min(2),
        lastName: z.string().optional(),
        email: z.string().email(),
        phoneNumber: z.string().min(10).openapi({
          description: 'US Phone numbers only',
          example: '555-555-5555',
        }),
      }).openapi(
      {
        title: 'User',
        description: 'A user schema',
      }
    );
const myOpenApiSchema = generateSchema(aZodExtendedSchema);
// ...
```

This will generate an extended schema:

```json
{
  "type": "object",
  "properties": {
    "uid": {
      "type": "string",
      "minLength": 1,
      "title": "Unique ID",
      "description": "A UUID generated by the server"
    },
    "firstName": {
      "type": "string",
      "minLength": 2
    },
    "lastName": {
      "type": "string"
    },
    "email": {
      "type": "string",
      "format": "email"
    },
    "phoneNumber": {
      "type": "string",
      "minLength": 10,
      "description": "US Phone numbers only",
      "example": "555-555-5555"
    }
  },
  "required": [
    "uid",
    "firstName",
    "email",
    "phoneNumber"
  ],
  "title": "User",
  "description": "A user schema"
}
```

----

## Credits

- ### [express-zod-api](https://github.com/RobinTail/express-zod-api)

  A great lib that provided some insights on dealing with various zod types.

- ### [zod-dto](https://github.com/kbkk/abitia/tree/master/packages/zod-dto)

  Lib providing insights into using Zod with NestJS

