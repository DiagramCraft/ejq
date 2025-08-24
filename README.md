# ejq - Embeddable JQ Implementation

A lightweight, embeddable TypeScript implementation of the [jq](https://jqlang.github.io/jq/) command-line JSON processor. Designed for easy integration into web applications and JavaScript environments.

## Features

- **~95% JQ Feature Complete** - Supports most jq operations and built-in functions
- **Small Bundle Size** - Only 8.52kb compressed (35kb uncompressed)
- **TypeScript Native** - Written in TypeScript with full type definitions
- **Zero Dependencies** - No external runtime dependencies
- **Multiple Formats** - ESM, CommonJS, and UMD builds included
- **Browser & Node.js** - Works in both browser and Node.js environments

## Installation

```bash
npm install ejq
```

## Quick Start

```typescript
import { parseAndQuery } from 'ejq';

// Basic field access
const data = { name: "John", age: 30 };
const result = parseAndQuery('.name', [data]);
console.log(result); // ["John"]

// Array operations
const users = [{ name: "John" }, { name: "Jane" }];
const names = parseAndQuery('.[] | .name', [users]);
console.log(names); // ["John", "Jane"]

// Complex queries
const complex = { users: [{ name: "John", posts: ["Hello", "World"] }] };
const posts = parseAndQuery('.users[] | .posts[]', [complex]);
console.log(posts); // ["Hello", "World"]
```

## API Reference

### `parseAndQuery(query: string, input: unknown[], bindings?: Record<string, unknown>): unknown[]`

Parses a jq query string and executes it against the input data.

- `query` - The jq query string
- `input` - Array of input values to process
- `bindings` - Optional variable bindings for the query
- Returns array of results

### `parse(query: string, includeBuiltins?: boolean): Generator`

Parses a jq query string into an executable generator.

- `query` - The jq query string to parse
- `includeBuiltins` - Whether to include built-in function definitions (default: true)
- Returns a generator that can be executed with `query()`

### `query(parsedQuery: Generator, input: unknown[], bindings?: Record<string, unknown>): Generator`

Executes a pre-parsed query against input data.

- `parsedQuery` - The parsed query generator from `parse()`
- `input` - Array of input values to process  
- `bindings` - Optional variable bindings for the query
- Returns generator yielding results

## Usage Examples

### Basic Operations

```typescript
import { parseAndQuery } from 'ejq';

// Identity - returns input unchanged
parseAndQuery('.', [42]); // [42]

// Field access
parseAndQuery('.foo', [{ foo: "bar" }]); // ["bar"]

// Array indexing
parseAndQuery('.[1]', [[1, 2, 3]]); // [2]

// Array slicing
parseAndQuery('.[1:3]', [[0, 1, 2, 3, 4]]); // [[1, 2]]
```

### Array Operations

```typescript
// Iterate array elements
parseAndQuery('.[]', [[1, 2, 3]]); // [1, 2, 3]

// Map over array
parseAndQuery('map(. + 1)', [[1, 2, 3]]); // [[2, 3, 4]]

// Filter array
parseAndQuery('.[] | select(. > 1)', [[1, 2, 3]]); // [2, 3]

// Array construction
parseAndQuery('[.foo, .bar]', [{ foo: 1, bar: 2 }]); // [[1, 2]]
```

### Object Operations

```typescript
// Object construction
parseAndQuery('{name: .name, age: .age}', [{ name: "John", age: 30 }]);
// [{ name: "John", age: 30 }]

// Multiple outputs
parseAndQuery('.name, .age', [{ name: "John", age: 30 }]); 
// ["John", 30]

// Recursive descent
parseAndQuery('.. | .foo?', [{ a: { foo: "found" } }]); // ["found"]
```

### Built-in Functions

```typescript
// Type checking
parseAndQuery('type', ["hello"]); // ["string"]
parseAndQuery('type', [42]); // ["number"]

// Length
parseAndQuery('length', [["a", "b", "c"]]); // [3]
parseAndQuery('length', ["hello"]); // [5]

// Keys
parseAndQuery('keys', [{ b: 2, a: 1 }]); // [["a", "b"]]

// Has key
parseAndQuery('has("foo")', [{ foo: 1, bar: 2 }]); // [true]

// String functions
parseAndQuery('tostring', [42]); // ["42"]
parseAndQuery('tonumber', ["42"]); // [42]
```

### Advanced Queries

```typescript
// Complex data transformation
const data = {
  users: [
    { name: "John", posts: [{ title: "Hello" }, { title: "World" }] },
    { name: "Jane", posts: [{ title: "Hi" }] }
  ]
};

// Get all post titles
parseAndQuery('.users[] | .posts[] | .title', [data]);
// ["Hello", "World", "Hi"]

// Group by user
parseAndQuery('.users[] | {name: .name, titles: [.posts[].title]}', [data]);
// [{ name: "John", titles: ["Hello", "World"] }, { name: "Jane", titles: ["Hi"] }]
```

### Using Variables

```typescript
// With variable bindings
parseAndQuery('$var + .", " + .name', [{ name: "World" }], { var: "Hello" });
// ["Hello, World"]

// Define and use variables
parseAndQuery('.name as $n | .age as $a | "\($n) is \($a) years old"', 
              [{ name: "John", age: 30 }]);
// ["John is 30 years old"]
```

## Supported JQ Features

### Core Operators
- Identity (`.`)
- Field access (`.foo`, `.["foo"]`)
- Array indexing (`.[0]`, `.[-1]`)
- Array slicing (`.[1:3]`, `.[2:]`, `.[:2]`)
- Iterator (`.[]`)
- Recursive descent (`..`)
- Pipe (`|`)
- Comma (`,`)

### Data Construction
- Object construction (`{foo: .bar}`)
- Array construction (`[.foo, .bar]`)
- String interpolation (`"Hello \(.name)"`)

### Built-in Functions
- Type functions: `type`, `length`, `keys`, `keys_unsorted`, `values`, `empty`
- Test functions: `has()`, `in()`, `contains()`, `inside()`
- Array functions: `map()`, `select()`, `sort()`, `sort_by()`, `group_by()`, `unique`, `reverse`, `add`, `min`, `max`, `flatten`
- String functions: `tostring`, `tonumber`, `ascii_downcase`, `ascii_upcase`, `split()`, `join()`
- Math functions: `floor`, `ceil`, `round`, `sqrt`, `abs`
- Date functions: `now`, `strftime()`, `strptime()`

### Control Flow
- Conditionals: `if-then-else`
- Try operator: `?`
- Alternative operator: `//`
- Error handling with `try-catch`

### Advanced Features
- Variable binding (`as $var`)
- Function definitions (`def`)
- Reduce expressions (`reduce`)
- Path expressions (`path()`, `paths`, `getpath()`, `delpaths()`)

## Browser Usage

```html
<script type="module">
  import { parseAndQuery } from './node_modules/ejq/dist/ejq.js';
  
  const result = parseAndQuery('.name', [{ name: 'Browser' }]);
  console.log(result); // ['Browser']
</script>
```

## Performance

ejq is optimized for embeddability and small bundle size while maintaining good performance:

- Minimal memory allocation during query execution
- Lazy evaluation using generators
- No external dependencies
- Tree-shakeable when using ES modules

## Limitations

While ejq implements ~95% of jq functionality, some advanced features are not yet supported:

- Some complex built-in functions
- Module system (`import`/`include`)
- Some edge cases in complex recursive operations
- Advanced streaming operations

## License

MIT License - see [LICENSE](LICENSE) file for details.