# Resident Directory Database Schema

## Connection
```
psql postgresql://appuser:dbuser123@localhost:5000/myapp
```

## Tables

### 1. users
Stores user authentication and role information.

| Column | Type | Constraints | Default |
|--------|------|------------|---------|
| id | UUID | PRIMARY KEY | uuid_generate_v4() |
| username | VARCHAR(100) | NOT NULL, UNIQUE | - |
| email | VARCHAR(255) | NOT NULL, UNIQUE | - |
| password_hash | VARCHAR(255) | NOT NULL | - |
| role | VARCHAR(20) | NOT NULL, CHECK (admin/resident) | 'resident' |
| is_active | BOOLEAN | NOT NULL | TRUE |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | NOW() |
| updated_at | TIMESTAMP WITH TIME ZONE | NOT NULL | NOW() |

**Indexes:** idx_users_email, idx_users_username, idx_users_role
**Trigger:** trigger_users_updated_at (auto-updates updated_at on UPDATE)

### 2. residents
Stores resident profile information.

| Column | Type | Constraints | Default |
|--------|------|------------|---------|
| id | UUID | PRIMARY KEY | uuid_generate_v4() |
| user_id | UUID | REFERENCES users(id) ON DELETE SET NULL | - |
| first_name | VARCHAR(100) | NOT NULL | - |
| last_name | VARCHAR(100) | NOT NULL | - |
| email | VARCHAR(255) | | - |
| phone | VARCHAR(50) | | - |
| unit_number | VARCHAR(50) | NOT NULL | - |
| building | VARCHAR(100) | | - |
| photo_url | VARCHAR(500) | | - |
| move_in_date | DATE | | - |
| move_out_date | DATE | | - |
| is_active | BOOLEAN | NOT NULL | TRUE |
| notes | TEXT | | - |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | NOW() |
| updated_at | TIMESTAMP WITH TIME ZONE | NOT NULL | NOW() |

**Indexes:** idx_residents_user_id, idx_residents_unit_number, idx_residents_last_name, idx_residents_is_active
**Trigger:** trigger_residents_updated_at (auto-updates updated_at on UPDATE)

### 3. announcements
Stores community announcements.

| Column | Type | Constraints | Default |
|--------|------|------------|---------|
| id | UUID | PRIMARY KEY | uuid_generate_v4() |
| title | VARCHAR(255) | NOT NULL | - |
| content | TEXT | NOT NULL | - |
| priority | VARCHAR(20) | NOT NULL, CHECK (low/normal/high/urgent) | 'normal' |
| author_id | UUID | REFERENCES users(id) ON DELETE SET NULL | - |
| is_published | BOOLEAN | NOT NULL | TRUE |
| published_at | TIMESTAMP WITH TIME ZONE | | - |
| expires_at | TIMESTAMP WITH TIME ZONE | | - |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | NOW() |
| updated_at | TIMESTAMP WITH TIME ZONE | NOT NULL | NOW() |

**Indexes:** idx_announcements_author_id, idx_announcements_is_published, idx_announcements_priority
**Trigger:** trigger_announcements_updated_at (auto-updates updated_at on UPDATE)

### 4. emergency_contacts
Stores emergency contact information for residents.

| Column | Type | Constraints | Default |
|--------|------|------------|---------|
| id | UUID | PRIMARY KEY | uuid_generate_v4() |
| resident_id | UUID | REFERENCES residents(id) ON DELETE CASCADE | - |
| contact_name | VARCHAR(200) | NOT NULL | - |
| relationship | VARCHAR(100) | | - |
| phone | VARCHAR(50) | NOT NULL | - |
| email | VARCHAR(255) | | - |
| is_primary | BOOLEAN | NOT NULL | FALSE |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | NOW() |
| updated_at | TIMESTAMP WITH TIME ZONE | NOT NULL | NOW() |

**Indexes:** idx_emergency_contacts_resident_id
**Trigger:** trigger_emergency_contacts_updated_at (auto-updates updated_at on UPDATE)

### 5. audit_log
Tracks all user actions in the system.

| Column | Type | Constraints | Default |
|--------|------|------------|---------|
| id | UUID | PRIMARY KEY | uuid_generate_v4() |
| user_id | UUID | REFERENCES users(id) ON DELETE SET NULL | - |
| action | VARCHAR(100) | NOT NULL | - |
| entity_type | VARCHAR(100) | NOT NULL | - |
| entity_id | UUID | | - |
| details | JSONB | | - |
| ip_address | VARCHAR(45) | | - |
| created_at | TIMESTAMP WITH TIME ZONE | NOT NULL | NOW() |

**Indexes:** idx_audit_log_user_id, idx_audit_log_entity_type, idx_audit_log_created_at

## Extensions
- **uuid-ossp**: Used for UUID generation via `uuid_generate_v4()`

## Trigger Function
- **update_updated_at_column()**: Automatically sets `updated_at = NOW()` before any UPDATE on tables with that column.

## Seed Data

### Default Admin User
- **Username:** admin
- **Email:** admin@community.com
- **Password:** admin123 (bcrypt hashed)
- **Role:** admin

### Sample Residents
| Name | Unit | Building | User Account |
|------|------|----------|-------------|
| John Smith | 101A | Building A | jsmith |
| Maria Johnson | 205B | Building B | mjohnson |
| David Williams | 310C | Building C | dwilliams |
| Sarah Chen | 412A | Building A | (none) |

### Sample Announcements
1. Welcome to the Community Portal (high priority)
2. Scheduled Maintenance - Water Shut Off (urgent)
3. Community BBQ This Weekend (normal)

### Sample Emergency Contacts
- John Smith → Jane Smith (Spouse, primary)
- Maria Johnson → Carlos Johnson (Brother, primary)
- David Williams → Linda Williams (Mother, primary), Robert Williams (Father)
