### organizations

Purpose:
Stores brewery/company information.

Primary Key:
id

Relationships:
- has many employees
- has many recipes

Fields:
- id - UUID
- name - VARCHAR/TEXT
- address - VARCHAR/TEXT
- city - VARCHAR/TEXT 
- state - VARCHAR/TEXT
- zip - VARCHAR/TEXT
- primary_phone - VARCHAR/TEXT
- secondary_phone - VARCHAR/TEXT
- email - VARCHAR/TEXT
- website - VARCHAR/TEXT

### users

Purpose:
Stores information about application user account

Relationships:
- has one or zero employee

Fields:
- id - UUID
- username - VARCHAR/TEXT
- password_hash - VARCHAR/TEXT
- created_at - TIMESTAMP/TIMESTAMPTZ
- email - VARCHAR/TEXT
- employee_id - UUID, nullable, foreign key → employees.id

### employees

Purpose:

Stores information about employees of an Organization.

Relationships:

- belongs to one organization
- has zero or one user
- belongs to zero or one supervisor
- has one role

Fields:

- id - UUID
- organization_id - UUID, foreign key → organizations.id
- first_name - VARCHAR/TEXT
- last_name - VARCHAR/TEXT
- hire_date - DATE
- birthday - DATE
- phone_1 - VARCHAR/TEXT
- phone_2 - VARCHAR/TEXT
- email - VARCHAR/TEXT
- address - VARCHAR/TEXT
- supervisor_id - UUID, nullable, foreign key → employees.id
- role_id - UUID, foreign key → roles.id

### roles

Purpose:

Define user role within the application

Relationships:

- has zero or many employees
- has zero or many permissions

Fields:

- id - UUID
- name - VARCHAR/TEXT

### permissions

Purpose:

Define permissions grantable to each role

Relationships:

- has zero to many roles

Fields:

- id - UUID
- name - VARCHAR/TEXT

### role_permissions

Purpose:

Connects roles to permissions

Relationships:

- belongs to one role
- belongs to one permission

Fields:

- role_id - UUID, foreign key → roles.id
- permission_id - UUID, foreign key → permissions.id

Primary Key:
role_id + permission_id (composite)



recipes
recipe_versions
grain_bill_items
hop_additions
yeast_additions
water_chemistry

batches
fermentation_events
dry_hop_events
yeast_harvests
quality_control_records
packaging_runs

### recipes

Purpose:
Stores recipe metadata and links to versioned recipe definitions.

Relationships:
- belongs to one organization
- has many recipe_versions

Fields:
- id - UUID
- organization_id - UUID, foreign key → organizations.id
- name - VARCHAR/TEXT
- style - VARCHAR/TEXT, nullable
- default_batch_size_l - NUMERIC/DECIMAL
- created_by - UUID, nullable, foreign key → employees.id
- created_at - TIMESTAMP/TIMESTAMPTZ

### recipe_versions

Purpose:
Immutable versioned recipe specification used to brew batches.

Relationships:
- belongs to one recipe
- has many grain_bill_items, hop_additions, yeast_additions, water_chemistry

Fields:
- id - UUID
- recipe_id - UUID, foreign key → recipes.id
- version_number - INTEGER
- notes - TEXT
- target_og - NUMERIC/DECIMAL, nullable
- target_fg - NUMERIC/DECIMAL, nullable
- target_abv - NUMERIC/DECIMAL, nullable
- target_ibu - NUMERIC/DECIMAL, nullable
- target_color_srm - NUMERIC/DECIMAL, nullable
- mash_temp_c - NUMERIC/DECIMAL, nullable
- mash_time_min - INTEGER, nullable
- boil_time_min - INTEGER, nullable
- estimated_efficiency_pct - NUMERIC/DECIMAL, nullable
- created_by - UUID, foreign key → employees.id
- created_at - TIMESTAMP/TIMESTAMPTZ

Primary Key:
- id

### grain_bill_items

Purpose:
Grains/adjuncts used in a specific recipe version.

Relationships:
- belongs to one recipe_version
- references an ingredient (grain) in the inventory catalog

Fields:
- id - UUID
- recipe_version_id - UUID, foreign key → recipe_versions.id
- ingredient_id - UUID, foreign key → ingredients.id
- amount_kg - NUMERIC/DECIMAL
- percentage - NUMERIC/DECIMAL (optional; for convenience)
- order - INTEGER (mash order)

### hop_additions

Purpose:
Hop additions for a recipe version (boil/whirlpool/dry hop).

Relationships:
- belongs to one recipe_version
- references an ingredient (hop) in the inventory catalog

Fields:
- id - UUID
- recipe_version_id - UUID, foreign key → recipe_versions.id
- ingredient_id - UUID, foreign key → ingredients.id
- amount_g - NUMERIC/DECIMAL
- addition_time_min - INTEGER (minutes before end of boil; 0 = flameout; negative for post-boil)
- form - VARCHAR/TEXT (pellet/whole/plug)
- alpha_acid_pct - NUMERIC/DECIMAL, nullable
- notes - TEXT

### yeast_additions

Purpose:
Yeast strain and pitching details for a recipe version.

Fields:
- id - UUID
- recipe_version_id - UUID, foreign key → recipe_versions.id
- ingredient_id - UUID, foreign key → ingredients.id
- amount_g_or_billion_cells - NUMERIC/DECIMAL
- pitch_temp_c - NUMERIC/DECIMAL, nullable
- rehydrate - BOOLEAN, default false
- notes - TEXT

### water_chemistry

Purpose:
Target water profile and mineral additions for a recipe version.

Fields:
- id - UUID
- recipe_version_id - UUID, foreign key → recipe_versions.id
- calcium_mg_l - NUMERIC/DECIMAL, nullable
- magnesium_mg_l - NUMERIC/DECIMAL, nullable
- sodium_mg_l - NUMERIC/DECIMAL, nullable
- chloride_mg_l - NUMERIC/DECIMAL, nullable
- sulfate_mg_l - NUMERIC/DECIMAL, nullable
- bicarbonate_mg_l - NUMERIC/DECIMAL, nullable
- target_ph - NUMERIC/DECIMAL, nullable
- notes - TEXT

### batches

Purpose:
Track individual brewed batches linked to a specific recipe version.

Relationships:
- belongs to one recipe_version
- may be assigned to one or more tanks over time

Fields:
- id - UUID
- organization_id - UUID, foreign key → organizations.id
- recipe_version_id - UUID, foreign key → recipe_versions.id
- batch_number - VARCHAR/TEXT (human-friendly identifier)
- target_volume_l - NUMERIC/DECIMAL
- actual_volume_l - NUMERIC/DECIMAL, nullable
- target_og - NUMERIC/DECIMAL, nullable
- actual_og - NUMERIC/DECIMAL, nullable
- target_fg - NUMERIC/DECIMAL, nullable
- actual_fg - NUMERIC/DECIMAL, nullable
- status - VARCHAR/TEXT (enum: brewing, fermenting, conditioning, packaged, cancelled)
- brewed_at - TIMESTAMP/TIMESTAMPTZ, nullable
- completed_at - TIMESTAMP/TIMESTAMPTZ, nullable
- created_by - UUID, foreign key → employees.id

### fermentation_events

Purpose:
Time-series events and measurements during fermentation for a batch.

Fields:
- id - UUID
- batch_id - UUID, foreign key → batches.id
- event_type - VARCHAR/TEXT (fermentation_start, temp_change, gravity_reading, racked, etc.)
- event_time - TIMESTAMP/TIMESTAMPTZ
- temperature_c - NUMERIC/DECIMAL, nullable
- gravity - NUMERIC/DECIMAL, nullable
- notes - TEXT

### dry_hop_events

Fields:
- id - UUID
- batch_id - UUID, foreign key → batches.id
- ingredient_id - UUID, foreign key → ingredients.id
- amount_g - NUMERIC/DECIMAL
- added_at - TIMESTAMP/TIMESTAMPTZ
- duration_days - NUMERIC/DECIMAL, nullable
- notes - TEXT

### yeast_harvests

Purpose:
Track harvested yeast from fermentation for reuse or disposal.

Fields:
- id - UUID
- batch_id - UUID, foreign key → batches.id
- ingredient_id - UUID, foreign key → ingredients.id (yeast strain)
- harvested_volume_l - NUMERIC/DECIMAL
- cell_count_billion - NUMERIC/DECIMAL, nullable
- harvested_at - TIMESTAMP/TIMESTAMPTZ
- storage_location - VARCHAR/TEXT, nullable
- harvested_by - UUID, foreign key → employees.id
- notes - TEXT

### quality_control_records

Purpose:
QC samples and measurements for batches.

Fields:
- id - UUID
- batch_id - UUID, foreign key → batches.id
- sample_time - TIMESTAMP/TIMESTAMPTZ
- ph - NUMERIC/DECIMAL, nullable
- gravity - NUMERIC/DECIMAL, nullable
- temperature_c - NUMERIC/DECIMAL, nullable
- sensory_notes - TEXT
- analyst_id - UUID, foreign key → employees.id

### packaging_runs

Purpose:
Records of packaging activities that consume batch volume and produce package units.

Fields:
- id - UUID
- batch_id - UUID, foreign key → batches.id
- packaged_at - TIMESTAMP/TIMESTAMPTZ
- package_type - VARCHAR/TEXT (keg/can/bottle)
- package_volume_l_each - NUMERIC/DECIMAL
- units - INTEGER
- total_volume_l - NUMERIC/DECIMAL
- lot_code - VARCHAR/TEXT
- packaged_by - UUID, foreign key → employees.id

### tanks

Purpose:
Physical vessels used throughout the brew and packaging process.

Fields:
- id - UUID
- organization_id - UUID, foreign key → organizations.id
- name - VARCHAR/TEXT
- tank_type - VARCHAR/TEXT (mash/lauter/kettle/whirlpool/fermenter/bright/storage)
- capacity_l - NUMERIC/DECIMAL
- working_volume_l - NUMERIC/DECIMAL
- current_status - VARCHAR/TEXT (available, in_use, cleaning, maintenance)
- location - VARCHAR/TEXT, nullable
- notes - TEXT

### tank_assignments

Purpose:
Track which batch is in which tank and for what role/time window.

Fields:
- id - UUID
- tank_id - UUID, foreign key → tanks.id
- batch_id - UUID, foreign key → batches.id
- role - VARCHAR/TEXT (fermentation, conditioning, cold_storage)
- assigned_at - TIMESTAMP/TIMESTAMPTZ
- unassigned_at - TIMESTAMP/TIMESTAMPTZ, nullable
- notes - TEXT

### maintenance_records

Purpose:
Maintenance history for tanks and other equipment.

Fields:
- id - UUID
- tank_id - UUID, foreign key → tanks.id
- performed_at - TIMESTAMP/TIMESTAMPTZ
- performed_by - UUID, foreign key → employees.id
- maintenance_type - VARCHAR/TEXT
- notes - TEXT
- next_due_date - DATE, nullable

### Inventory & suppliers (simple scheme)

Purpose:
Manage on-hand stock of ingredients, packaging, and materials.

Tables:

- suppliers

	Purpose: vendor contact information

	Fields:
	- id - UUID
	- organization_id - UUID, foreign key → organizations.id
	- name - VARCHAR/TEXT
	- contact_name - VARCHAR/TEXT
	- phone - VARCHAR/TEXT
	- email - VARCHAR/TEXT
	- address - VARCHAR/TEXT

- ingredients

	Purpose: catalog of purchasable/usable items (grains, hops, yeasts, chemicals, packaging)

	Fields:
	- id - UUID
	- organization_id - UUID, foreign key → organizations.id
	- supplier_id - UUID, foreign key → suppliers.id, nullable
	- name - VARCHAR/TEXT
	- sku - VARCHAR/TEXT, nullable
	- ingredient_type - VARCHAR/TEXT (grain, hop, yeast, adjunct, chemical, packaging)
	- default_uom - VARCHAR/TEXT (kg,g,L,ea)
	- purchase_unit_size - NUMERIC/DECIMAL (e.g., 25 kg sack)
	- density_kg_per_l - NUMERIC/DECIMAL, nullable
	- notes - TEXT

- stock_items

	Purpose: lot-tracked stock entries for ingredients (represents physical inventory units)

	Fields:
	- id - UUID
	- ingredient_id - UUID, foreign key → ingredients.id
	- organization_id - UUID, foreign key → organizations.id
	- lot_code - VARCHAR/TEXT, nullable
	- quantity - NUMERIC/DECIMAL
	- uom - VARCHAR/TEXT
	- received_at - TIMESTAMP/TIMESTAMPTZ
	- best_before - DATE, nullable
	- location - VARCHAR/TEXT (silo/room/bin)
	- cost_per_unit - NUMERIC/DECIMAL, nullable

- inventory_transactions

	Purpose: ledger of inventory movements (receipts, consumption by batches, transfers, adjustments)

	Fields:
	- id - UUID
	- organization_id - UUID, foreign key → organizations.id
	- stock_item_id - UUID, foreign key → stock_items.id, nullable
	- ingredient_id - UUID, foreign key → ingredients.id
	- transaction_type - VARCHAR/TEXT (receipt, consumption, transfer, adjustment, spoilage)
	- quantity - NUMERIC/DECIMAL (positive for increase, negative for decrease)
	- uom - VARCHAR/TEXT
	- related_batch_id - UUID, nullable, foreign key → batches.id
	- related_document - VARCHAR/TEXT (e.g., PO-123)
	- performed_by - UUID, foreign key → employees.id
	- performed_at - TIMESTAMP/TIMESTAMPTZ
	- notes - TEXT

Notes:
- Current stock for an ingredient can be derived by summing `inventory_transactions.quantity` grouped by `ingredient_id` (or by stock_item_id for lot-level tracking).
- `stock_items` is optional: systems that don't track lots may omit it and use `inventory_transactions` per `ingredient_id` only.

### purchase_orders (basic)

Purpose:
Track orders placed to suppliers for ingredients and materials.

Fields:
- id - UUID
- organization_id - UUID, foreign key → organizations.id
- supplier_id - UUID, foreign key → suppliers.id
- po_number - VARCHAR/TEXT
- status - VARCHAR/TEXT (open, received, cancelled)
- ordered_at - TIMESTAMP/TIMESTAMPTZ
- expected_at - DATE, nullable
- created_by - UUID, foreign key → employees.id

### purchase_order_lines

Fields:
- id - UUID
- purchase_order_id - UUID, foreign key → purchase_orders.id
- ingredient_id - UUID, foreign key → ingredients.id
- quantity - NUMERIC/DECIMAL
- uom - VARCHAR/TEXT
- unit_cost - NUMERIC/DECIMAL
- received_quantity - NUMERIC/DECIMAL, default 0

---
Additional notes:
- Consider adding lightweight indexes on `ingredient_id`, `batch_id`, and `recipe_version_id` for the most queried joins.
- Inventory adjustments should always be recorded as `inventory_transactions` with an atomic reference to the user and optional related document (e.g., QC failure, spoilage report).

