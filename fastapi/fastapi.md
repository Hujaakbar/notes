# FastAPI

For in detail explanation refer to [official docs](https://fastapi.tiangolo.com/tutorial/).

## Quick Tips and Reminders

1. No trailing slashes in URL paths
1. all the type-hinted input parameters are validated by Pydantic
1. all the type-hinted output data are validated by Pydantic
1. Prefer return type over `response_model` for better editor support.
1. always declare status code

## Table Of Contents

- [How to Install FastAPI](#install-fastapi)
- [Basic Explanation](#basic-explanation)
- [Operation Parameters](#operation-parameters)
  - [Path parameters](#path-parameters)
  - [Query parameters](#query-parameters)
  - [Request body parameters](#request-body-parameters)
  - [Cookie parameters](#cookie-parameters)
  - [Header parameters](#header-parameters)
  - [Form Fields (parameters)](#form-data)
  - [Uploading files](#request-files)
- [Response Model](#response-model)
- [Response Status Code](#response-status-codes)
- [Handling and Returning Errors](#handling-and-returning-errors)
  - [How It works](#how-it-works)
  - [Overriding Default Exception Handlers](#overriding-default-exception-handlers)
  - [Creating Custom Exception Handlers](#creating-custom-exception-handlers)

## Install FastAPI

Install FastAPI without `fastapi-cloud-cli`

```txt
uv add "fastapi[standard-no-fastapi-cloud-cli]"
```

It installs below packages

```txt
uv tree --depth 2

fastapi
    ├── annotated-doc
    ├── pydantic
    ├── starlette
    ├── typing-extensions
    ├── typing-inspection
    ├── email-validator
    ├── fastapi-cli
    ├── httpx
    ├── jinja2
    ├── pydantic-extra-types
    ├── pydantic-settings
    ├── python-multipart
    └── uvicorn[standard]
```

### FastAPI VSCode extension

There is official [FastAPI Extension](https://marketplace.visualstudio.com/items?itemName=FastAPILabs.fastapi-vscode) for VSCode.

After installing the extension, in VSCode settings, uncheck below settings:

- `fastapi.telemetry.enabled`
- `fastapi.cloud.enabled`

## Basic Explanation

[Main docs](https://fastapi.tiangolo.com/tutorial/first-steps/#recap-step-by-step)

FastAPi is similar to Flask.

- create an instance of `FastAPI`
- use it to create `get`, `post` etc. endpoints

```python
from fastapi import FastAPI

app = FastAPI()


@app.get(path="/greeting/")
def greeting(name: str):
    return {"hello": "world"}
```

### Operations

In FastAPI jargon, HTTP methods like `GET`, `POST` etc. are called "**operations**".

The following operation function is called by FastAPI whenever it receives a request to the URL `/api/v1/greeting/<some_name>` using a `GET` operation.

```python
from fastapi import FastAPI

app = FastAPI()


# GET operation decorator
@app.get(path="/api/v1/greeting/{name}")
# GET operation function
def sample_endpoint(name: str):
    # the output data is converted to JSON automatically
    return {"hello": name}
```

> Note 📝
>
> 1. Outgoing (output) data is converted to JSON automatically.
> 1. When a type hints/annotations are used, whether it is simple data types like `str`, `int` or Pydantic models, FastAPI carries out data validation using Pydantic.

Serve the app using development server:

```txt
uv run fastapi dev
```

It serves the app at [http://127.0.0.1:8000]( http://127.0.0.1:8000).
Auto-generated interactive docs live at [http://127.0.0.1:8000/docs]( http://127.0.0.1:8000/docs).

FastAPI also provides auto-generated docs using ReDoc, but it is not interactive. So, I found it less useful.

### Define Entrypoint

To make it easy for `fastapi cli`, VSCode extensions, and other tools to find the FastAPI app, define the `entrypoint` field in `pyproject.toml`:

```toml
[tool.fastapi]
# entrypoint = "path.to.file:appName"
entrypoint = "backend.main:app"
```

## Operation Parameters

- [Path parameters](#path-parameters)
- [Query parameters](#query-parameters)
- [Request body parameters](#request-body-parameters)
- [Cookie parameters](#cookie-parameters)
- [Header parameters](#header-parameters)
- [Form Fields (parameters)](#form-data)
- [Uploading files](#request-files)

Operation function can take a number of parameters such as path parameters, query parameters, header parameters and request body parameters.

As long as a parameter has a type hint (does not matter simple types like `str`, `int` or Pydantic models), FastAPI validates it using Pydantic.

> Note 📝
> FastAPI validates **all** parameters using Pydantic

### Path parameters

Path parameter is a parameter defined in the path part of the `@app.operation()`

It can use simple type hints:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get(path="/books/{book_id}")
def book(book_id: int): ...
```

It can also use `Annotated[type, Path()]` annotation:

```python
from fastapi import FastAPI, Path

app = FastAPI()


@app.get(path="/books/{book_id}")
def book(book_id: Annotated[type, Path(ge=1)]): ...
```

#### Enums in path parameters

When a path parameter uses enum type hints, the generated docs show enum values as available values and during the testing it displays them as drop-down options list.

```python
from enum import Enum

from fastapi import FastAPI

app = FastAPI()


class RoseColors(str, Enum):
    red = "red"
    pink = "pink"
    black = "black"
    white = "white"


@app.get("/rose/{rose_color}")
def get_model(rose_color: RoseColors):
    return {"you chose": f"{rose_color} color"}
```

![available options](./images/available_colors.png)

![drop-down options](./images/drop_down_list.png)

### Query parameters

A parameter is recognized as a query parameter in following cases:

- not a part of the path parameters and it is of a singular type (like `int`, `float`, `str`, `bool`, etc) i.e. not a Pydantic Model
- not a part of the path parameters, and uses `Annotated[type, Query()]` annotation

Example 1. Singular types:

```python
from fastapi import FastAPI

app = FastAPI()


# book_id is a path parameter
# skip is a query parameter
# limit is a query parameter
@app.get("/books/{book_id}")
def book_pages(book_id: int, skip: int = 0, limit: int = 10):
    return {"book_id": book_id, "skip": skip, "limit": limit}
```

Example 2. `Annotated` type hints:

```python
from typing import Annotated

from fastapi import FastAPI, Path, Query

app = FastAPI()


@app.get("/pages/{book_id}")
def get_model(
    book_id: Annotated[int, Path(ge=1)],
    start: Annotated[int, Query(ge=1, description="starting page number")],
    pages: Annotated[int, Query(ge=1, description="the num of pages to return")] = 10,
):
    return {"book": book_id, "pages": {"from": start, "until": start + pages}}
```

URL:

```txt
http://127.0.0.1:8000/pages/2?start=20&pages=15
```

Example 3. `Annotated` type hints with Pydantic Model:

```python
from typing import Annotated

from fastapi import FastAPI, Path, Query
from pydantic import BaseModel, Field

app = FastAPI()


class QueryParams(BaseModel):
    start: int = Field(ge=1, description="starting page number")
    pages: int = Field(ge=1, description="the num of pages to return", default=10)


@app.get("/pages/{book_id}")
def get_model(
    book_id: Annotated[int, Path(ge=1)], query_params: Annotated[QueryParams, Query()]
):
    return {
        "book": book_id,
        "pages": {
            "from": query_params.start,
            "until": query_params.start + query_params.pages,
        },
    }
```

URL:

```txt
http://127.0.0.1:8000/pages/2?start=20&pages=15
```

### Request Body parameters

A parameter is recognized as body parameter in following cases:

- the parameter is declared to be a Pydantic model type
- the parameter is declared with `Annotated[type, Body()]` annotation

```python
from datetime import date

from fastapi import FastAPI, status
from pydantic import BaseModel, Field

app = FastAPI()


class Book(BaseModel):
    title: str = Field(max_length=100)
    authors: list[str] = Field(min_length=1)
    published_date: date | None = Field(default=None, examples=["2008-09-15"])


# user_id is a path parameter
# book is a body parameter
# return_content is a query parameter
@app.post(path="/users/{user_id}/book", status_code=status.HTTP_201_CREATED)
def add_book(user_id: int, book: Book, return_content: bool = False):
    if return_content:
        return {"message": "Book is added", "user_id": user_id, "book": book}
    return {"message": "Book is added", "user_id": user_id}
```

`Annotated[type, Body()]` example:

```python
from datetime import date
from typing import Annotated

from fastapi import Body, FastAPI, status
from pydantic import BaseModel, Field, HttpUrl

app = FastAPI()


class Image(BaseModel):
    url: HttpUrl
    name: str


class Book(BaseModel):
    title: str = Field(max_length=100, examples=["Harry Potter"])
    authors: list[str] = Field(min_length=1)
    published_date: date | None = Field(default=None, examples=["2008-09-15"])
    image: Image


# user_id is a path parameter
# book is a body parameter
# return_content is a query parameter
@app.post(path="/users/{user_id}/book", status_code=status.HTTP_201_CREATED)
def add_book(user_id: int, book: Annotated[Book, Body()], return_content: bool = False):
    if return_content:
        return {"message": "Book is added", "user_id": user_id, "book": book}
    return {"message": "Book is added", "user_id": user_id}
```

Take a look at [this page](https://fastapi.tiangolo.com/tutorial/extra-data-types/#other-data-types) regarding `datetime` related data conversation.

#### Functions that return classes

Even though `Path`, `Query` and `Body` etc. look like classes, they are in fact **functions** that return classes of the same name.

In addition to `Path`, `Query` and `Body`, there are following functions.

- `Path()`
- `Query()`
- `Header()`
- `Cookie()`
- `Body()`
- `Form()`
- `File()`

> Note 📝
> `Path`, `Query`, `Body` etc. classes are subclasses of a `Param` class, which is itself a subclass of Pydantic's FieldInfo class.

#### Request Bodies of pure lists

If the top level value of the JSON body needs to be a JSON array (a Python list), we can declare the list type in the parameter of the function:

```python
from fastapi import FastAPI
from pydantic import BaseModel, Field, HttpUrl

app = FastAPI()


class Image(BaseModel):
    url: HttpUrl
    name: str = Field(examples=["photo.png"])


@app.post("/images/multiple")
async def create_multiple_images(images: list[Image]):
    return images
```

Expected Body JSON example:

```JSON
[
  {
    "url": "https://example.com/",
    "name": "string"
  }
]
```

It is also possible to declare [a request body of arbitrary dicts](https://fastapi.tiangolo.com/tutorial/body-nested-models/#bodies-of-arbitrary-dicts). But I cannot come up with good use case for it though.

### Cookie parameters

We can define `Cookie` parameters the same way we define `Query` and `Path` parameters.

`Cookie` example:

```python
from typing import Annotated

from fastapi import Cookie, FastAPI

app = FastAPI()


@app.get("/items/")
async def read_items(my_cookie: Annotated[str | None, Cookie()] = None):
    return {"my_cookie": my_cookie}
```

#### Using Pydantic Model

If we have a group of cookies that are related, we can create a Pydantic model to declare them.

```python
from typing import Annotated

from fastapi import Cookie, FastAPI
from pydantic import BaseModel

app = FastAPI()


class Cookies(BaseModel):
    session_id: str
    some_tracker: str | None = None
    other_tracker: str | None = None


@app.get("/items/")
async def read_items(cookies: Annotated[Cookies, Cookie()]):
    return cookies
```

### Header parameters

We can define `Header` parameters the same way we define `Query` and `Path` parameters.

> **Note📝**
> Even though most HTTP headers use hyphens and capitalization, in FastAPI, it is possible to use underscores instead of hyphens and case-insensitive variable names. For example: to represent `User-Agent`, use `user_agent`. That's because using hyphen in variable name is not allowed in python.

`Header` example:

```python
from typing import Annotated

from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    return {"User-Agent": user_agent}
```

#### Duplicate headers

It is possible to receive duplicate headers, i.e., the same header with multiple values. In that case, use a list in the type declaration.

```python
from typing import Annotated

from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(x_token: Annotated[list[str] | None, Header()] = None):
    return {"X-Token values": x_token}
```

#### Header Parameter Model

If we have a group of related header parameters, we can create a Pydantic model to declare them.

```python
from typing import Annotated

from fastapi import FastAPI, Header
from pydantic import BaseModel

app = FastAPI()


class CommonHeaders(BaseModel):
    host: str
    save_data: bool
    if_modified_since: str | None = None
    traceparent: str | None = None
    x_tag: list[str] = []


@app.get("/items/")
async def read_items(headers: Annotated[CommonHeaders, Header()]):
    return headers
```

### Form Data

When we need to receive form fields instead of JSON, we can use `Form`.

> `Form` is a class that inherits directly from `Body`.

Singular types:

```python
from typing import Annotated

from fastapi import FastAPI, Form  # <=====

app = FastAPI()


@app.post("/login/")
async def login(username: Annotated[str, Form()], password: Annotated[str, Form()]):
    return {"username": username}
```

Pydantic Model types:

```python
from typing import Annotated

from fastapi import FastAPI, Form
from pydantic import BaseModel

app = FastAPI()


class FormData(BaseModel):
    username: str
    password: str


@app.post("/login/")
async def login(data: Annotated[FormData, Form()]):
    return data
```

The [OAuth2](https://datatracker.ietf.org/wg/oauth/about/) specification requires the fields to be exactly named `username` and `password`, and to be sent as form fields, not JSON.

> Note 📝
> Since the request data is encoded either as a form or JSON, only `Form` or `Body` can be used at a time as path operation function parameters (not both at the same time).

### Request Files

We can define files to be uploaded by the client using `UploadFile`.

```python
from fastapi import FastAPI, UploadFile, status
from pydantic import BaseModel, Field

app = FastAPI()


class FileMetadata(BaseModel):
    filename: str
    content_type: str = Field(alias="content-type")


@app.post(path="/files", status_code=status.HTTP_200_OK)
def create_user(m_file: UploadFile) -> FileMetadata:

    return {"filename": m_file.filename, "content-type": m_file.content_type}
```

Features of `UploadFile`:

- It uses a "spooled" file:
  - A file stored in memory up to a maximum size limit, and after passing this limit it will be stored on disk.
  - This means that it will work well for large files like images, videos, large binaries, etc. without consuming all the memory.
- It has a [file-like](https://docs.python.org/3/glossary.html#term-file-like-object) `async` interface. (It supports Async `write`, `read` etc.)
- we can get metadata from the uploaded file.
- It exposes an actual Python [SpooledTemporaryFile](https://docs.python.org/3/library/tempfile.html#tempfile.SpooledTemporaryFile) object that we can pass directly to other libraries that expect a file-like object.

### Request Files and Form Data

Needless to say, it is possible to use form data along with file uploads.

```python
from typing import Annotated

from fastapi import FastAPI, Form, UploadFile, status
from pydantic import BaseModel, Field

app = FastAPI()


class FileMetadata(BaseModel):
    owner: str
    filename: str
    content_type: str = Field(alias="content-type")


@app.post(path="/files", status_code=status.HTTP_200_OK)
def create_user(username: Annotated[str, Form()], m_file: UploadFile) -> FileMetadata:

    return {
        "filename": m_file.filename,
        "content-type": m_file.content_type,
        "owner": username,
    }
```

## Response Model

It is possible to define what the path operation returns using both `response_model` parameter in the decorator and *return type* annotation.

If both return type and `response_model` parameter are declared, `response_model` takes priority.

In either case, FastAPI validates the objects being returned with Pydantic.

**Prefer return type over `response_model` for better editor support !**

`response_model` example:

```python
from typing import Annotated, Any

from fastapi import FastAPI, Path
from pydantic import BaseModel, Field

app = FastAPI()


class User(BaseModel):
    id: int
    name: str
    age: int = Field(ge=18)


@app.get(path="/users/{user_id}", response_model=User)  # <======
def get_user(user_id: Annotated[int, Path(ge=0)]) -> Any:
    return User(id=user_id, name="John", age=21)
```

return type annotation:

```python
from typing import Annotated

from fastapi import FastAPI, Path
from pydantic import BaseModel, Field

app = FastAPI()


class User(BaseModel):
    id: int
    name: str
    age: int = Field(ge=18)


@app.get(path="/users/{user_id}")
def get_user(user_id: Annotated[int, Path(ge=0)]) -> User:  # <======
    return User(id=user_id, name="John", age=21)
```

### Automatic Pydantic Validation

Regardless of the data type (singular data types like `int` or pydantic models), FastAPI validates the output data using pydantic.

If the data being returned is dictionary and the type hint is pydantic model, FastAPI can handle it gracefully by calling the pydantic model with the dictionary.

```python
from typing import Annotated

from fastapi import FastAPI, Path
from pydantic import BaseModel, Field

app = FastAPI()


class User(BaseModel):
    id: int
    name: str
    age: int = Field(ge=18)


@app.get(path="/users/{user_id}")
def get_user(user_id: Annotated[int, Path(ge=0)]) -> User:
    return {"id": user_id, "name": "Jane", "age": 12}
    # throws error because age is less than 18
```

### User Separate Pydantic Models

Oftentimes Request body model and response model have many common proprieties. For that reason, it is a good idea create them from a common parent like below:

```python
from pydantic import BaseModel


class UserBase(BaseModel):
    name: str
    surname: str
    age: int


# This is used to validate request body data
class UserRequest(UserBase):
    password: str


# This is used to validate response data
class UserResponse(UserBase):
    id: int
```

## Response Status Codes

Always declare status code for intended successful completion of the path operation.

Check out [HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status) list for the meaning of each status code.

```python
from fastapi import FastAPI, status  # <==========
from pydantic import BaseModel, Field

app = FastAPI()


class UserBase(BaseModel):
    name: str = Field(min_length=1, max_length=50)
    age: int = Field(ge=18)


class UserRequest(UserBase):
    password: str = Field(min_length=8, max_length=20)


class UserResponse(UserBase):
    id: int


@app.post(path="/users", status_code=status.HTTP_201_CREATED)  # <========
def create_user(new_user: UserRequest) -> UserResponse:
    return {"id": 12, "name": new_user.name, "age": new_user.age}
```

## Handling and Returning Errors

When request has invalid data, for example `str` instead of `int`, FastAPI catches it during request validation and returns `422 Validation Error` response.

Try below path with wrong parameter:

```python
from fastapi import FastAPI, status

app = FastAPI()


@app.get(path="/books/{id}", status_code=status.HTTP_200_OK)
def create_user(id: int) -> str:

    return "succeeded"


# request to /books/12 works but
# request to /books/twelve returns 422 Validation Error
```

### How It works

The error handling mechanism has two parts:

- The path operation **raises** exception (implicitly or explicitly)
- Exception handlers catch the exception and returns JSON Response to the client

FastAPI has two built-in exception handlers:

- `request_validation_exception_handler`: handles `RequestValidationError` exceptions. When a **request** contains invalid data, FastAPI internally raises a `RequestValidationError`.
- `http_exception_handler`: handles `HTTPException` exceptions. Path operation function should explicitly raise `HTTPException` to return HTTP responses with errors such as `404 Not Found`.

> Note:
> Both handlers returns JSON response.

In the following example the path operation function raises `HTTPException` with status code of `404`:

```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel

app = FastAPI()

items = {
    1: "item_1",
    2: "item_2",
}


@app.get(path="/success/{id}", status_code=status.HTTP_200_OK)
def create_user(id: int) -> str:

    if items.get(id):
        return "item exists"

    raise HTTPException(  # <======
        status_code=status.HTTP_404_NOT_FOUND,
        detail="the item with given id is not found",
    )
```

`http_exception_handler` catches this exception and returns JSON data.

### Exception handlers

Exception handlers are functions with three characteristics:

1. they are decorated with `app.exception_handler` function
1. the function takes "request" and "exception" as arguments
1. the function returns response which will be sent to the client

```python
@app.exception_handler(ExceptionClass)
async def example_exception_handler(request: Request, exc: ExceptionClass):
    return JSONResponse(
        status_code=418,
        content={"message": "Something went wrong. Try again later."},
    )
```

### Overriding Default Exception Handlers

We can override default exception handlers by handling the same exceptions.

> Override default exception handlers is especially useful for logging purposes.

Overriding `http_exception_handler`:

```python
from logging import DEBUG, basicConfig, getLogger

from fastapi import FastAPI, HTTPException, status
from fastapi.exception_handlers import http_exception_handler
from starlette.exceptions import HTTPException as StarletteHTTPException

logger = getLogger(name=__name__)
basicConfig(level=DEBUG, filename="my_app.log")

app = FastAPI()

# Both FastAPI and Starlette have `HTTPException`.
# FastAPI's `HTTPException` subclasses that of Starlette's.
# To catch exceptions raised by both FastAPI path operations and
# Starlette's internal functions, we should register
# Starlette's `HTTPException` with the handler.
@app.exception_handler(StarletteHTTPException)
async def custom_http_exception_handler(request, exc: StarletteHTTPException):
    logger.error(msg=f"HTTPException: {exc.status_code}.\n\tDetail: {exc.detail}")
    return await http_exception_handler(request, exc)


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    if item_id == 7:
        return {"item_id": item_id}
    raise HTTPException(
        status_code=status.HTTP_404_NOT_FOUND, detail="Item was not found"
    )
```

> Note:
> Both FastAPI and Starlette have `HTTPException`. FastAPI's `HTTPException` subclasses that of Starlette's. To **catch** exceptions raised by both FastAPI path operations and Starlette's internal functions, we should register Starlette's `HTTPException` with the handler.

### Creating Custom Exception Handlers

We can raise any exception we want. We can even create our own exception and raise it. However, FastAPI's built-in exception handlers catch only `HTTPException` and `RequestValidationError` exceptions. To handle other exceptions, we should create custom exception handlers.

Example: Creating an exception handler to catch `NotImplementedError` exceptions

```python
from fastapi import FastAPI, Request, status
from fastapi.responses import JSONResponse

app = FastAPI()


@app.exception_handler(NotImplementedError)
async def not_implemented_exception_handler(request: Request, exc: NotImplementedError):
    return JSONResponse(
        status_code=status.HTTP_425_TOO_EARLY,
        content={
            "message": "This feature has not been implemented yet. Please, check back later."
        },
    )


@app.get("/new-feature/")
async def read_unicorn():
    raise NotImplementedError
```
