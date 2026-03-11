---
name: cakephp-architecture
description: "CakePHP architecture principles and patterns as applied in this project. Covers Controller/Service/Table/Entity layering, naming conventions, response structure, enums, authentication, and logging. Consumed by cakephp-developer and solution-architect."
---

# CakePHP Architecture — Project Conventions

## Core Philosophy

### KISS — Keep It Simple

- Always start with the simplest possible approach
- Don't add features, configurability, or abstractions beyond what was asked
- A simple solution that works is better than an elegant solution that's over-built

### DRY — But Not Prematurely

- Extract into functions/classes ONLY when something is used MORE than once
- Three similar lines of code is better than a premature abstraction
- Don't create helpers, utilities, or base classes for one-time operations
- Don't design for hypothetical future requirements

### No Over-Engineering

- Only make changes directly requested or clearly necessary
- Don't add error handling for scenarios that can't happen
- Don't add feature flags or backwards-compatibility shims — just change the code
- Don't add docstrings, comments, or type annotations to code you didn't change
- Only add comments where logic isn't self-evident

## Architecture: What Goes Where

### Controller — Thin Orchestrator

Controllers handle ONLY:

1. `$this->request->allowMethod()` — restrict HTTP method
2. `$this->Authentication->getIdentity()` — get current user
3. `$this->request->getData()` — get request payload
4. Call service method
5. Catch exceptions and return formatted response

Controllers MUST NOT contain business logic, database queries, or data transformation.

### Service — Business Logic (`src/Service/`)

Services handle:

1. Validate business rules (not input format — that's Table validation)
2. Orchestrate operations across multiple tables
3. Perform calculations and transformations
4. Throw specific exceptions on failure:
    - `InvalidArgumentException` — validation/input errors
    - `RuntimeException` — operational failures (not found, state conflicts)

### Table — Data Layer (`src/Model/Table/`)

Tables handle:

1. `validationDefault()` — input validation rules
2. `buildRules()` — application-level rules (unique, exists)
3. Custom finders (`findByStatus`, `findActive`)
4. `beforeMarshal()` — normalize data before validation
5. Schema type mapping (enums, JSON columns)
6. Association definitions (belongsTo, hasMany)

### Entity — Data Object (`src/Model/Entity/`)

Entities handle:

1. `$_accessible` — mass assignment protection
2. Accessor methods (`_getFieldName`) — format on read
3. Mutator methods (`_setFieldName`) — validate/transform on write
4. Helper methods for the entity's own data only

### Enum — Value Objects (`src/Model/Enum/`)

- String-backed PHP enums
- UPPERCASE case names: `case ACTIVE = 'active';`
- Class naming: `{Entity}{Column}` (e.g., `UserStatus`, `OrderType`)
- Implements `EnumLabelInterface` when used in forms
- Registered in Table via `EnumType::from()`

## Naming Conventions

| Element    | Convention            | Example                                 |
| ---------- | --------------------- | --------------------------------------- |
| Variables  | snake_case            | `$user_id`, `$order_data`               |
| Methods    | camelCase             | `createSupplierOrder()`, `findActive()` |
| Classes    | PascalCase            | `ProductLifeCycleService`, `UsersTable` |
| Migrations | CamelCase descriptive | `AddApiTokenToUsersTable`               |
| Enum cases | UPPERCASE             | `case ACTIVE = 'active'`                |
| DB fields  | snake_case            | `created_at`, `user_id`                 |
| Routes     | kebab-case            | `/inventory-transfers`                  |

## Response Structure

### Success Response

```json
{
	"success": true,
	"message": "Resource created successfully",
	"data": { "id": 1 }
}
```

### Error Response

```json
{
	"success": false,
	"message": "Specific error description",
	"error_code": "VALIDATION_ERROR"
}
```

### HTTP Status Codes

| Code | When                   | Exception Type             | Error Code         |
| ---- | ---------------------- | -------------------------- | ------------------ |
| 200  | Successful read/update | —                          | —                  |
| 201  | Successful create      | —                          | —                  |
| 400  | Invalid input          | `InvalidArgumentException` | `VALIDATION_ERROR` |
| 500  | Operational failure    | `RuntimeException`         | `RUNTIME_ERROR`    |
| 500  | Unexpected error       | `Exception`                | `INTERNAL_ERROR`   |

## Authentication

- MUST use CakePHP Authentication plugin at middleware level
- Never check auth manually in controllers
- Access user via `$this->Authentication->getIdentity()`

## Logging

- Placeholder syntax: `Log::error('Failed for user={user_id}', ['user_id' => $id])`
- Include entity identifiers and business context
- Levels: error (failures), warning (degraded), info (operations), debug (development)

## Migration Standards

### Required Structure

- MUST use `up()` and `down()` methods (NEVER use `change()`)
- MUST include `'comment'` parameter on every field
- MUST use named foreign key constraints: `'constraint' => 'fk_{table}_{field}'`
- MUST use named indexes: `'name' => 'idx_{table}_{field}'`
- MUST provide rollback in `down()` methods
- Use `docker exec [container_name] bin/cake bake migration` to scaffold

### Migration Template

```php
<?php
declare(strict_types=1);

use Migrations\BaseMigration;

class CreateExampleTable extends BaseMigration
{
    public function up(): void
    {
        $table = $this->table('example');
        $table->addColumn('name', 'string', [
            'limit' => 255,
            'null' => false,
            'comment' => 'Example name field'
        ]);
        $table->addColumn('user_id', 'integer', [
            'signed' => false,
            'null' => false,
            'comment' => 'Foreign key to users table'
        ]);
        $table->addForeignKey('user_id', 'users', 'id', [
            'delete' => 'CASCADE',
            'update' => 'CASCADE',
            'constraint' => 'fk_example_user_id'
        ]);
        $table->addIndex('user_id', ['name' => 'idx_example_user_id']);
        $table->create();
    }

    public function down(): void
    {
        $this->table('example')->drop()->save();
    }
}
```

### Model Baking After Migrations

- Always bake models after schema changes: `docker exec [container_name] bin/cake bake model`
- Preserve existing custom methods, associations, and validation rules
- Verify migration status: `bin/cake migrations status`
