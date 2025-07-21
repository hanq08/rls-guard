# RLS Guard

A powerful CLI tool for managing PostgreSQL Row Level Security (RLS) policies as code using TypeScript.

## Features

- 🔒 **Declarative RLS policies** - Define your security policies in TypeScript using a fluent API
- 🚀 **Easy deployment** - Deploy policies to your PostgreSQL database with a single command
- 🔍 **Dry-run support** - Preview SQL commands before executing them
- 🎯 **Type-safe configuration** - Full TypeScript support with intellisense and type checking
- 🏗️ **Built-in helpers** - Common RLS patterns like user isolation, tenant separation, and role-based access
- 🔧 **Cross-platform** - Works on macOS, Linux, and Windows

## Installation

Install globally via npm:

```bash
npm install -g rls-guard
```

## Quick Start

1. **Initialize a new configuration**:
   ```bash
   rls-guard init
   ```

2. **Configure your database and policies** in `rls.config.ts`:
   ```typescript
   import { config, currentUserId, tenantId, publicAccess } from 'rls-guard/lib/rls-config';

   const rlsConfig = config()
     .database(db => db
       .connectionUrl("postgresql://user:pass@localhost:5432/mydb")
     )
     
     // Users can only see their own records
     .addPolicy(p => p
       .name("user_isolation")
       .onTable("users")
       .forCommand("SELECT")
       .withExpression(currentUserId())
       .forRoles("authenticated_user")
     )
     
     // Admin users have full access
     .addPolicy(p => p
       .name("admin_full_access")
       .onTable("users")
       .forCommand("ALL")
       .withExpression(publicAccess())
       .forRoles("admin")
     );

   export default rlsConfig;
   ```

3. **Deploy your policies**:
   ```bash
   # Preview changes
   rls-guard deploy --dry-run
   
   # Apply to database
   rls-guard deploy
   ```

## Configuration

### Database Connection

Connect using a connection URL:
```typescript
.database(db => db
  .connectionUrl("postgresql://user:pass@localhost:5432/mydb?sslmode=disable")
)
```

Or individual parameters:
```typescript
.database(db => db
  .host("localhost")
  .port(5432)
  .database("mydb")
  .username("user")
  .password("pass")
  .ssl(false)
)
```

### Policy Types

**Permissive policies** (default) - Allow access when conditions are met:
```typescript
.addPolicy(p => p
  .name("user_data_access")
  .onTable("user_data")
  .forCommand("SELECT")
  .withExpression(currentUserId())
  .forRoles("user")
  .asPermissive()  // This is the default
)
```

**Restrictive policies** - Block access unless conditions are met:
```typescript
.addPolicy(p => p
  .name("sensitive_data_restriction")
  .onTable("sensitive_data")
  .forCommand("SELECT")
  .withExpression("false")  // Block by default
  .forRoles("public")
  .asRestrictive()
)
```

### Built-in Helper Functions

- `currentUserId(column?)` - Match current user ID
- `tenantId(column?)` - Multi-tenant isolation  
- `recentData(column, days)` - Time-based access
- `ownerOnly(userCol, ownerCol)` - Owner-based access
- `roleCheck(role)` - Role-based conditions
- `publicAccess()` - Always allow (returns `true`)
- `noAccess()` - Always deny (returns `false`)

## Commands

### `rls-guard init`
Create a new `rls.config.ts` file with example policies.

### `rls-guard deploy [options]`
Deploy RLS policies to your PostgreSQL database.

**Options:**
- `--dry-run` - Show SQL commands without executing them
- `--config, -c <path>` - Path to config file (default: `rls.config.ts`)

### `rls-guard version`
Show the current version.

## Requirements

- Node.js 12+
- PostgreSQL 9.5+ (RLS support)
- TypeScript configuration file

## License

MIT License