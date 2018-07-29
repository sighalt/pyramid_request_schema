# pyramid_request_schema

Validation of incoming requests is a must-have for modern rest-like APIs.

## What this package will do for you

- validate incoming JSON-Requests as defined by your colander schemas
- raise HTTP 400 (Bad request) if the request does not match your schema
- output readable error message in case of an invalid request
- stay out of your way in any other cases

## Installation

You can simply install `pyramid_request_schema` via pip:

```bash
$ pip install pyramid_request_schema
```

## Configuration

You can either include `pyramid_request_schema` in your ini file

```
pyramid.includes =
    ...
    pyramid_request_schema
    ...
```

or programmatically in your main function

```python

def main(global_config, **settings):
    config = Configurator(...)
    ...
    config.include("pyramid_request_schema")

```

Each of both includes does nothing but registering the view deriver
`schema_validated_view`.


## Usage

The usage of `pyramid_request_schema` is dead simple. Define a proper
colander schema representing your expected input:

```python
class TestRequestSchema(colander.MappingSchema):

    @colander.instantiate()
    class body(colander.MappingSchema):
        str_val = colander.SchemaNode(colander.String())
        float_val = colander.SchemaNode(colander.Float())

    @colander.instantiate()
    class queryset(colander.MappingSchema):
        page = colander.SchemaNode(colanger.Integer(),
                                   validator=colander.Range(1))

```

and add the `request_schema` option to your view configuration:

```python
@view_config(request_schema=TestRequestSchema)
def my_view(request):
    pass
```

You can then access the validated data via the `validated` attribute on the
request object. The resulting structure for the above defined schema and the
following curl request will look like:

```python
request.validated = {
    "body": {
        "str_val": "Some arbitrary string",
        "float_val": 12.34,
    },
    "queryset": {
        "page": 1
    }
}
```

```bash
$ curl -H "content-type: application/json" \
    --data='{"str_val": "Some arbitrary string", "float_val": "12.34"}' \
    http://example.com/?page=1
```
