# Carousel App - Database Flow & Data Population

## Phase 1: Initial Setup & Seed Data

### 1.1 Create Lookup Tables (First Priority)
These tables must be created and populated first since other tables reference them.

```sql
-- Create lookup tables in order
CREATE TABLE relationship_types (...);
CREATE TABLE clothing_types (...);
CREATE TABLE state_types (...);
CREATE TABLE context_types (...);
CREATE TABLE trip_status_types (...);
```

### 1.2 Populate Seed Data
**Must happen before any user data can be added.**

```sql
-- Relationship types
INSERT INTO relationship_types VALUES
('self-uuid', 'self', 'Self', 1, true),
('spouse-uuid', 'spouse', 'Spouse', 2, true),
('child-uuid', 'child', 'Child', 3, true),
('other-uuid', 'other', 'Other', 4, true);

-- Context types  
INSERT INTO context_types VALUES
('indoor-uuid', 'indoor', 'Indoor', 1, true),
('outdoor-uuid', 'outdoor', 'Outdoor', 2, true),
('shared-uuid', 'shared', 'Shared', 3, true);

-- State types
INSERT INTO state_types VALUES
('available-uuid', 'available', 'Available', true, 1, true),
('worn-uuid', 'worn_today', 'Worn Today', false, 2, true),
('rack-uuid', 'on_rack', 'On Rack', false, 3, true),
('laundry-uuid', 'in_laundry', 'In Laundry', false, 4, true);

-- Trip status types
INSERT INTO trip_status_types VALUES
('planning-uuid', 'planning', 'Planning', 1, true),
('active-uuid', 'active', 'Active', 2, true),
('completed-uuid', 'completed', 'Completed', 3, true);

-- Clothing types (sample - can be extended later)
INSERT INTO clothing_types VALUES
('shirt-uuid', 'shirt', 'Shirt', 'tops', false, true, 1, true),
('pants-uuid', 'pants', 'Pants', 'bottoms', false, true, 2, true),
('underwear-uuid', 'underwear', 'Underwear', 'underwear', true, false, 10, true),
('shoes-uuid', 'shoes', 'Shoes', 'accessories', false, true, 20, true);
```

## Phase 2: User Onboarding

### 2.1 Create First Profile
**User creates their primary profile**

```sql
-- Create main user profile
INSERT INTO profiles (id, name, photo_url, relationship_type_id, is_active)
VALUES ('user1-uuid', 'John Doe', 'profile.jpg', 'self-uuid', true);

-- Optional: Create default settings
INSERT INTO profile_settings (profile_id, setting_key, setting_value)
VALUES 
('user1-uuid', 'default_indoor_context', 'indoor-uuid'),
('user1-uuid', 'default_outdoor_context', 'outdoor-uuid');
```

### 2.2 Add Family Members (Optional)
```sql
-- Add spouse
INSERT INTO profiles (id, name, relationship_type_id)
VALUES ('spouse-uuid', 'Jane Doe', 'spouse-uuid');

-- Add children
INSERT INTO profiles (id, name, relationship_type_id)  
VALUES ('child1-uuid', 'Little John', 'child-uuid');
```

## Phase 3: Wardrobe Building

### 3.1 Add Clothing Items
**User uploads photos and categorizes clothing**

```sql
-- Add a shirt (outdoor only)
INSERT INTO clothing_items (id, profile_id, name, photo_url, clothing_type_id, context_type_id)
VALUES ('shirt1-uuid', 'user1-uuid', 'Blue Work Shirt', 'shirt1.jpg', 'shirt-uuid', 'outdoor-uuid');

-- Add wardrobe context (since it's outdoor only, just outdoor)
INSERT INTO wardrobe_contexts (clothing_item_id, profile_id, context_type_id)
VALUES ('shirt1-uuid', 'user1-uuid', 'outdoor-uuid');

-- Set initial state (available)
INSERT INTO clothing_states (clothing_item_id, profile_id, state_type_id, is_current)
VALUES ('shirt1-uuid', 'user1-uuid', 'available-uuid', true);
```

### 3.2 Add Shared Items
**Items that work for both indoor and outdoor**

```sql
-- Add shared jeans
INSERT INTO clothing_items (id, profile_id, name, photo_url, clothing_type_id, context_type_id)
VALUES ('jeans1-uuid', 'user1-uuid', 'Blue Jeans', 'jeans1.jpg', 'pants-uuid', 'shared-uuid');

-- Add to both wardrobe contexts
INSERT INTO wardrobe_contexts (clothing_item_id, profile_id, context_type_id) VALUES
('jeans1-uuid', 'user1-uuid', 'indoor-uuid'),
('jeans1-uuid', 'user1-uuid', 'outdoor-uuid');

-- Set initial state
INSERT INTO clothing_states (clothing_item_id, profile_id, state_type_id, is_current)
VALUES ('jeans1-uuid', 'user1-uuid', 'available-uuid', true);
```

## Phase 4: Daily Usage

### 4.1 Morning - View Available Clothes
**Query available clothes for outdoor wardrobe**

```sql
SELECT ci.*, ct.display_name as clothing_type
FROM clothing_items ci
JOIN clothing_types ct ON ci.clothing_type_id = ct.id
JOIN wardrobe_contexts wc ON ci.id = wc.clothing_item_id
JOIN context_types ctx ON wc.context_type_id = ctx.id
JOIN clothing_states cs ON ci.id = cs.clothing_item_id
JOIN state_types st ON cs.state_type_id = st.id
WHERE ci.profile_id = 'user1-uuid'
  AND ctx.code = 'outdoor'
  AND st.code = 'available'
  AND cs.is_current = true
  AND ci.is_deleted = false;
```

### 4.2 Log Daily Outfit
**User takes photo of outfit - app uses image recognition to identify clothing**

```sql
-- App processes photo with image recognition to identify clothing items
-- No need to store photo since we extract the clothing IDs we need

-- Create outfit log (no photo_url needed)
INSERT INTO outfit_logs (id, profile_id, date, context_type_id, notes)
VALUES ('outfit1-uuid', 'user1-uuid', '2025-08-31', 'outdoor-uuid', null);

-- Add identified items to outfit (automatically detected from photo)
INSERT INTO outfit_items (outfit_log_id, clothing_item_id) VALUES
('outfit1-uuid', 'shirt1-uuid'),      -- Identified via image recognition
('outfit1-uuid', 'jeans1-uuid');      -- Identified via image recognition

-- Update clothing states to "worn_today"
-- First, mark current states as not current
UPDATE clothing_states 
SET is_current = false 
WHERE clothing_item_id IN ('shirt1-uuid', 'jeans1-uuid') 
  AND is_current = true;

-- Add new "worn_today" states
INSERT INTO clothing_states (clothing_item_id, profile_id, state_type_id, is_current) VALUES
('shirt1-uuid', 'user1-uuid', 'worn-uuid', true),
('jeans1-uuid', 'user1-uuid', 'worn-uuid', true);

-- Record state history
INSERT INTO state_history (clothing_item_id, profile_id, from_state_type_id, to_state_type_id, trigger_event) VALUES
('shirt1-uuid', 'user1-uuid', 'available-uuid', 'worn-uuid', 'outfit_logged'),
('jeans1-uuid', 'user1-uuid', 'available-uuid', 'worn-uuid', 'outfit_logged');
```

### 4.3 End of Day - Sort Worn Items
**User decides what goes to rack vs laundry**

```sql
-- Send shirt to laundry
UPDATE clothing_states 
SET is_current = false 
WHERE clothing_item_id = 'shirt1-uuid' AND is_current = true;

INSERT INTO clothing_states (clothing_item_id, profile_id, state_type_id, is_current)
VALUES ('shirt1-uuid', 'user1-uuid', 'laundry-uuid', true);

-- Send jeans to rack
UPDATE clothing_states 
SET is_current = false 
WHERE clothing_item_id = 'jeans1-uuid' AND is_current = true;

INSERT INTO clothing_states (clothing_item_id, profile_id, state_type_id, is_current)
VALUES ('jeans1-uuid', 'user1-uuid', 'rack-uuid', true);

-- Record history
INSERT INTO state_history (clothing_item_id, profile_id, from_state_type_id, to_state_type_id, trigger_event) VALUES
('shirt1-uuid', 'user1-uuid', 'worn-uuid', 'laundry-uuid', 'sent_to_laundry'),
('jeans1-uuid', 'user1-uuid', 'worn-uuid', 'rack-uuid', 'sent_to_rack');
```

## Phase 5: Laundry Management

### 5.1 Laundry Day
**Mark items as clean and return to available**

```sql
-- View all items in laundry for profile
SELECT ci.name, ci.photo_url
FROM clothing_items ci
JOIN clothing_states cs ON ci.id = cs.clothing_item_id  
JOIN state_types st ON cs.state_type_id = st.id
WHERE ci.profile_id = 'user1-uuid'
  AND st.code = 'in_laundry'
  AND cs.is_current = true;

-- Mark shirt as clean
UPDATE clothing_states 
SET is_current = false 
WHERE clothing_item_id = 'shirt1-uuid' AND is_current = true;

INSERT INTO clothing_states (clothing_item_id, profile_id, state_type_id, is_current)
VALUES ('shirt1-uuid', 'user1-uuid', 'available-uuid', true);

-- Record history
INSERT INTO state_history (clothing_item_id, profile_id, from_state_type_id, to_state_type_id, trigger_event)
VALUES ('shirt1-uuid', 'user1-uuid', 'laundry-uuid', 'available-uuid', 'marked_clean');
```

## Phase 6: Travel Management

### 6.1 Plan Trip
**Create trip and pack items**

```sql
-- Create trip
INSERT INTO trips (id, profile_id, name, start_date, end_date, status_type_id, destination)
VALUES ('trip1-uuid', 'user1-uuid', 'Business Trip', '2025-09-15', '2025-09-18', 'planning-uuid', 'Mumbai');

-- Pack items (removes from home wardrobe)
INSERT INTO trip_packing (trip_id, clothing_item_id) VALUES
('trip1-uuid', 'shirt1-uuid'),
('trip1-uuid', 'jeans1-uuid');

-- Update states to indicate they're packed
UPDATE clothing_states 
SET is_current = false 
WHERE clothing_item_id IN ('shirt1-uuid', 'jeans1-uuid') AND is_current = true;

INSERT INTO clothing_states (clothing_item_id, profile_id, state_type_id, context, is_current) VALUES
('shirt1-uuid', 'user1-uuid', 'available-uuid', 'travel', true),
('jeans1-uuid', 'user1-uuid', 'available-uuid', 'travel', true);
```

### 6.2 During Travel
**Log travel outfits using image recognition**

```sql
-- Start trip
UPDATE trips SET status_type_id = 'active-uuid' WHERE id = 'trip1-uuid';

-- Log travel outfit (image recognition identifies clothing)
INSERT INTO travel_outfit_logs (id, trip_id, profile_id, date, notes)
VALUES ('travel-outfit1-uuid', 'trip1-uuid', 'user1-uuid', '2025-09-15', null);

INSERT INTO travel_outfit_items (travel_outfit_log_id, clothing_item_id)
VALUES ('travel-outfit1-uuid', 'shirt1-uuid');  -- Identified from photo

-- Update travel states
UPDATE clothing_states 
SET is_current = false 
WHERE clothing_item_id = 'shirt1-uuid' AND context = 'travel' AND is_current = true;

INSERT INTO clothing_states (clothing_item_id, profile_id, state_type_id, context, is_current)
VALUES ('shirt1-uuid', 'user1-uuid', 'worn-uuid', 'travel', true);
```

### 6.3 Return from Travel
**Unpack and sort items**

```sql
-- Complete trip
UPDATE trips SET status_type_id = 'completed-uuid' WHERE id = 'trip1-uuid';

-- Unpack items - shirt goes to laundry, jeans to rack
UPDATE trip_packing 
SET unpacked_at = CURRENT_TIMESTAMP, unpacked_to_state_type_id = 'laundry-uuid'
WHERE trip_id = 'trip1-uuid' AND clothing_item_id = 'shirt1-uuid';

UPDATE trip_packing 
SET unpacked_at = CURRENT_TIMESTAMP, unpacked_to_state_type_id = 'rack-uuid'
WHERE trip_id = 'trip1-uuid' AND clothing_item_id = 'jeans1-uuid';

-- Update home states
UPDATE clothing_states 
SET is_current = false 
WHERE clothing_item_id IN ('shirt1-uuid', 'jeans1-uuid') AND is_current = true;

INSERT INTO clothing_states (clothing_item_id, profile_id, state_type_id, context, is_current) VALUES
('shirt1-uuid', 'user1-uuid', 'laundry-uuid', 'home', true),
('jeans1-uuid', 'user1-uuid', 'rack-uuid', 'home', true);
```

## Image Recognition Flow

### Outfit Photo Processing
1. **Photo Capture** - User takes photo of complete outfit
2. **Image Recognition** - App identifies clothing items in photo and matches to existing wardrobe items
3. **Automatic Logging** - Identified items automatically added to `outfit_items` table
4. **Fallback Options** - For unrecognized items or low confidence matches:
   - Manual selection interface
   - Option to add new clothing items
   - Confirmation prompts for uncertain matches

### Benefits of Image Recognition Approach
- **Automatic logging** - No manual selection of worn items needed
- **Storage efficiency** - No need to store redundant outfit photos
- **Data consistency** - Direct mapping to existing clothing items
- **Smart analytics** - Accurate usage patterns vs manual logging errors
- **Better UX** - Simple photo capture workflow

## Key Database Flow Principles

### 1. **Seed Data First**: All lookup tables must be populated before user data
### 2. **State Transitions**: Never update states directly - always create new records and mark old as not current
### 3. **History Tracking**: Every state change gets recorded in state_history
### 4. **Context Separation**: Travel and home states are tracked separately via context field
### 5. **Referential Integrity**: All foreign keys must exist before insertion
### 6. **Soft Deletes**: Use is_deleted flags rather than actual deletion to preserve history

## Common Queries During App Usage

```sql
-- Get wardrobe summary
SELECT 
  ct.display_name as context,
  st.display_name as state,
  COUNT(*) as count
FROM clothing_items ci
JOIN wardrobe_contexts wc ON ci.id = wc.clothing_item_id
JOIN context_types ct ON wc.context_type_id = ct.id
JOIN clothing_states cs ON ci.id = cs.clothing_item_id
JOIN state_types st ON cs.state_type_id = st.id
WHERE ci.profile_id = 'user1-uuid' AND cs.is_current = true
GROUP BY ct.display_name, st.display_name;

-- Get outfit history
SELECT ol.date, STRING_AGG(ci.name, ', ') as items
FROM outfit_logs ol
JOIN outfit_items oi ON ol.id = oi.outfit_log_id
JOIN clothing_items ci ON oi.clothing_item_id = ci.id
WHERE ol.profile_id = 'user1-uuid'
GROUP BY ol.date
ORDER BY ol.date DESC;
```