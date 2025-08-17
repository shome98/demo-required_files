# for bulk write operations *some fixes are needed 
```ts
import { Model, Document, Types } from "mongoose";

export interface BaseDocument extends Document {
  id: string;
}
interface BulkWriteResult {
  acknowledged: boolean;
  matchedCount: number;
  modifiedCount: number;
  upsertedCount: number;
  insertedCount: number;
  deletedCount: number;
  upsertedIds: { [key: number]: Types.ObjectId };
}
export class MongodbService<T extends BaseDocument> {
  protected model: Model<T>;
  protected entityName: string;

  constructor(model: Model<T>, entityName = "Resource") {
    this.model = model;
    this.entityName = entityName;
  }

  async updateManyById(
    ids: string[],
    data: Array<Partial<Omit<T, keyof BaseDocument>>>
  ): Promise<{ matchedCount: number; modifiedCount: number }> {
    if (ids.length !== data.length) {
      throw new Error(
        `Ids and object count mismatch. Ids are ${ids.length} objects are ${data.length}`
      );
    }

    const operations = ids.map((id, index) => ({
      updateOne: {
        filter: { _id: new Types.ObjectId(id) },
        update: { $set: data[index] },
        runValidators: true,
      },
    }));

    const result = await this.model.bulkWrite(operations);
    return {
      matchedCount: result.matchedCount,
      modifiedCount: result.modifiedCount,
    };
  }

  /**
   * Bulk insert multiple documents
   */
  async bulkInsert(
    documents: Array<Partial<Omit<T, keyof BaseDocument>>>
  ): Promise<{
    insertedCount: number;
    upsertedIds: { [key: number]: Types.ObjectId };
  }> {
    const operations = documents.map((doc) => ({
      insertOne: {
        document: doc,
      },
    }));

    const result = await this.model.bulkWrite(operations);
    return {
      insertedCount: result.insertedCount,
      upsertedIds: result.upsertedIds,
    };
  }

  /**
   * Bulk delete documents by IDs
   */
  async bulkDelete(ids: string[]): Promise<{ deletedCount: number }> {
    const objectIds = ids.map((id) => new Types.ObjectId(id));

    const operations = [
      {
        deleteMany: {
          filter: { _id: { $in: objectIds } },
        },
      },
    ];

    const result = await this.model.bulkWrite(operations);
    return {
      deletedCount: result.deletedCount,
    };
  }

  /**
   * Bulk upsert (update or insert) documents by ID
   */
  async bulkUpsert(
    ids: (string | null)[],
    data: Array<Partial<Omit<T, keyof BaseDocument>>>
  ): Promise<{ upsertedCount: number; modifiedCount: number }> {
    if (ids.length !== data.length) {
      throw new Error(
        `Ids and object count mismatch. Ids are ${ids.length} objects are ${data.length}`
      );
    }

    const operations = ids.map((id, index) => {
      if (id) {
        // Update existing document
        return {
          updateOne: {
            filter: { _id: id },
            update: { $set: data[index] },
            upsert: true,
            runValidators: true,
          },
        };
      } else {
        // Insert new document (null or undefined ID)
        return {
          updateOne: {
            filter: data[index], // Use data as filter for upsert
            update: { $set: data[index] },
            upsert: true,
            runValidators: true,
          },
        };
      }
    });

    const result = await this.model.bulkWrite(operations);
    return {
      upsertedCount: result.upsertedCount,
      modifiedCount: result.modifiedCount,
    };
  }

  /**
   * Generic bulk write operation for mixed operations
   */
  async bulkWrite(operations: Array<any>): Promise<BulkWriteResult> {
    const result = await this.model.bulkWrite(operations);
    return {
      acknowledged: result.acknowledged,
      matchedCount: result.matchedCount,
      modifiedCount: result.modifiedCount,
      upsertedCount: result.upsertedCount,
      insertedCount: result.insertedCount,
      deletedCount: result.deletedCount,
      upsertedIds: result.upsertedIds,
    };
  }
}

```
