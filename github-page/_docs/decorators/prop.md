---
title: "Prop"
---

`@prop(options: object)` is used for setting properties in a Class (without this set, it is just a type and will **NOT** be in the final model/document)

## Options

### required

Accepts Type: `boolean`

Set if this Property is required (best practice is `public property!: any`, note the `!`)  
For more infomation look at [Mongoose's Documentation](http://mongoosejs.com/docs/api.html#schematype_SchemaType-required)

Example:

```ts
class Something {
  @prop({ required: true }) // this is now required in the schema
  public firstName!: string;

  @prop() // by default, a property is not required
  public lastName?: string; // using the "?" marks the property as optional
}
```

### index

Accepts Type: `boolean`

Create an Index for this Property

-> it should act the same as in the `@index` class decorator, but without options

Example:

```ts
class IndexedClass {
  @prop({ index: true })
  public indexedField?: string;
}
```

### unique

Accepts Type: `boolean`

Create an Index that sets this property to unique

Look at [Mongoose unique](http://mongoosejs.com/docs/api.html#schematype_SchemaType-unique) for more infomation

Example:

```ts
class IndexedClass {
  @prop({ unique: true }) // implicitly has "index: true"
  public uniqueId?: string;
}
```

### default

Accepts Type: `any`

Set an default always when no value is givin at creation time

Example:

```ts
class Defaulted {
  @prop({ default: "hello world" })
  public upperCase?: string; // mark as optional, because it will be defaulted
}
```

### _id

Accepts Type: `boolean`

Set this to `false` if you want to turn of creating ID's for sub-documents

Example:

```ts
class Nested {}

class Parent {
  @prop({ _id: false })
  public nest: Nested;
}
```

### ref

Accepts Type: `Ref<any>`

Set which class to use for Reference (this cannot be inferred by the type)

Example:

```ts
class Nested {}

class Parent {
  @prop({ ref: "Nested" }) // it is a "String" because of reference errors
  public nest: Ref<Nested>;
}
```

### refPath

Accepts Type: `string`

Set at which path to look for which Class to use

Example:

```ts
class Car {}
class Shop {}

// in another class
class Another {
  @prop({ required: true, enum: 'Car' | 'Shop' })
  public which!: string;

  @prop({ refPath: 'which' })
  public kind?: Ref<Car | Shop>;
}
```

### refType

Accepts Type: `mongoose.Schema.Types.Number` \| `mongoose.Schema.Types.String` \| `mongoose.Schema.Types.Buffer` \| `mongoose.Schema.Types.ObjectId`

Set which Type to use for refs

-> [`@prop`'s `type`]({{ site.baseurl }}{% link _docs/decorators/prop.md %}#type) can be used too

```ts
class Nested {}

class Parent {
  @prop({ ref: "Nested", refType: mongoose.Schema.Types.ObjectId }) // it is a "String" because of reference errors
  public nest: Ref<Nested>;
}
```

### validate

Accepts Type: `object` OR `RegExp` OR `(value) => boolean` OR `object[]`
Required options of the object:
  - `validator`: `(value) => boolean`
  - `message`: `String`, the message shows when the validator fails

Set a custom function for validation (must return an boolean)

Example: (For more Examples look at [Mongoose's Documentation](https://mongoosejs.com/docs/api.html#schematype_SchemaType-validate))

```ts
// "maxlength" already exists as an option, this just shows how to use validate
class Validated {
  @prop({ validate: {
    validator: (v) => {
      return v.length <= 10;
    },
    message: "value is over 10 characters long!"
  } })
  public validated?: string;
}
```

### alias

Accepts Type: `string`

Set an Alias for a property (best practice is to add type infomation for it)

-> For more infomation look at [Mongoose's Documentation](https://mongoosejs.com/docs/guide.html#aliases)

Example:

```ts
class Dummy {
  @prop({ alias: "helloWorld" })
  public hello: string; // will be included in the DB
  public helloWorld: string; // will NOT be included in the DB, just for type completion (gets passed as hello in the DB)
}
```

### get & set

Accepts Type: `(input) => output`

set gets & setters for fields, it is not virtual
-> both get & set must be defined all the time, even when you just want to use one, we are sorry

Example:

```ts
class Dummy {
  @prop({ set: (val: string) => val.toLowerCase(), get: (val: string) => val })
  public hello: string;
}
```

### type

Accepts Type: `any`

This option is mainly used for [get & set](#get--set) to override the inferred type  
but it can also be used to override the inferred type of any prop  

-> this overwriting is meant as a last resort, please open a new issue if you need to use it

Example: get as `string[]`, save as `string`

```ts
class Dummy {
  @prop({ set: (val: string[]) => val.join(' '), get: (val: string) => val.split(' '), type: String })
  public hello: string[];
}
```

Example: Overwrite inferred type as last resort

```ts
class Dummy {
  @prop({ type: mongoose.Schema.Types.Mixed }) // used for mongoose / how it is stored to the DB
  public something: NewableFunction; // used for intellisense / typescript
}
```

<!--Below are just the Specific Options-->

### String Transform options

#### lowercase

Accepts Type: `boolean`

Set this to `true` if the value should always be lowercased

Example:

```ts
class LowerCased {
  @prop({ lowercase: true })
  public lowerCase: string; // "HELLO" -> "hello"
}
```

#### uppercase

Accepts Type: `boolean`

Set this to `true` if the value should always be UPPERCASED

Example:

```ts
class UpperCased {
  @prop({ uppercase: true })
  public upperCase: string; // "hello" -> "HELLO"
}
```

#### trim

Accepts Type: `boolean`

Set this to `true` if the value should always be trimmed

Example:

```ts
class Trimmed {
  @prop({ trim: true })
  public trim: string; // "   Trim me   " -> "Trim me"
}
```

### String Validation options

#### maxlength

Accepts Type: `number`

Set a maximal length the string can have

Example:

```ts
class MaxLengthed {
  @prop({ maxlength: 10 })
  public maxlengthed?: string; // the string can only be 10 characters long
}
```

#### minlength

Accepts Type: `number`

Set a minimal length the string must have (must be above 0)

Example:

```ts
class MinLengthed {
  @prop({ minlength: 10 })
  public minlengthed?: string; // the string must be at least 10 characters long
}
```

#### enum

Accepts Type: `enum | any[]`

Only allow Values from the enum (best practice is to use TypeScript's enum)

-> Please know that mongoose currently only allows enums when the type is a `String`

```ts
enum Gender {
  MALE = 'male',
  FEMALE = 'female',
}

class Enumed {
  @prop({ enum: Gender, type: String })
  public gender?: Gender;
}
```

Typegoose disallows enums that dont have strings associated with them (only with the new behaviour)
-> Reason, image the following example:

```ts
// imaginge having this enum
enum Here {
  Here1,
  Here2,
  Here3
}
// it would compile down to ["Here1", "Here2", "Here3"] to be mongoose use-able
class SomeClass {
  @prop({ enum: Here, type: String })
  public somestring: Here;
}

const SomeClassModel = getModelForClass(SomeClass);
const doc = new SomeClassModel({ somestring: Here.Here2 }) // you would expect to be "Here2" right? wrong, it would use the number 1
// which *could* be used, BUT it could be very inaccurate OR when chaning an enum, it would invalidate the whole collection
```

and when `globalOptions.globalOptions.useNewEnum` is activated, typegoose will convert the following:

```ts
enum SomeThing {
  Hi = "Hi SomeOne",
  Hi2 = "Hi SomeThing"
}
// to (only when "useNewEnum" is activated)
["Hi SomeOne", "Hi SomeThing"]
// old behaviour
["Hi", "Hi2"]
```

### Number Validation options

#### max

Accepts Type: `number`

Set a highest number the property can have

Example:

```ts
class Maxed {
  @prop({ max: 10 })
  public maxed?: number; // the value can be at maximum of 10
}
```

#### min

Accepts Type: `number`

Set a lowest number the property can have

Example:

```ts
class Mined {
  @prop({ min: 0 })
  public mined?: number; // the value must be a minimum of 0
}
```
