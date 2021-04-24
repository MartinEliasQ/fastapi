# Respuestas personalizadas - HTML, Stream, Archivos, otros

Por defecto, **FastAPI** devolverá las respuestas utilizando `JSONResponse`.

Se puede anular devolviendo una `Response` directamente como se ve en [Return a Response directly](response-directly.md){.internal-link target=_blank}.

Pero si devuelves una `Response` directamente, los datos no se convertirán automáticamente, y la documentación no se generará automáticamente (por ejemplo, incluyendo el "media type" específico, en la cabecera HTTP `Content-Type` como parte de la OpenAPI generada).

Pero también puedes declarar la `Response` que quieras que se utilice, en *decorador de operacion de la ruta*.

El contenido que devuelva tu *decorador de operacion de la ruta* se pondrá dentro de esa`Response`.

Y si `Response` es de tipo JSON (`application/json`), como es el caso de `JSONResponse` y `UJSONResponse`, los datos que devuelvas se convertirán automáticamente (ya filtrados) con cualquier Pydantic `response_model` que declares en el *decorador de operacion de la ruta*.

!!! note
    Si utiliza una clase de respuesta sin media type, FastAPI esperará que tu respuesta no tenga contenido, por lo que no documentará el formato de la respuesta en tus documentos OpenAPI generados.

## Usar `ORJSONResponse`

Por ejemplo, si estás exprimiendo el rendimiento, puede instalar y utilizar <a href="https://github.com/ijl/orjson" class="external-link" target="_blank">`orjson`</a> y establecer que la respuesta sea `ORJSONResponse`.

Importa la clase `Response` (sub-clase) que tu quieras utilizar y declárala e el *decorador de operacion de la ruta*.

```Python hl_lines="2  7"
{!../../../docs_src/custom_response/tutorial001b.py!}
```

!!! info
    El parámetro `response_class` también se utilizará para definir el "media type" de la respuesta.

    En este caso, la cabecera HTTP `Content-Type` se establecerá como `application/json`.

    Y se documentará como tal en OpenAPI.

!!! tip
    `ORJSONResponse` actualmente sólo está disponible en FastAPI, no en Starlette.

## HTML Response

TPara devolver una respuesta con HTML directamente desde **FastAPI**, usa `HTMLResponse`

* Import `HTMLResponse`.
* Pass `HTMLResponse` as the parameter `response_class` of your *decorador de operacion de la ruta*.

```Python hl_lines="2  7"
{!../../../docs_src/custom_response/tutorial002.py!}
```

!!! info
     El parámetro  `response_class` también se utilizará para definir el "media type" de la respuesta.

    En este caso, la cabecera HTTP  `Content-Type` se establecerá como  `text/html`.

    Y se documentará como tal en OpenAPI.

### Retornando una `Response`

Como se ha visto en [Return a Response directly](response-directly.md){.internal-link target=_blank}, también puedes anular la respuesta directamente en tu  *operacion de ruta*, retornandolo.

El mismo ejemplo anterior, devolviendo un `HTMLResponse`, podría verse como:

```Python hl_lines="2  7  19"
{!../../../docs_src/custom_response/tutorial003.py!}
```

!!! warning
    Un `Response` retornado directamente desde la *path operation function* no se documentará en OpenAPI (por ejemplo, el `Content-Type` no se documentará) y no será visible en los documentos interactivos automáticos.

!!! info
    Por supuesto, la cabecera `Content-Type`, codigos de estado, etc, provendrán del objeto `Response` que hayas retornado.

### Document in OpenAPI and override `Response`

If you want to override the response from inside of the function but at the same time document the "media type" in OpenAPI, you can use the `response_class` parameter AND return a `Response` object.

The `response_class` will then be used only to document the OpenAPI *path operation*, but your `Response` will be used as is.

#### Return an `HTMLResponse` directly

For example, it could be something like:

```Python hl_lines="7  21  23"
{!../../../docs_src/custom_response/tutorial004.py!}
```

In this example, the function `generate_html_response()` already generates and returns a `Response` instead of returning the HTML in a `str`.

By returning the result of calling `generate_html_response()`, you are already returning a `Response` that will override the default **FastAPI** behavior.

But as you passed the `HTMLResponse` in the `response_class` too, **FastAPI** will know how to document it in OpenAPI and the interactive docs as HTML with `text/html`:

<img src="/img/tutorial/custom-response/image01.png">

## Available responses

Here are some of the available responses.

Have in mind that you can use `Response` to return anything else, or even create a custom sub-class.

!!! note "Technical Details"
    You could also use `from starlette.responses import HTMLResponse`.

    **FastAPI** provides the same `starlette.responses` as `fastapi.responses` just as a convenience for you, the developer. But most of the available responses come directly from Starlette.

### `Response`

The main `Response` class, all the other responses inherit from it.

You can return it directly.

It accepts the following parameters:

* `content` - A `str` or `bytes`.
* `status_code` - An `int` HTTP status code.
* `headers` - A `dict` of strings.
* `media_type` - A `str` giving the media type. E.g. `"text/html"`.

FastAPI (actually Starlette) will automatically include a Content-Length header. It will also include a Content-Type header, based on the media_type and appending a charset for text types.

```Python hl_lines="1  18"
{!../../../docs_src/response_directly/tutorial002.py!}
```

### `HTMLResponse`

Takes some text or bytes and returns an HTML response, as you read above.

### `PlainTextResponse`

Takes some text or bytes and returns an plain text response.

```Python hl_lines="2  7  9"
{!../../../docs_src/custom_response/tutorial005.py!}
```

### `JSONResponse`

Takes some data and returns an `application/json` encoded response.

This is the default response used in **FastAPI**, as you read above.

### `ORJSONResponse`

A fast alternative JSON response using <a href="https://github.com/ijl/orjson" class="external-link" target="_blank">`orjson`</a>, as you read above.

### `UJSONResponse`

An alternative JSON response using <a href="https://github.com/ultrajson/ultrajson" class="external-link" target="_blank">`ujson`</a>.

!!! warning
    `ujson` is less careful than Python's built-in implementation in how it handles some edge-cases.

```Python hl_lines="2  7"
{!../../../docs_src/custom_response/tutorial001.py!}
```

!!! tip
    It's possible that `ORJSONResponse` might be a faster alternative.

### `RedirectResponse`

Returns an HTTP redirect. Uses a 307 status code (Temporary Redirect) by default.

```Python hl_lines="2  9"
{!../../../docs_src/custom_response/tutorial006.py!}
```

### `StreamingResponse`

Takes an async generator or a normal generator/iterator and streams the response body.

```Python hl_lines="2  14"
{!../../../docs_src/custom_response/tutorial007.py!}
```

#### Using `StreamingResponse` with file-like objects

If you have a file-like object (e.g. the object returned by `open()`), you can return it in a `StreamingResponse`.

This includes many libraries to interact with cloud storage, video processing, and others.

```Python hl_lines="2  10-11"
{!../../../docs_src/custom_response/tutorial008.py!}
```

!!! tip
    Notice that here as we are using standard `open()` that doesn't support `async` and `await`, we declare the path operation with normal `def`.

### `FileResponse`

Asynchronously streams a file as the response.

Takes a different set of arguments to instantiate than the other response types:

* `path` - The filepath to the file to stream.
* `headers` - Any custom headers to include, as a dictionary.
* `media_type` - A string giving the media type. If unset, the filename or path will be used to infer a media type.
* `filename` - If set, this will be included in the response `Content-Disposition`.

File responses will include appropriate `Content-Length`, `Last-Modified` and `ETag` headers.

```Python hl_lines="2  10"
{!../../../docs_src/custom_response/tutorial009.py!}
```

## Default response class

When creating a **FastAPI** class instance or an `APIRouter` you can specify which response class to use by default.

The parameter that defines this is `default_response_class`.

In the example below, **FastAPI** will use `ORJSONResponse` by default, in all *path operations*, instead of `JSONResponse`.

```Python hl_lines="2  4"
{!../../../docs_src/custom_response/tutorial010.py!}
```

!!! tip
    You can still override `response_class` in *path operations* as before.

## Additional documentation

You can also declare the media type and many other details in OpenAPI using `responses`: [Additional Responses in OpenAPI](additional-responses.md){.internal-link target=_blank}.
