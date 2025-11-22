# siftly
`siftly` is a lightweight, portable faceted indexing engine for JavaScript and TypeScript.
Build your index once, store it as JSON, and load it instantly anywhere.

It does this by turning structured data into a compact, multi-dimensional index optimized for fast, attribute-based filtering. It is designed for scenarios where you want zero-latency queries without shipping your entire dataset or rebuilding indexes at runtime.

It's designed to be minimal, dependency-free, and works the same in the browser, Node.js, serverless runtimes, workers, and edge environments.


## Features
- **Precompute once:** Build a faceted index from raw objects offline or at build time.
- **JSON-portable:** Serialize the index to disk, GCS, S3, or any CDN. Load it as JSON whenever needed.
- **Instant queries:** Fast equality and facet-based filtering without scanning raw data.
- **Faceted drill-downs:** Get associated label/value pairs to power dynamic filters, dashboards, and guided exploration.
- **Universal runtime support:** Works in browsers, Node.js, serverless functions, web workers, and edge runtimes.
- **Zero dependencies:** Fully written in TypeScript with strong types and predictable behaviors.


## Use cases

`siftly` is ideal for:
- Analytics dashboards
- Data-heavy SaaS applications
- E-commerce faceted search.
- Static-site or offline search over structured metadata.
- Edge/worker applications that must load data quickly.
- Mobile and embedded apps with limited compute.
- Local-first applications.
- Any system where latency matters more than memory footprint.

## Installation

```bash
npm install siftly
```
```bash
yarn add siftly
```
```bash
pnpm add siftly
```
```bash
bun add siftly
```

## Quick start

1. Build an index from your data.

    ```typescript
    import { siftly } from 'siftly';

    const data = [
      { id: 1, category: 'fruit', color: 'red' },
      { id: 2, category: 'fruit', color: 'yellow' },
      { id: 3, category: 'vegetable', color: 'green' },
    ];

    const index = await siftly.create(data, {
      category: 'category',
      color: 'color',
    });
    ```

2. Query the index.

    ```typescript
    const results = index.getItems({ category: 'fruit' });
    // → [ { id: 1, category: 'fruit', color: 'red' }, { id: 2, category: 'fruit', color: 'yellow' } ]
    ```

3. Explore associated label/value pairs.

    ```typescript
    index.getAssociatedLabels({ category: 'fruit' });
    // → Map { 'color' => Set(['red', 'yellow']) }
    ```

4. Serialize and store.

    ```typescript
    const json = siftly.serialize(index);
    // Save to GCS/S3/local filesystem/etc.
    ```

5. Load and query the index later.

    ```typescript
    const index = siftly.deserialize(json);
    const results = index.getItems({ category: 'fruit' });
    // → [ { id: 1, category: 'fruit', color: 'red' }, { id: 2, category: 'fruit', color: 'yellow' } ]
    ```

## Lexicon

- **Index:** A precomputed structure that groups items by combinations of label/value pairs.
- **Labels:** Logical dimensions you want to filter on. Example: "category", "color", "size", "price", etc.
- **LabelValues:** Unique combinations of "label=value" pairs stored as integer IDs for compactness.
- **Items:** Your actual dataset, grouped by the label-value combinations they match.

## API

### `siftly.create(data, keymap)`

Builds and initializes an index.

- `data: Record<string, unknown>[]` array of objects
- `keyMap: { labelName: KeyNameInData }` object that maps label names to their corresponding keys in the data.

```typescript
const index = await siftly.create(users, { 
  role: 'role',
  status: 'status',
});
// → Index { items: [ { id: 1, role: 'admin', status: 'active' }, { id: 2, role: 'user', status: 'inactive' } ], labels: [ 'role', 'status' ] }
```

### `index.getItems(query?)`

Returns items that match the given label/value pairs.

- `filter`: `{ label: value }`

```typescript
const results = index.getItems({ role: 'admin' });
// → [ { id: 1, role: 'admin', status: 'active' } ]
```

If no query is provided, returns all items.

```typescript
const results = index.getItems();
// → [ { id: 1, role: 'admin', status: 'active' }, { id: 2, role: 'user', status: 'inactive' } ]
```

### `index.getLabelValues(label)`

Returns all disctinct values for a given label. Useful for building dropdowns, filters, or any UI that depends on the available values for a given dimension.


- `label: string`

```typescript
index.getLabelValues('role');
// → Set(['admin', 'user', 'guest'])
```


### `index.getAssociatedLabels(query)`

Returns a `Map<string, Set<unknown>>` describing all other label/value pairs that co-occur with the given query. Ideal for faceted search, drill-down analytics, and dynamic filtering.

- `query: { label: value }`

```typescript
index.getAssociatedLabels({ status: 'active' });
// → Map { 'role' => Set(['admin', 'user']) }
```

### `siftly.serialize(index)`

Serializes an index into a JSON-safe representation. The resulting JSON is portable, stable, and can be loaded in any environment without access to the original dataset.


```typescript
const json = siftly.serialize(index);
// Save to disk, GCS, S3, CDN, etc.
```

### `siftly.deserialize(json)`

Rehydrates an index from previously serialized JSON. The resulting index is identical to the original, but can be loaded from disk, GCS, S3, CDN, etc. You can immediately run queries without rebuilding or scanning raw data.

```typescript
const saved = await fetch('/my-index.json').then(r => r.json());
const index = siftly.deserialize(saved);
```

## Performance

### Indexing Performance
- Proportional to number of items × number of labels.
- Runs asynchronously, chunked to avoid blocking the event loop.
- Intended for offline or build-time execution.

### Query Performance
- Proportional to the number of unique label-value combinations.
- Does not scan raw data.
- Equality-based queries return results near-instantly.

### Memory Usage
- Favors speed over minimal memory footprint.
- Ideal for read-heavy workloads, dashboards, offline apps, and edge runtimes.

## When to use `siftly`

Use `siftly` when you need:
- Zero-latency client-side or edge queries
- Multi-dimensional filtering over structured data
- A portable index that loads instantly in any environment
- Static or semi-static datasets
- Deterministic, reproducible indexing
- Faceted drill-downs or guided data exploration

Do not use `siftly` for:
- Range queries (e.g. “price between 10 and 50”)
- Full-text search
- Real-time or high-frequency mutations
- Multi-million row datasets without sharding

## Roadmap

Planned enhancements:
- [ ] Incremental updates (partial rebuilds)
- [ ] Alternate indexing modes (posting lists, typed arrays)
- [ ] Sharded index loading
- [ ] WebAssembly acceleration
- [ ] Enhanced statistics and metadata
- [ ] Optional compression of serialized JSON
- [ ] Developer tooling for validation and schema checks

If you have ideas, feedback, or feature requests, please open an issue or PR.

## Contributing

Contributions are welcome! Please open an issue to discuss large changes before submitting a PR.
The project is written in TypeScript with zero dependencies, and contributions should maintain that philosophy.

## License

`siftly` is [licensed under the MIT License](LICENSE) and you are free to use this library in commercial and open-source projects.