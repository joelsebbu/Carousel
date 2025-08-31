# Carousel App - Functional Specification

## Overview
Carousel is a personal wardrobe management app that helps users track their clothing, plan outfits, and manage laundry efficiently. The app supports multiple profiles for family use and handles different contexts like indoor/outdoor clothing and travel scenarios.

## Core Concept
- Build a digital wardrobe by uploading photos of each clothing item
- Track daily outfit usage through photo logging
- Manage clothing states: Available → Worn → Rack/Laundry → Available
- Separate wardrobes for different contexts and users

## Terminology
- **Wardrobe** - Collection of all clothing items
- **Clothing** - Individual items (shirts, pants, accessories, etc.)
- **Profile** - Individual user account within the app
- **Rack** - Temporary storage for items that can be worn again soon
- **Laundry** - Items that need washing before next wear

## Clothing States
1. **Available** - Ready to wear, in active rotation
2. **Worn Today** - Currently being worn (logged today)
3. **On Rack** - Worn but can be used again soon
4. **In Laundry** - Needs washing before next wear

## Clothing Properties
- **Type** - Categorizes items (Shirts, Pants, Underwear, Shoes, Accessories, etc.)
- **Wardrobe Context** - Indoor, Outdoor, or Shared between both
- **Current State** - Available, Worn Today, On Rack, In Laundry

## Wardrobe Structure

### Dual Wardrobe System
Each profile maintains separate wardrobes:
- **Indoor Wardrobe** - House clothes, loungewear, indoor shoes
- **Outdoor Wardrobe** - Work clothes, going-out clothes, outdoor shoes
- **Shared Items** - Items that appear in both wardrobes (glasses, some underwear, versatile pieces)

### Category-Specific Behaviors
- **Undergarments** - Default to Laundry after use
- **Outer Clothing** - User chooses between Rack or Laundry
- **Accessories** - May have special rules (watches always available, shoes rarely need washing)

## Daily Workflow
1. **Morning Planning** - View available clothing in chosen wardrobe context
2. **Outfit Logging** - Take photo of worn outfit to mark items as "Worn Today"
3. **End of Day** - Sort worn items to either Rack or Laundry
4. **State Management** - Move items back to Available from Rack or after washing

## Travel Mode
### Packing Process
- Select items from Indoor/Outdoor wardrobes to pack
- Packed items move to temporary **Travel Wardrobe**
- Home wardrobes show reduced available items

### Travel States
- **Available (Travel)** - Packed items ready to wear
- **Worn Today (Travel)** - Currently wearing during trip
- **Travel Rack** - Can be worn again during trip
- **Travel Laundry** - Need washing (but carrying during trip)

### Unpacking Process
Return items to home based on usage:
- **Unused items** → Back to original wardrobe (Available)
- **Travel Rack items** → Home Rack
- **Travel Laundry items** → Home Laundry

## Profile System
### Multi-User Support
- Create profiles for family members (Self, Spouse, Children, etc.)
- Each profile has complete wardrobe system (Indoor/Outdoor/Travel)
- Quick profile switching for managing different users

### Family Features
- **Laundry Day View** - Consolidated view of all family members' laundry
- **Family Packing** - Coordinate travel wardrobes when traveling together
- **Profile Dashboard** - Overview of each person's wardrobe status
- **Quick Stats** - Family-wide wardrobe summaries

### Profile-Specific Settings
- Age-appropriate clothing categories
- Different default behaviors per user
- Individual privacy settings

## Key Actions
- **Add Clothing** - Upload photos to build wardrobe
- **Log Outfit** - Photo logging of daily wear
- **Send to Rack** - Mark as reusable
- **Send to Laundry** - Mark as needing wash
- **Mark Clean** - Return laundry items to Available
- **Pack for Trip** - Move items to travel wardrobe
- **Unpack** - Process travel items back to home states

## Smart Features
- **Context Filtering** - Show relevant clothes based on Indoor/Outdoor selection
- **Category Defaults** - Intelligent suggestions based on clothing type
- **Shared Item Management** - Consistent state across wardrobe contexts
- **Travel Integration** - Seamless packing/unpacking workflows
- **Family Coordination** - Multi-user laundry and travel management

## Benefits
- Reduces morning decision fatigue
- Prevents over-wearing favorite items
- Organizes laundry planning
- Maximizes wardrobe utilization
- Handles real-world clothing management scenarios
- Supports family wardrobe coordination