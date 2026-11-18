# Walk through the DummyJSON Products tap

The SDK repository includes a runnable REST tap in `packages/meltano-tap-dummyjson`.
This walkthrough follows one stream, `products`, from discovery to its first two
requests.

## 1. Register a stream

`tap_dummyjson/tap.py` defines the tap and its configuration schema. The
`[project.scripts]` entry in the package's `pyproject.toml` exposes its CLI as
`tap-dummyjson`. The SDK calls `discover_streams` to find the streams, both when
building a catalog and when running the tap.

```{literalinclude} ../../packages/meltano-tap-dummyjson/tap_dummyjson/tap.py
:language: python
:start-at: def discover_streams
:end-before: if __name__
:dedent: 4
```

## 2. Describe the response

In `tap_dummyjson/streams.py`, `Products` requests `/products` and selects one record
per element of the response's `products` array. The `id` field is the primary key.
The stream also declares a JSON Schema for the records it emits.

```{literalinclude} ../../packages/meltano-tap-dummyjson/tap_dummyjson/streams.py
:language: python
:start-at: class Products(
:end-before: replication_key
```

## 3. Follow a page

`tap_dummyjson/client.py` connects the SDK's `OffsetPaginator` to the request
parameters. The authenticator adds a bearer token separately. With a page size of 25,
the tap sends these pagination parameters on the first two requests when both
pages are needed:

```text
GET /products?skip=0&limit=25
GET /products?skip=25&limit=25
```

Other parameters can also be present. For example, `Products.get_http_request` can
send `modifiedAfter` when a starting timestamp is available. That describes what
the tap sends; it does not guarantee that the server filters its results.

## 4. Discover and run the tap

From the SDK repository root, create a local `config.json` using your DummyJSON
demo login, and keep it out of version control. To use a test server instead, add
its base URL as `api_url`.

```json
{
  "username": "<demo username>",
  "password": "<demo password>"
}
```

```console
uv run --package meltano-tap-dummyjson --no-dev tap-dummyjson --config config.json --discover > catalog.json
uv run --package meltano-tap-dummyjson --no-dev tap-dummyjson --config config.json --catalog catalog.json
```

The first command writes a catalog with one stream. The second reads that catalog
and writes Singer messages to standard output, including `SCHEMA` and `RECORD`
messages for `products`.
