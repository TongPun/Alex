# Next.js API: Connect to MySQL Database

## Installing mysql2 library
To be able to connect to your MySQL server and execute queries, you will need to install mysql2 library. To do that, u can either use npm or yarn.

npm install --save mysql2

If you are using TypeScript you will need to install @types/node.

npm install --save-dev @types/node


After installation, you should be able to see them in package.json file.

## Creating a Singleton Connection Pool
Create lit/db.ts for db configuration variables in .env

import mysql from 'mysql2/promise';

const pool = mysql.createPool({
host:             process.env.MYSQL_HOST,
port:             Number(process.env.MYSQL_PORT ?? 3306),
user:             process.env.MYSQL_USER,
password:         process.env.MYSQL_PASSWORD,
database:         process.env.MYSQL_DATABASE,
connectionLimit:  10,
waitForConnections: true,
queueLimit:       0,
});

export default pool;

## Environment Variables

MYSQL_HOST=127.0.0.1
MYSQL_PORT=3306
MYSQL_USER=app_user
MYSQL_PASSWORD=strongpassword
MYSQL_DATABASE=app_db

## API Route Example

// app/api/users/route.ts
import { NextResponse } from 'next/server';
import pool from '@/lib/db';

export async function GET() {
const [rows] = await pool.query('SELECT id, name, email FROM users LIMIT 20');
return NextResponse.json(rows);
}
