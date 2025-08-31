# Carousel App - Database Design (Lookup Tables)

## Table Structure

### 1. User Management

#### `profiles`
```sql
id (PK) - UUID
name - VARCHAR(100) NOT NULL
photo_url - VARCHAR(500)
relationship_type_id (FK) - UUID REFERENCES relationship_types(id)
is_active - BOOLEAN DEFAULT true
created_at - TIMESTAMP
updated_at - TIMESTAMP
```

### 2. Lookup Tables

#### `relationship_types`
```sql
id (PK) - UUID
code - VARCHAR(50) UNIQUE NOT NULL -- 'self', 'spouse', 'child', 'other'
display_name - VARCHAR(100) NOT NULL -- 'Self', 'Spouse', 'Child', 'Other'
sort_order - INTEGER
is_active - BOOLEAN DEFAULT true
created_at - TIMESTAMP
```

#### `clothing_types`
```sql
id (PK) - UUID
code - VARCHAR(50) UNIQUE NOT NULL -- 'shirt', 'pants', 'dress', etc.
display_name - VARCHAR(100) NOT NULL -- 'Shirt', 'Pants', 'Dress'
category - VARCHAR(50) -- 'tops', 'bottoms', 'accessories', 'underwear'
default_to_laundry - BOOLEAN DEFAULT true
can_go_to_rack - BOOLEAN DEFAULT true
sort_order - INTEGER
is_active - BOOLEAN DEFAULT true
created_at - TIMESTAMP
```

#### `state_types`
```sql
id (PK) - UUID
code - VARCHAR(50) UNIQUE NOT NULL -- 'available', 'worn_today', 'on_rack', 'in_laundry'
display_name - VARCHAR(100) NOT NULL -- 'Available', 'Worn Today', 'On Rack', 'In Laundry'
is_available_state - BOOLEAN DEFAULT false -- true only for 'available'
sort_order - INTEGER
is_active - BOOLEAN DEFAULT true
created_at - TIMESTAMP
```

#### `context_types`
```sql
id (PK) - UUID
code - VARCHAR(50) UNIQUE NOT NULL -- 'indoor', 'outdoor', 'shared'
display_name - VARCHAR(100) NOT NULL -- 'Indoor', 'Outdoor', 'Shared'
sort_order - INTEGER
is_active - BOOLEAN DEFAULT true
created_at - TIMESTAMP
```

#### `trip_status_types`
```sql
id (PK) - UUID
code - VARCHAR(50) UNIQUE NOT NULL -- 'planning', 'active', 'completed', 'cancelled'
display_name - VARCHAR(100) NOT NULL -- 'Planning', 'Active', 'Completed', 'Cancelled'
sort_order - INTEGER
is_active - BOOLEAN DEFAULT true
created_at - TIMESTAMP
```

### 3. Clothing Management

#### `clothing_items`
```sql
id (PK) - UUID
profile_id (FK) - UUID REFERENCES profiles(id)
name - VARCHAR(200) NOT NULL
photo_url - VARCHAR(500) NOT NULL
clothing_type_id (FK) - UUID REFERENCES clothing_types(id)
context_type_id (FK) - UUID REFERENCES context_types(id)
color - VARCHAR(50)
brand - VARCHAR(100)
notes - TEXT
is_deleted - BOOLEAN DEFAULT false
created_at - TIMESTAMP
updated_at - TIMESTAMP
```

#### `wardrobe_contexts`
```sql
id (PK) - UUID
clothing_item_id (FK) - UUID REFERENCES clothing_items(id)
profile_id (FK) - UUID REFERENCES profiles(id)
context_type_id (FK) - UUID REFERENCES context_types(id) -- 'indoor' or 'outdoor'
is_active - BOOLEAN DEFAULT true
created_at - TIMESTAMP
```

### 4. State Management

#### `clothing_states`
```sql
id (PK) - UUID
clothing_item_id (FK) - UUID REFERENCES clothing_items(id)
profile_id (FK) - UUID REFERENCES profiles(id)
state_type_id (FK) - UUID REFERENCES state_types(id)
context - VARCHAR(20) DEFAULT 'home' -- 'home' or 'travel' (kept simple)
changed_at - TIMESTAMP DEFAULT CURRENT_TIMESTAMP
is_current - BOOLEAN DEFAULT true
notes - TEXT
```

#### `state_history`
```sql
id (PK) - UUID
clothing_item_id (FK) - UUID REFERENCES clothing_items(id)
profile_id (FK) - UUID REFERENCES profiles(id)
from_state_type_id (FK) - UUID REFERENCES state_types(id)
to_state_type_id (FK) - UUID REFERENCES state_types(id)
context - VARCHAR(20) DEFAULT 'home' -- 'home' or 'travel'
changed_at - TIMESTAMP DEFAULT CURRENT_TIMESTAMP
trigger_event - VARCHAR(50) -- 'outfit_logged', 'sent_to_rack', 'sent_to_laundry', 'marked_clean', 'packed', 'unpacked'
```

### 5. Outfit Tracking

#### `outfit_logs`
```sql
id (PK) - UUID
profile_id (FK) - UUID REFERENCES profiles(id)
date - DATE NOT NULL
context_type_id (FK) - UUID REFERENCES context_types(id) -- indoor or outdoor
notes - TEXT
created_at - TIMESTAMP
updated_at - TIMESTAMP
```

#### `outfit_items`
```sql
id (PK) - UUID
outfit_log_id (FK) - UUID REFERENCES outfit_logs(id)
clothing_item_id (FK) - UUID REFERENCES clothing_items(id)
created_at - TIMESTAMP
```

### 6. Travel Management

#### `trips`
```sql
id (PK) - UUID
profile_id (FK) - UUID REFERENCES profiles(id)
name - VARCHAR(200) NOT NULL
start_date - DATE NOT NULL
end_date - DATE
status_type_id (FK) - UUID REFERENCES trip_status_types(id)
destination - VARCHAR(200)
notes - TEXT
created_at - TIMESTAMP
updated_at - TIMESTAMP
```

#### `trip_packing`
```sql
id (PK) - UUID
trip_id (FK) - UUID REFERENCES trips(id)
clothing_item_id (FK) - UUID REFERENCES clothing_items(id)
packed_at - TIMESTAMP DEFAULT CURRENT_TIMESTAMP
unpacked_at - TIMESTAMP NULL
unpacked_to_state_type_id (FK) - UUID REFERENCES state_types(id) NULL
```

#### `travel_outfit_logs`
```sql
id (PK) - UUID
trip_id (FK) - UUID REFERENCES trips(id)
profile_id (FK) - UUID REFERENCES profiles(id)
date - DATE NOT NULL
notes - TEXT
created_at - TIMESTAMP
```

#### `travel_outfit_items`
```sql
id (PK) - UUID
travel_outfit_log_id (FK) - UUID REFERENCES travel_outfit_logs(id)
clothing_item_id (FK) - UUID REFERENCES clothing_items(id)
created_at - TIMESTAMP
```

### 7. Settings

#### `profile_settings`
```sql
id (PK) - UUID
profile_id (FK) - UUID REFERENCES profiles(id)
setting_key - VARCHAR(100) NOT NULL
setting_value - TEXT
created_at - TIMESTAMP
updated_at - TIMESTAMP
```

## Initial Lookup Data

### Sample Clothing Types
```sql
INSERT INTO clothing_types (id, code, display_name, category, default_to_laundry, can_go_to_rack) VALUES
(uuid(), 'shirt', 'Shirt', 'tops', false, true),
(uuid(), 'pants', 'Pants', 'bottoms', false, true),
(uuid(), 'dress', 'Dress', 'tops', false, true),
(uuid(), 'jacket', 'Jacket', 'outerwear', false, true),
(uuid(), 'underwear', 'Underwear', 'underwear', true, false),
(uuid(), 'socks', 'Socks', 'underwear', true, false),
(uuid(), 'shoes', 'Shoes', 'accessories', false, true),
(uuid(), 'belt', 'Belt', 'accessories', false, true),
(uuid(), 'watch', 'Watch', 'accessories', false, true),
(uuid(), 'glasses', 'Glasses', 'accessories', false, true);
```

### Sample State Types
```sql
INSERT INTO state_types (id, code, display_name, is_available_state) VALUES
(uuid(), 'available', 'Available', true),
(uuid(), 'worn_today', 'Worn Today', false),
(uuid(), 'on_rack', 'On Rack', false),
(uuid(), 'in_laundry', 'In Laundry', false);
```

### Sample Context Types
```sql
INSERT INTO context_types (id, code, display_name) VALUES
(uuid(), 'indoor', 'Indoor'),
(uuid(), 'outdoor', 'Outdoor'),
(uuid(), 'shared', 'Shared');
```

## Indexes

### Primary Indexes
```sql
-- Performance for daily queries
CREATE INDEX idx_clothing_states_current ON clothing_states(clothing_item_id, profile_id, is_current);
CREATE INDEX idx_wardrobe_contexts_active ON wardrobe_contexts(profile_id, context_type_id, is_active);
CREATE INDEX idx_outfit_logs_date ON outfit_logs(profile_id, date DESC);

-- Lookup table performance
CREATE INDEX idx_clothing_types_active ON clothing_types(is_active, sort_order);
CREATE INDEX idx_state_types_active ON state_types(is_active, sort_order);
CREATE INDEX idx_context_types_active ON context_types(is_active, sort_order);

-- Foreign key performance
CREATE INDEX idx_clothing_items_type ON clothing_items(clothing_type_id);
CREATE INDEX idx_clothing_items_context ON clothing_items(context_type_id);
CREATE INDEX idx_clothing_states_state_type ON clothing_states(state_type_id);

-- Travel queries
CREATE INDEX idx_trips_status ON trips(profile_id, status_type_id);
CREATE INDEX idx_trip_packing_active ON trip_packing(trip_id, unpacked_at);

-- History and analytics
CREATE INDEX idx_state_history_item_date ON state_history(clothing_item_id, changed_at DESC);
CREATE INDEX idx_outfit_items_lookup ON outfit_items(outfit_log_id, clothing_item_id);
```

## Image Recognition Approach

### Outfit Logging Without Photo Storage
The app uses image recognition to identify clothing items from photos without storing the actual photos:

1. **Photo Processing**: User takes photo of outfit
2. **Image Recognition**: App identifies clothing items and maps to existing wardrobe items  
3. **Data Extraction**: Only the clothing item IDs are stored in `outfit_items` table
4. **No Photo Storage**: Photos are processed and discarded, saving storage space

### Benefits
- **Storage Efficiency**: No redundant photo storage needed
- **Automatic Logging**: Direct identification and mapping to existing items
- **Data Consistency**: Clean relationship between outfits and clothing items
- **Better Analytics**: Accurate usage tracking based on actual item relationships

## Benefits of Lookup Table Approach

### 1. Flexibility
- **Easy additions**: Add new clothing types without schema changes
- **Soft disable**: Deactivate categories without breaking existing data
- **Ordering control**: Custom sort orders for UI display
- **Metadata storage**: Additional properties like `default_to_laundry`

### 2. Maintainability
- **Centralized management**: All categories managed in one place
- **Consistent naming**: Code vs display name separation
- **Future-proof**: Easy to add new properties to lookup tables

### 3. Performance
- **Indexed lookups**: Fast joins with proper indexing
- **Cached data**: Lookup tables can be cached in application
- **Referential integrity**: Foreign key constraints ensure data consistency

### 4. Internationalization Ready
- **Separate display names**: Easy to extend for multiple languages
- **Code stability**: Internal codes remain constant across locales

## Common Query Examples

```sql
-- Get available outdoor clothes for a profile
SELECT ci.*, ct.display_name as type_name
FROM clothing_items ci
JOIN clothing_types ct ON ci.clothing_type_id = ct.id
JOIN wardrobe_contexts wc ON ci.id = wc.clothing_item_id
JOIN context_types ctx ON wc.context_type_id = ctx.id
JOIN clothing_states cs ON ci.id = cs.clothing_item_id
JOIN state_types st ON cs.state_type_id = st.id
WHERE ci.profile_id = ? 
  AND ctx.code = 'outdoor'
  AND st.is_available_state = true
  AND cs.is_current = true;

-- Add new clothing type without schema change
INSERT INTO clothing_types (id, code, display_name, category, default_to_laundry, can_go_to_rack)
VALUES (uuid(), 'sweater', 'Sweater', 'tops', false, true);
```