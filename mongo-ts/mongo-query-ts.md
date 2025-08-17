# MongodbService with Query Utilities

## Index

- [Purpose and Structure](#purpose-and-structure)
- [Key Interfaces](#key-interfaces)
  - [BaseDocument](#basedocument)
- [Class: MongodbService](#class-mongodbservice)
  - [Constructor](#constructor)
  - [updateManyByQuery(query, data)](#updatemanybyquery)
  - [deleteManyByQuery(query)](#deletemanybyquery)
  - [findByQuery(query, projection, options)](#findbyquery)
  - [findOneByQuery(query, projection, options)](#findonebyquery)
  - [checkExistsByQuery(query)](#checkexistsbyquery)
  - [countDocumentsByQuery(query)](#countdocumentsbyquery)
- [Code Reference](#code-reference)

---

## Purpose and Structure

The `MongodbService<T>` class is a generic service for interacting with MongoDB using **Mongoose**.  
It provides utility methods for querying, updating, deleting, and checking existence of documents,  
with a consistent error-handling structure.

---

## Key Interfaces

### BaseDocument
```ts
export interface BaseDocument extends Document {
  id: string;
}

* Extends Mongoose's `Document`.
* Ensures each entity has an `id` field.

---

## Class: MongodbService

### Constructor

```ts
constructor(model: Model<T>, entityName = "Resource")
```

* `model`: The Mongoose model to be used.
* `entityName`: A friendly name for error messages (defaults to `"Resource"`).

---

### updateManyByQuery

```ts
async updateManyByQuery(
  query: FilterQuery<T>,
  data: UpdateQuery<T>
): Promise<{ matchedCount: number; modifiedCount: number }>
```

* Updates multiple documents that match a query.
* Returns the number of matched and modified documents.

---

### deleteManyByQuery

```ts
async deleteManyByQuery(
  query: FilterQuery<T>
): Promise<{ deletedCount: number }>
```

* Deletes multiple documents based on the given query.
* Returns the number of deleted documents.

---

### findByQuery

```ts
async findByQuery(
  query: FilterQuery<T>,
  projection?: ProjectionType<T>,
  options?: QueryOptions
): Promise<T[]>
```

* Finds all documents that match the query.
* Throws an error if no documents are found.

---

### findOneByQuery

```ts
async findOneByQuery(
  query: FilterQuery<T>,
  projection?: ProjectionType<T>,
  options?: QueryOptions
): Promise<T>
```

* Finds a single document based on the query.
* Throws an error if the document does not exist.

---

### checkExistsByQuery

```ts
async checkExistsByQuery(query: FilterQuery<T>): Promise<boolean>
```

* Returns `true` if any document matches the query, otherwise `false`.

---

### countDocumentsByQuery

```ts
async countDocumentsByQuery(query: FilterQuery<T> = {}): Promise<number>
```

* Counts the number of documents matching the query.

---

## Code Reference

```ts
import { 
  Model, Document, Types, 
  FilterQuery, UpdateQuery, 
  ProjectionType, QueryOptions 
} from "mongoose";

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

  async updateManyByQuery(
    query: FilterQuery<T>,
    data: UpdateQuery<T>
  ): Promise<{ matchedCount: number; modifiedCount: number }> {
    const result = await this.model.updateMany(query, data, {
      runValidators: true,
    });
    return {
      matchedCount: result.matchedCount,
      modifiedCount: result.modifiedCount,
    };
  }
    
  async deleteManyByQuery(
    query: FilterQuery<T>
  ): Promise<{ deletedCount: number }> {
    const result = await this.model.deleteMany(query);
    return { deletedCount: result.deletedCount };
  }

  async findByQuery(
    query: FilterQuery<T>,
    projection?: ProjectionType<T>,
    options?: QueryOptions
  ): Promise<T[]> {
    const docs = this.model.find(query, projection, options);
    if (!docs) {
      throw new Error(
        `${this.entityName}s with ${JSON.stringify(query)} not found`
      );
    }
    return docs;
  }

  async findOneByQuery(
    query: FilterQuery<T>,
    projection?: ProjectionType<T>,
    options?: QueryOptions
  ): Promise<T> {
    const doc = await this.model.findOne(query, projection, options);
    if (!doc) {
      throw new Error(
        `${this.entityName} with ${JSON.stringify(query)} not found`
      );
    }
    return doc as T;
  }

  async checkExistsByQuery(query: FilterQuery<T>): Promise<boolean> {
    const count = await this.model.countDocuments(query);
    return count > 0;
  }

  async countDocumentsByQuery(query: FilterQuery<T> = {}): Promise<number> {
    return this.model.countDocuments(query);
  }
}
```

---
