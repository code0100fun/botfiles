---
name: ecto-guidelines
description: Ecto best practices covering preloading, schema types, changeset validation, field access, security, and migration conventions. Use when working with Ecto schemas, changesets, queries, or migrations.
---

# Ecto Guidelines

## Preloading Associations

- **Always** preload Ecto associations in queries when they'll be accessed in templates:

```elixir
# If the template needs message.user.email
messages = Repo.all(from m in Message, preload: [:user])
```

## Seeds and Queries

- Remember `import Ecto.Query` and other supporting modules when you write `seeds.exs`

## Schema Field Types

- `Ecto.Schema` fields always use the `:string` type, even for `:text` columns:

```elixir
field :name, :string
field :description, :string  # Even if the DB column is :text
```

## Changeset Validation

- `Ecto.Changeset.validate_number/2` **DOES NOT SUPPORT the `:allow_nil` option**. By default, Ecto validations only run if a change for the given field exists and the change value is not nil, so such an option is never needed

## Accessing Changeset Fields

- You **must** use `Ecto.Changeset.get_field(changeset, :field)` to access changeset fields
- **Never** use map access syntax (`changeset[:field]`) on changesets

## Security: Programmatic Fields

- Fields which are set programmatically, such as `user_id`, must not be listed in `cast` calls or similar for security purposes. Instead they must be explicitly set when creating the struct:

```elixir
# GOOD: Set user_id explicitly, not through cast
%Message{}
|> Message.changeset(params)  # cast only includes :content, :subject, etc.
|> Ecto.Changeset.put_assoc(:user, user)
```

## Migrations

- **Always** invoke `mix ecto.gen.migration migration_name_using_underscores` when generating migration files, so the correct timestamp and conventions are applied
