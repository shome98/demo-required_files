# MongodbService with Pagination

## Index

- [Purpose and Structure](#purpose-and-structure)
- [Key Interfaces](#key-interfaces)
  - [BaseDocument](#basedocument)
  - [PaginatedResult](#paginatedresult)
- [Class: MongodbService](#class-mongodbservice)
  - [Constructor](#constructor)
  - [paginate(query, page, limit, sort, projection)](#paginate)
- [Code Reference](#code-reference)


## Purpose and Structure

This version of `MongodbService` introduces **pagination support** on top of the base CRUD features.
It allows querying large collections efficiently by fetching documents in **pages** with sorting and projections.

---

## Key Interfaces

### BaseDocument

Extends Mongoose’s `Document` to ensure every entity has a string `id`.

```ts
export interface BaseDocument extends Document {
  id: string;
}
```

---

### PaginatedResult

The return type for the `paginate` method.

```ts
export interface PaginatedResult<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}
```

**Fields**

* `data`: The documents in the current page.
* `total`: Total number of documents matching the query.
* `page`: Current page number.
* `limit`: Number of documents per page.
* `totalPages`: Total number of pages.

---

## Class: MongodbService

### Constructor

```ts
constructor(model: Model<T>, entityName = "Resource") {
  this.model = model;
  this.entityName = entityName;
}
```

* **model**: Mongoose model instance (e.g., `UserModel`).
* **entityName**: Friendly entity name used in error handling/logging (defaults to `"Resource"`).

---

### paginate

Fetches a **paginated list** of documents with optional filtering, sorting, and projection.

```ts
async paginate(
  query: FilterQuery<T> = {},
  page: number = 1,
  limit: number = 10,
  sort: Record<string, SortOrder> = { createdAt: -1 },
  projection?: ProjectionType<T>
): Promise<PaginatedResult<T>> {
  const skip = (page - 1) * limit;
  const [data, total] = await Promise.all([
    this.model.find(query, projection).sort(sort).skip(skip).limit(limit),
    this.model.countDocuments(query),
  ]);

  const totalPages = Math.ceil(total / limit);

  return { data, total, page, limit, totalPages };
}
```

**Parameters**

* `query`: MongoDB filter (default `{}` = all docs).
* `page`: Page number (default `1`).
* `limit`: Items per page (default `10`).
* `sort`: Sort order (default `{ createdAt: -1 }` = newest first).
* `projection`: Fields to include/exclude (optional).

**Returns**

* A `PaginatedResult<T>` object with documents and pagination metadata.

---

## Code Reference

```ts
import { Model, Document, FilterQuery, SortOrder, ProjectionType } from "mongoose";

export interface BaseDocument extends Document {
  id: string;
}

export interface PaginatedResult<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

export class MongodbService<T extends BaseDocument> {
  protected model: Model<T>;
  protected entityName: string;

  constructor(model: Model<T>, entityName = "Resource") {
    this.model = model;
    this.entityName = entityName;
  }

  async paginate(
    query: FilterQuery<T> = {},
    page: number = 1,
    limit: number = 10,
    sort: Record<string, SortOrder> = { createdAt: -1 },
    projection?: ProjectionType<T>
  ): Promise<PaginatedResult<T>> {
    const skip = (page - 1) * limit;
    const [data, total] = await Promise.all([
      this.model.find(query, projection).sort(sort).skip(skip).limit(limit),
      this.model.countDocuments(query),
    ]);

    const totalPages = Math.ceil(total / limit);

    return { data, total, page, limit, totalPages };
  }
}
```

---

