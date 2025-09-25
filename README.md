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

- IDataSource — A minimal interface that abstracts persistence. Barra includes ready-to-use implementations for PostgreSQL and MySQL, and you can also provide your own.

- Controllers — Generic controllers you embed in your code:

   - CrudController — CRUD for any model

   - JwtController — Issues JSON Web Tokens

   - PinController — Email-based PINs for password resets / onboarding

- Filters — Query parameters like *?eq=status|active&like=name|%john%&in=role|admin,officer* are automatically converted into SQL conditions. See the operator reference below.

- Dig (Relations) — Load related data by marking fields with tags. See example below.

- Minimal Coupling — Your app handles routing. Barra focuses on request/response handling, input binding, query decoding, and talking to an IDataSource.



## Example: Controller
```
type SongController struct {
    crud ctrl.CrudController
}

func (bc SongController) Fetch(w http.ResponseWriter, r *http.Request) {
    bc.crud.Fetch(w, r, new(model.Song))
}
```

## Example: Model
```
type Song struct {
	mysql.Table
	SongID       int64     `db:"song_id"          json:"song_id"                  pk:"1"`
	Title        string    `db:"title"            json:"title"`
	AlbumID      int64     `db:"album_id"         json:"album_id"`
	Album        *Album    `db:"-"                json:"album,omitempty"          fk:"album_id"`
}
```

## Example: Fetch

*GET /songs?eq=album_id|2&dig=album&limit=2* will return:

```
{
    song_id: 1,
    title: "Blues Deluxe",
    album_id: 2,
    album: {
        album_id: 2,
        title: "Truth"
    }
},
{
    song_id: 2,
    title: "Beck Bolero",
    album_id: 2,
    album: {
        album_id: 2,
        title: "Truth"
    }
}
```

With tags in your model, you get filtering, pagination, dig relations, validation, and JSON responses automatically.

## Filters — Operator Reference

|    Operator   | Example                       | Meaning                      |
| ------------: | ----------------------------- | ---------------------------- |
|          `eq` | `eq=status\|active`           | Equals                       |
|       `noteq` | `noteq=status\|inactive`      | Not equal                    |
|        `like` | `like=name\|%john%`           | SQL LIKE (pattern match)     |
|     `notlike` | `notlike=name\|%john%`        | SQL NOT LIKE (pattern match) |
|          `gt` | `gt=age\|18`                  | Greater than                 |
|          `lt` | `lt=age\|18`                  | Less than                    |
|        `gteq` | `gteq=created_at\|2024-01-01` | Greater or equal             |
|        `lteq` | `lteq=created_at\|2024-12-31` | Less or equal                |
|          `in` | `in=role\|admin,user`         | IN list                      |
|       `notin` | `notin=role\|guest,test`      | NOT IN list                  |
|      `isnull` | `isnull=deleted_at`           | Is NULL check                |
|   `isnotnull` | `isnotnull=deleted_at`        | Is NOT NULL check            |
|       `limit` | `limit=50`                    | SQL Limit                    |
|      `offset` | `offset=50`                   | SQL Limit,OFFSET             |
|       `order` | `order=artist_id\|DESC`       | SQL Order                    |

- Multiple operators of the same type can be set in a given query

## Why Barra?

- Solid foundation — built directly on Go’s standard library
- Extensible — swap databases, add custom controllers, extend filters
- Convention over configuration — minimal setup, sensible defaults
- Not a framework — you stay in control of routing and app structure

## Status

Barra is experimental. The API may evolve as core interfaces are refined.
