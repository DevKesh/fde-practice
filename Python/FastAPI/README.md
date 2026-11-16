# FastAPI

FastAPI is a Python framework for building web APIs. It uses standard Python
type hints to validate input, convert data, and describe your API. It is built
on Starlette and Pydantic, and generates interactive API documentation
automatically.

## Why use FastAPI?

- **Type-based validation:** Declare the data your endpoint expects with Python
  types and Pydantic models.
- **Automatic documentation:** FastAPI creates OpenAPI documentation and
  interactive Swagger UI and ReDoc pages.
- **Async support:** Use `async def` for asynchronous endpoints when your
  application needs it. Regular `def` endpoints are supported too.
- **Editor-friendly:** Type hints make code easier to understand and help
  editors provide completion and diagnostics.

## Getting started

Use Python and create a virtual environment in this folder:

```bash
python -m venv .venv
```

Activate it:

```bash
# macOS or Linux
source .venv/bin/activate

# Windows PowerShell
.venv\Scripts\Activate.ps1
```

Install FastAPI and its standard command-line server tools:

```bash
python -m pip install "fastapi[standard]"
```

Create a file named `main.py`:

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI(title="Books API")


class Book(BaseModel):
    title: str
    author: str
    year: int | None = None


books: dict[int, Book] = {}


@app.get("/")
def read_root():
    return {"message": "Welcome to the Books API"}


@app.get("/books/{book_id}", response_model=Book)
def read_book(book_id: int):
    book = books.get(book_id)
    if book is None:
        raise HTTPException(status_code=404, detail="Book not found")
    return book


@app.post("/books/{book_id}", response_model=Book, status_code=201)
def create_book(book_id: int, book: Book):
    books[book_id] = book
    return book
```

Start the development server from the folder containing `main.py`:

```bash
fastapi dev main.py
```

The server prints its local address, usually `http://127.0.0.1:8000`. Open
these pages in your browser:

- `http://127.0.0.1:8000/docs` — interactive Swagger UI
- `http://127.0.0.1:8000/redoc` — ReDoc API reference
- `http://127.0.0.1:8000/openapi.json` — generated OpenAPI schema

The example stores books in memory, so its data is reset when the application
restarts. Use a database for persistent application data.

## How the example works

- `FastAPI()` creates the application.
- `@app.get(...)` and `@app.post(...)` register functions as HTTP endpoints.
- `{book_id}` is a **path parameter**. The `int` annotation converts and
  validates it as an integer.
- `book: Book` is a **request body**. FastAPI parses the JSON body and
  validates it against the Pydantic model.
- `response_model=Book` documents and validates the response shape.
- `HTTPException` returns an HTTP error response, here a `404 Not Found`.

For example, create a book by sending a `POST` request to
`/books/1` with this JSON body:

```json
{
  "title": "The Hobbit",
  "author": "J. R. R. Tolkien",
  "year": 1937
}
```

Then request `GET /books/1` to retrieve it.

## Path, query, and body parameters

FastAPI uses endpoint function signatures to identify request data:

- **Path parameters** are part of the URL, such as `/books/{book_id}`.
- **Query parameters** follow `?` in the URL, such as `/books?limit=10`.
  Declare them as function parameters not included in the route path.
- **Request bodies** are commonly represented by Pydantic models, as in the
  `create_book` endpoint.

The parameter's Python type and default value help FastAPI validate the request
and describe it in the API documentation.

## Development and deployment

`fastapi dev` is intended for local development and enables automatic reload.
For a production deployment, follow the
[FastAPI deployment guide](https://fastapi.tiangolo.com/deployment/) and use
the deployment command and server configuration appropriate for your hosting
environment. Do not rely on the in-memory example store for production data.

## Learn more

- [Official FastAPI tutorial](https://fastapi.tiangolo.com/tutorial/)
- [FastAPI user guide](https://fastapi.tiangolo.com/)
- [Pydantic documentation](https://docs.pydantic.dev/)
