# CakePHP Architecture — Full Reference

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

## HTTP Status Codes

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
