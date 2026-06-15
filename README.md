# Users Table — Documentation

## Overview
The `users` table stores account and profile information for all users of the gym e-commerce platform, including customers, trainers, vendors, and admins. It serves as the central reference table for authentication, role-based access, and basic profile data.

## Schema

```sql
CREATE TABLE users (
    user_id         CHAR(36) PRIMARY KEY DEFAULT (UUID()),
    first_name      VARCHAR(50)  NOT NULL,
    last_name       VARCHAR(50)  NOT NULL,
    email           VARCHAR(150) NOT NULL UNIQUE,
    phone           VARCHAR(20)  UNIQUE,
    password_hash   VARCHAR(255) NOT NULL,
    role            VARCHAR(20)  NOT NULL DEFAULT 'customer'
                        CHECK (role IN ('customer', 'admin', 'trainer', 'vendor')),
    date_of_birth   DATE,
    gender          VARCHAR(10) CHECK (gender IN ('male', 'female', 'other')),
    is_verified     BOOLEAN NOT NULL DEFAULT FALSE,
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    profile_picture VARCHAR(255),
    created_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at      TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    last_login_at   TIMESTAMP NULL
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role  ON users(role);
```

## Column Reference

| Column            | Type           | Constraints / Default                     | Description |
|-------------------|----------------|--------------------------------------------|-------------|
| `user_id`         | CHAR(36)       | PRIMARY KEY, DEFAULT `UUID()`               | Unique identifier for each user (UUID format). |
| `first_name`      | VARCHAR(50)    | NOT NULL                                    | User's first name. |
| `last_name`       | VARCHAR(50)    | NOT NULL                                    | User's last name. |
| `email`           | VARCHAR(150)   | NOT NULL, UNIQUE                            | Primary login identifier; must be unique across all users. |
| `phone`           | VARCHAR(20)    | UNIQUE                                       | Contact number; optional but must be unique if provided. |
| `password_hash`   | VARCHAR(255)   | NOT NULL                                    | Hashed password (e.g., bcrypt). Never store plain text. |
| `role`            | VARCHAR(20)    | NOT NULL, DEFAULT `'customer'`, CHECK       | Defines access level: `customer`, `admin`, `trainer`, or `vendor`. |
| `date_of_birth`   | DATE           | Nullable                                    | Used for age verification and personalization. |
| `gender`          | VARCHAR(10)    | Nullable, CHECK                             | One of `male`, `female`, `other`. |
| `is_verified`     | BOOLEAN        | NOT NULL, DEFAULT `FALSE`                   | Whether the user has verified their email/phone. |
| `is_active`       | BOOLEAN        | NOT NULL, DEFAULT `TRUE`                    | Soft-disable flag; inactive users cannot log in. |
| `profile_picture` | VARCHAR(255)   | Nullable                                    | File path or URL to the user's avatar image. |
| `created_at`      | TIMESTAMP      | NOT NULL, DEFAULT `CURRENT_TIMESTAMP`       | Record creation timestamp. |
| `updated_at`      | TIMESTAMP      | NOT NULL, auto-updates on change            | Last modification timestamp. |
| `last_login_at`   | TIMESTAMP      | Nullable                                    | Timestamp of the user's most recent login. |

## Indexes
- `idx_users_email` — speeds up lookups during login/authentication.
- `idx_users_role` — speeds up filtering users by role (e.g., listing all trainers or vendors).

## Roles
| Role       | Description |
|------------|-------------|
| `customer` | Default role; can browse and purchase products/memberships. |
| `trainer`  | Can manage training programs, schedules, or related listings. |
| `vendor`   | Can manage product listings (supplements, equipment, apparel). |
| `admin`    | Full administrative access to the platform. |

## Common Queries

**Find a user by email (login lookup):**
```sql
SELECT * FROM users WHERE email = 'example@gmail.com';
```

**List all active trainers:**
```sql
SELECT first_name, last_name, email 
FROM users 
WHERE role = 'trainer' AND is_active = TRUE;
```

**Find duplicate emails (should not occur due to UNIQUE constraint):**
```sql
SELECT email, COUNT(*) AS total
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

**Deactivate a user instead of deleting:**
```sql
UPDATE users SET is_active = FALSE WHERE user_id = '<uuid>';
```

## Notes & Best Practices
- `password_hash` must always store a hashed value (e.g., bcrypt/argon2) — never plain text passwords.
- Prefer setting `is_active = FALSE` over deleting rows, to preserve order/transaction history tied to a user.
- `email` and `phone` uniqueness is enforced at the database level — handle duplicate-entry errors gracefully in the application layer.
- `gen_random_uuid()`/`UUID()` default for `user_id` requires **MySQL 8.0.13+**. On older versions, generate UUIDs in the application layer before insert.
- Related tables that typically reference `user_id` as a foreign key: `addresses`, `orders`, `cart_items`, `memberships`, `fitness_profiles`.

## Sample Data
A sample dataset of 220 dummy users (`insert_users.sql`) is available for testing and development purposes.
