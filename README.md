# Barra

**Barra** is a stdlib-first Go toolkit for building REST APIs with minimal boilerplate.

It focuses on:
- Using Go’s standard library (`net/http`, `context`) as the foundation
- Built-in support for JWT and PIN-based authentication flows
- Generic CRUD controllers for any `IDataSource` implementation
- Ready-to-use `IDataSource` implementations for **PostgreSQL** and **MySQL**
- Rich query layer with filters and **dig** (related data loading)
- Minimal dependencies, clear interfaces

Barra is **not a web framework** — it plugs into whatever routing solution you prefer.

---

## Installation

```bash
go get github.com/zicare/barra
```

## Core Concepts

- IDataSource
A minimal interface that abstracts persistence. Barra includes ready-to-use implementations for PostgreSQL and MySQL, and you can also provide your own.

- Controllers
Generic controllers you embed in your code:

- CrudController — CRUD for any model

- JwtController — issues JSON Web Tokens

- PinController — email-based PINs for password resets / onboarding

- Filters
Query parameters like ?like=name|%john%&eq=status|active are automatically converted into SQL conditions. See the operator reference below.

- Dig (Relations)
Load related data by marking fields with a dig tag (e.g., Album *Album `db:"-" json:"album,omitempty") and calling with GET /songs/1?dig=album.

- Minimal Coupling
Your app handles routing. Barra focuses on request/response handling, input binding, query decoding, and talking to an IDataSource.

## Example: CRUD in 5 lines
```
type SongController struct {
    crud ctrl.CrudController
}

func (bc SongController) Fetch(w http.ResponseWriter, r *http.Request) {
    bc.crud.Fetch(w, r, new(model.Song))
}
```


With tags in your model, you get filtering, pagination, dig relations, validation, and JSON responses automatically.

## Filters — Operator Reference

| Operator | Example                       | Meaning                  |
| -------: | ----------------------------- | ------------------------ |
|     `eq` | `eq=status\|active`           | Equals                   |
|     `ne` | `ne=status\|inactive`         | Not equal                |
|   `like` | `like=name\|%john%`           | SQL LIKE (pattern match) |
|     `gt` | `gt=age\|18`                  | Greater than             |
|     `lt` | `lt=age\|18`                  | Less than                |
|    `gte` | `gte=created_at\|2024-01-01`  | Greater or equal         |
|    `lte` | `lte=created_at\|2024-12-31`  | Less or equal            |
|     `in` | `in=role\|admin,user`         | IN list                  |
|  `notin` | `notin=role\|guest,test`      | NOT IN list              |
| `isnull` | `isnull=deleted_at`           | Is NULL check            |

## Why Barra?

- Solid foundation — built directly on Go’s standard library
- Extensible — swap databases, add custom controllers, extend filters
- Convention over configuration — minimal setup, sensible defaults
- Not a framework — you stay in control of routing and app structure

## Status

Barra is experimental. The API may evolve as core interfaces are refined — feedback welcome.
