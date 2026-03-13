

## Updated Plan: Add C2 Foreign Keys to Critical Tables

Adding the foreign key constraints issue to the existing QA fix plan. This will be placed in **Batch 3 (Medium Priority)** since it requires careful migration planning to avoid breaking existing data.

### C2. Add Foreign Keys on Critical Tables

**Problem**: `email_history`, `contacts`, `accounts`, `deals`, `campaigns`, `action_items` have no foreign keys to `auth.users` or to each other. Deleting an account does NOT cascade to related contacts, deals, or campaigns. Orphan records accumulate. No referential integrity at the database level.

**Important constraint**: Per Supabase instructions, we CANNOT create foreign keys referencing `auth.users` directly. Instead, we reference `profiles.id`.

**Migration plan** — single migration adding foreign keys with `ON DELETE` behavior:

| Table | Column | References | ON DELETE |
|-------|--------|------------|-----------|
| `contacts` | `contact_owner` | `profiles(id)` | SET NULL |
| `contacts` | `created_by` | `profiles(id)` | SET NULL |
| `accounts` | `account_owner` | `profiles(id)` | SET NULL |
| `accounts` | `created_by` | `profiles(id)` | SET NULL |
| `deals` | `created_by` | `profiles(id)` | SET NULL |
| `deals` | `account_id` | `accounts(id)` | SET NULL |
| `deals` | `budget_owner_contact_id` | `contacts(id)` | SET NULL |
| `deals` | `champion_contact_id` | `contacts(id)` | SET NULL |
| `deals` | `objector_contact_id` | `contacts(id)` | SET NULL |
| `deals` | `influencer_contact_id` | `contacts(id)` | SET NULL |
| `deals` | `campaign_id` | `campaigns(id)` | SET NULL |
| `deal_stakeholders` | `deal_id` | `deals(id)` | CASCADE |
| `deal_stakeholders` | `contact_id` | `contacts(id)` | CASCADE |
| `deal_action_items` | `deal_id` | `deals(id)` | CASCADE |
| `action_items` | `assigned_to` | `profiles(id)` | SET NULL |
| `campaign_accounts` | `campaign_id` | `campaigns(id)` | CASCADE |
| `campaign_accounts` | `account_id` | `accounts(id)` | CASCADE |
| `campaign_contacts` | `campaign_id` | `campaigns(id)` | CASCADE |
| `campaign_contacts` | `contact_id` | `contacts(id)` | CASCADE |
| `campaign_communications` | `campaign_id` | `campaigns(id)` | CASCADE |
| `campaign_communications` | `contact_id` | `contacts(id)` | SET NULL |
| `campaign_communications` | `account_id` | `accounts(id)` | SET NULL |
| `campaign_materials` | `campaign_id` | `campaigns(id)` | CASCADE |
| `campaign_email_templates` | `campaign_id` | `campaigns(id)` | CASCADE |
| `campaign_phone_scripts` | `campaign_id` | `campaigns(id)` | CASCADE |
| `email_history` | `contact_id` | `contacts(id)` | SET NULL |
| `email_history` | `account_id` | `accounts(id)` | SET NULL |
| `email_history` | `sent_by` | `profiles(id)` | SET NULL |
| `notifications` | `user_id` | `profiles(id)` | CASCADE |

**Pre-migration data cleanup**: Before adding FKs, we must delete orphan records that reference non-existent IDs. The migration will include cleanup queries first, then `ALTER TABLE ... ADD CONSTRAINT` statements.

**CASCADE vs SET NULL logic**:
- **CASCADE**: Used for child records that have no meaning without the parent (e.g., `deal_stakeholders` without a deal, `campaign_contacts` without a campaign)
- **SET NULL**: Used where the record should survive but lose the reference (e.g., a deal should survive if its linked account is deleted, just lose the `account_id`)

### Updated Batch Order

- **Batch 1 (Critical)**: Email analytics fix, insecure role fallback removal, Mark as Bounced button, Contact table schema alignment
- **Batch 2 (High)**: Confirmation dialogs, error boundary, module_type filter, campaign stale data fix
- **Batch 3 (Medium)**: **Foreign key constraints migration (C2)**, deals pagination, remove console.log spam, fix date sorting
- **Batch 4 (Low)**: Header height consistency, campaign filter UUID fallback, AccountViewModal ID lookup, dynamic years

