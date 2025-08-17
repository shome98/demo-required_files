# MongodbService Class Documentation

## Index

* [Purpose and Structure](#purpose-and-structure)
* [Key Components](#key-components)
* [Method Breakdown](#method-breakdown)

  * [create(data)](#create)
  * [findAll()](#findall)
  * [findById(id)](#findbyid)
  * [update(id, data)](#update)
  * [delete(id)](#delete)
* [Overall Analysis](#overall-analysis)
* [Code Reference](#code-reference)

---

## Purpose and Structure

The `MongodbService` is a **generic class** that abstracts common CRUD operations for MongoDB using **Mongoose**.
With `<T extends BaseDocument>`, it’s reusable across models while keeping strong typing.

---

## Key Components

* **BaseDocument Interface**
  Extends Mongoose `Document` and adds an `id` string for consistent ID access.

* **`model` (protected)**
  Type `Model<T>`. The specific Mongoose model instance (e.g., `UserModel`, `ProductModel`).

* **`entityName` (protected)**
  String used in error messages. Defaults to `"Resource"`; customize per entity (e.g., `"User"`).

---

## Method Breakdown

### create

Creates a new document.

```ts
async create(data: Omit<T, keyof BaseDocument>): Promise<T> {
  try {
    const document = await this.model.create({ ...data });
    return document;
  } catch (error) {
    throw new Error(
      `Failed to create ${this.entityName}: ${
        error instanceof Error ? error.message : String(error)
      }`
    );
  }
}
```

**Notes**

* Excludes `_id` and `__v` (auto-managed by MongoDB).
* Clear, contextual error handling.

---

### findAll

Gets all documents from the collection.

```ts
async findAll(): Promise<T[]> {
  return this.model.find();
}
```

**Notes**

* No parameters; great for listings.

---

### findById

Finds a single document by ID.

```ts
async findById(id: string): Promise<T> {
  const doc = await this.model.findOne({ _id: id });
  if (!doc) {
    throw new Error(`${this.entityName} with ID ${id} not found`);
  }
  return doc;
}
```

**Notes**

* Uses `findOne({ _id })` for explicit querying.
* Throws descriptive “not found” errors.

---

### update

Updates fields of a document by ID.

```ts
async update(
  id: string,
  data: Partial<Omit<T, keyof BaseDocument>>
): Promise<T> {
  const doc = await this.model.findOneAndUpdate(
    { _id: id },
    { $set: data as Record<string, any> },
    { new: true, runValidators: true }
  );

  if (!doc) {
    throw new Error(`Failed to update ${this.entityName} with ID ${id}.`);
  }
  return doc;
}
```

**Notes**

* Accepts partial updates while preventing `id/_id` edits.
* `{ new: true }` returns the updated document.
* `{ runValidators: true }` enforces schema rules on updates.

---

### delete

Deletes a document by ID.

```ts
async delete(id: string): Promise<void> {
  const result = await this.model.deleteOne({ _id: id });
  if (result.deletedCount === 0) {
    throw new Error(`${this.entityName} with ID ${id} not found`);
  }
}
```

**Notes**

* Checks `deletedCount` to confirm a deletion occurred.

---

## Overall Analysis

* **Reusable & type-safe** via generics.
* **Clear error messages** aid debugging.
* **Safe updates** (no `_id` changes) with validation.
* A solid base for consistent, scalable data services.

---

## Code Reference

```ts
import { Model, Document } from "mongoose";

export interface BaseDocument extends Document {
  id: string;
}

export class MongodbService<T extends BaseDocument> {
  protected model: Model<T>;
  protected entityName: string;

  constructor(model: Model<T>, entityName = "Resource") {
    this.model = model;
    this.entityName = entityName;
  }

  async create(data: Omit<T, keyof BaseDocument>): Promise<T> {
    try {
      const document = await this.model.create({ ...data });
      return document;
    } catch (error) {
      throw new Error(
        `Failed to create ${this.entityName}: ${
          error instanceof Error ? error.message : String(error)
        }`
      );
    }
  }

  async findAll(): Promise<T[]> {
    return this.model.find();
  }

  async findById(id: string): Promise<T> {
    const doc = await this.model.findOne({ _id: id });
    if (!doc) {
      throw new Error(`${this.entityName} with ID ${id} not found`);
    }
    return doc;
  }

  async update(
    id: string,
    data: Partial<Omit<T, keyof BaseDocument>>
  ): Promise<T> {
    const doc = await this.model.findOneAndUpdate(
      { _id: id },
      { $set: data as Record<string, any> },
      { new: true, runValidators: true }
    );

    if (!doc) {
      throw new Error(`Failed to update ${this.entityName} with ID ${id}.`);
    }
    return doc;
  }

  async delete(id: string): Promise<void> {
    const result = await this.model.deleteOne({ _id: id });
    if (result.deletedCount === 0) {
      throw new Error(`${this.entityName} with ID ${id} not found`);
    }
  }
}
```
