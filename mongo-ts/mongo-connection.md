to establish mongodb connection

in types need to use a mongoose in case of deploying it to edge 
```ts
import { Connection } from "mongoose";

declare global {
  var mongoose: {
    conn: Connection | null;
    promise: Promise<Connection> | null;
  };
  namespace Express {
    interface Request {
      userId?: string;
      accessToken?: string;
    }
  }
}
export {};
```
basic connection files
```ts
import mongoose, { Connection } from "mongoose";
import dotenv from "dotenv";
import config from "../environment/config";

dotenv.config();
const MONGODB_URI = config.MONGODB_URI;

if (!MONGODB_URI) {
  throw new Error(
    "Please define the MONGODB_URI environment variable inside .env"
  );
}

let cached = global.mongoose;

if (!cached) {
  cached = global.mongoose = { conn: null, promise: null };
}

export default async function connectToDatabase(): Promise<Connection> {
  if (cached.conn) {
    console.log("✅ Using existing MongoDB connection");
    return cached.conn;
  }

  if (!cached.promise) {
    console.log("🔄 Creating a new MongoDB connection...");
    const opts = {
      bufferCommands: true,
      maxPoolSize: 10,
    };

    cached.promise = mongoose
      .connect(MONGODB_URI!, opts)
      .then((mongoose) => mongoose.connection);
  }

  try {
    cached.conn = await cached.promise;
    console.log("✅ MongoDB connected!");
  } catch (error) {
    cached.promise = null;
    console.error("❌ MongoDB connection failed:", error);
    throw error;
  }
  return cached.conn;
}
```
