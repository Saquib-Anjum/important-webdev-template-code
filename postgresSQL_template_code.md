# PostgreSQL Database Setup

This README provides a complete template for PostgreSQL database connection setup and privilege management.

## Table of Contents
- [Environment Setup](#environment-setup)
- [Database Connection](#database-connection)
- [Privilege Management](#privilege-management)
- [Common Queries](#common-queries)
- [Usage Examples](#usage-examples)
- [Troubleshooting](#troubleshooting)

## Environment Setup

### 1. Create `.env` file in your project root:

```env
# Database Configuration
DB_HOST=localhost
DB_PORT=5432
DB_NAME=your_database_name
DB_USER=your_username
DB_PASSWORD=your_password

# Optional: SSL Configuration
DB_SSL=false
```

### 2. Install Dependencies

```bash
npm install pg dotenv
```

## Database Connection

### Connection Pool Setup (`db/connection.js`)

```javascript
import pkg from "pg";
const { Pool } = pkg;
import dotenv from "dotenv";

dotenv.config();

// Debug environment variables (remove in production)
console.log(`
Database Configuration:
- Host: ${process.env.DB_HOST}
- Port: ${process.env.DB_PORT}
- Database: ${process.env.DB_NAME}
- User: ${process.env.DB_USER}
- Password: ${process.env.DB_PASSWORD ? '***' : 'Not set'}
`);

const pool = new Pool({
  host: process.env.DB_HOST || "localhost",
  port: process.env.DB_PORT || 5432,
  database: process.env.DB_NAME || "postgres",
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  // Optional pool configuration
  max: 20, // Maximum number of clients in pool
  idleTimeoutMillis: 30000, // Close idle clients after 30 seconds
  connectionTimeoutMillis: 2000, // Return error if connection takes longer than 2 seconds
});

// Connection event handlers
pool.on("connect", () => {
  console.log("✅ Connected to PostgreSQL database");
});

pool.on("error", (err) => {
  console.error("❌ Unexpected error on idle client", err);
  process.exit(-1);
});

// Test connection on startup
pool.connect((err, client, release) => {
  if (err) {
    console.error("❌ Error acquiring client", err.stack);
  } else {
    console.log("🔗 Database connection pool initialized");
    release();
  }
});

// Wrapper for queries
export const query = (text, params) => pool.query(text, params);

// Export pool for advanced usage
export { pool };

// Graceful shutdown
process.on('SIGINT', () => {
  console.log('🔄 Gracefully shutting down database connections...');
  pool.end(() => {
    console.log('✅ Database pool has ended');
    process.exit(0);
  });
});
```

## Privilege Management

### User and Database Creation

```sql
-- Create new database
CREATE DATABASE your_database_name;

-- Create new user/role
CREATE USER your_username WITH PASSWORD 'your_password';

-- Grant connection privileges
GRANT CONNECT ON DATABASE your_database_name TO your_username;

-- Grant usage on schema
GRANT USAGE ON SCHEMA public TO your_username;
```

### Table Privileges

```sql
-- Grant all privileges on all tables in schema
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO your_username;

-- Grant privileges on future tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO your_username;

-- Specific table privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON table_name TO your_username;

-- Read-only access
GRANT SELECT ON ALL TABLES IN SCHEMA public TO your_username;
```

### Sequence Privileges

```sql
-- For auto-incrementing IDs - Grant usage and select on sequence
GRANT USAGE, SELECT ON SEQUENCE users_id_seq TO your_username;

-- For updating sequence values
GRANT USAGE, SELECT, UPDATE ON SEQUENCE users_id_seq TO your_username;

-- Full control over sequence
GRANT ALL PRIVILEGES ON SEQUENCE users_id_seq TO your_username;

-- Grant on all sequences
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO your_username;

-- Grant on future sequences
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT USAGE, SELECT ON SEQUENCES TO your_username;
```

### Function and Procedure Privileges

```sql
-- Execute privileges on functions
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO your_username;

-- Grant on future functions
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT EXECUTE ON FUNCTIONS TO your_username;
```

### Schema Privileges

```sql
-- Create objects in schema
GRANT CREATE ON SCHEMA public TO your_username;

-- All privileges on schema
GRANT ALL ON SCHEMA public TO your_username;
```

## Common Queries

### User Management

```sql
-- List all users
SELECT usename FROM pg_user;

-- Check user privileges
SELECT * FROM information_schema.role_table_grants WHERE grantee = 'your_username';

-- Drop user (must revoke privileges first)
REVOKE ALL PRIVILEGES ON DATABASE your_database_name FROM your_username;
DROP USER your_username;

-- Change password
ALTER USER your_username PASSWORD 'new_password';
```

### Database Information

```sql
-- List all databases
SELECT datname FROM pg_database;

-- List all tables
SELECT tablename FROM pg_tables WHERE schemaname = 'public';

-- List all sequences
SELECT sequencename FROM pg_sequences WHERE schemaname = 'public';

-- Check table permissions
SELECT * FROM information_schema.table_privileges WHERE table_schema = 'public';
```

### Performance Queries

```sql
-- Check active connections
SELECT pid, usename, application_name, client_addr, state 
FROM pg_stat_activity 
WHERE state = 'active';

-- Database size
SELECT pg_size_pretty(pg_database_size('your_database_name'));

-- Table sizes
SELECT 
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables 
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

## Usage Examples

### Basic Query Example

```javascript
import { query, pool } from './db/connection.js';

// Simple query
async function getUsers() {
  try {
    const result = await query('SELECT * FROM users');
    return result.rows;
  } catch (error) {
    console.error('Error fetching users:', error);
    throw error;
  }
}

// Parameterized query
async function getUserById(id) {
  try {
    const result = await query('SELECT * FROM users WHERE id = $1', [id]);
    return result.rows[0];
  } catch (error) {
    console.error('Error fetching user:', error);
    throw error;
  }
}

// Transaction example
async function transferFunds(fromAccount, toAccount, amount) {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN');
    
    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
      [amount, fromAccount]
    );
    
    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
      [amount, toAccount]
    );
    
    await client.query('COMMIT');
    console.log('✅ Transaction completed successfully');
  } catch (error) {
    await client.query('ROLLBACK');
    console.error('❌ Transaction failed:', error);
    throw error;
  } finally {
    client.release();
  }
}
```

### Error Handling

```javascript
// Enhanced query wrapper with error handling
export const safeQuery = async (text, params = []) => {
  const client = await pool.connect();
  
  try {
    const start = Date.now();
    const result = await client.query(text, params);
    const duration = Date.now() - start;
    
    console.log(`Query executed in ${duration}ms`);
    return result;
  } catch (error) {
    console.error('Database query error:', {
      query: text,
      params: params,
      error: error.message
    });
    throw error;
  } finally {
    client.release();
  }
};
```

## Troubleshooting

### Common Issues

1. **Connection refused**
   ```bash
   # Check if PostgreSQL is running
   sudo systemctl status postgresql
   
   # Start PostgreSQL
   sudo systemctl start postgresql
   ```

2. **Authentication failed**
   - Verify username/password in `.env`
   - Check `pg_hba.conf` for authentication method
   - Ensure user has connection privileges

3. **Permission denied**
   ```sql
   -- Grant necessary privileges
   GRANT ALL PRIVILEGES ON DATABASE your_database_name TO your_username;
   ```

4. **Too many connections**
   ```sql
   -- Check connection limit
   SHOW max_connections;
   
   -- Kill idle connections
   SELECT pg_terminate_backend(pid) FROM pg_stat_activity 
   WHERE state = 'idle' AND pid <> pg_backend_pid();
   ```

### Environment Variables Checklist

- [ ] `DB_HOST` - Database host (default: localhost)
- [ ] `DB_PORT` - Database port (default: 5432)
- [ ] `DB_NAME` - Database name
- [ ] `DB_USER` - Database username
- [ ] `DB_PASSWORD` - Database password

### Security Best Practices

1. **Never commit `.env` files** - Add to `.gitignore`
2. **Use environment-specific configurations**
3. **Implement connection pooling** - Already included in template
4. **Use parameterized queries** - Prevents SQL injection
5. **Limit user privileges** - Follow principle of least privilege
6. **Enable SSL in production**
7. **Regular backup and monitoring**

---

## Quick Setup Commands

```bash
# 1. Initialize project
npm init -y
npm install pg dotenv

# 2. Create directories
mkdir db config

# 3. Copy connection code to db/connection.js
# 4. Create and configure .env file
# 5. Test connection
node -e "import('./db/connection.js')"
```

---

**Note**: Replace placeholder values (`your_database_name`, `your_username`, etc.) with your actual database credentials and configuration.
