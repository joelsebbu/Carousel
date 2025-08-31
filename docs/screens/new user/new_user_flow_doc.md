# Carousel App - New User Flow Documentation

## Overview
This document outlines the complete new user onboarding flow for the Carousel wardrobe management app, including the backup restore functionality for users setting up the app on a new device.

## Flow Trigger
This flow is initiated when:
- App is launched for the first time on a device
- No local profiles exist in the database
- User manually triggers "Add Profile" when no profiles exist

## Screen 1: Initial Welcome & Path Selection

### Screen Layout
```
┌─────────────────────────────────────┐
│         Welcome to Carousel         │
│    Your Personal Wardrobe Manager   │
│                                     │
│  Do you already use Carousel?       │
│                                     │
│  [📱 This is my first time]         │
│                                     │
│  [☁️ Restore my data]              │
│                                     │
└─────────────────────────────────────┘
```

### Visual Elements
- **Header**: App logo/icon centered
- **Title**: "Welcome to Carousel" - primary heading
- **Subtitle**: "Your Personal Wardrobe Manager" - secondary text
- **Question**: "Do you already use Carousel?" - body text
- **Primary CTA**: "This is my first time" button - prominent styling
- **Secondary CTA**: "Restore my data" button - secondary styling
- **Background**: Clean, welcoming design consistent with app branding

### Button Behavior
- **"This is my first time"** → Navigates to Profile Creation Screen
- **"Restore my data"** → Navigates to Backup Restore Screen

## Path A: First Time User

### Screen 2A: Profile Creation Screen

#### Screen Layout
```
┌─────────────────────────────────────┐
│        Set Up Your Profile          │
│                                     │
│  👤 [Camera Circle - Add Photo]     │
│                                     │
│  Name: [Text Input Field]           │
│  Relationship: [Self - Disabled]    │
│                                     │
│  [Create My Profile] ───────────────│
│                                     │
│  You can add family members later   │
└─────────────────────────────────────┘
```

#### Profile Photo Options
When user taps the camera circle, show action sheet:
- **Take Photo** - Opens camera interface
- **Choose from Library** - Opens photo library
- **Use Initials** - Generates colored circle with user's initials
- **Skip Photo** - Uses default avatar icon

#### Form Fields
- **Name**: Required text input, placeholder "Enter your name"
- **Relationship**: Fixed as "Self" for primary profile, field disabled
- **Validation**: Name required (minimum 1 character)

#### Actions
- **Primary CTA**: "Create My Profile" - creates profile and continues to wardrobe onboarding
- **Helper Text**: "You can add family members later" - reduces pressure to add everyone immediately

## Path B: Restore from Backup

### Screen 2B: Backup Restore Options

#### Screen Layout
```
┌─────────────────────────────────────┐
│         Restore Your Wardrobe       │
│                                     │
│  We'll restore your profiles,       │
│  clothing, and outfit history       │
│                                     │
│  [🔐 Connect to Google Drive]       │
│                                     │
│  Or restore from file:              │
│  [📁 Choose backup file]            │
│                                     │
│  ─────────────────────────────────   │
│  [← Back] [Skip - Start Fresh]     │
└─────────────────────────────────────┘
```

#### Content Elements
- **Title**: "Restore Your Wardrobe"
- **Description**: Explains what will be restored
- **Primary Option**: Google Drive connection - preferred method
- **Secondary Option**: Manual file selection - fallback option
- **Navigation**: Back button returns to welcome screen
- **Alternative**: "Skip - Start Fresh" goes to profile creation

#### Button Actions
- **"Connect to Google Drive"** → Initiates Google authentication flow
- **"Choose backup file"** → Opens file picker for .carousel files
- **"Back"** → Returns to Screen 1 (Welcome screen)
- **"Skip - Start Fresh"** → Goes to Screen 2A (Profile Creation)

### Screen 3B: Google Drive Authentication

#### Process Flow
1. Launch Google OAuth flow
2. Request permissions for Google Drive access
3. On success → Navigate to backup file selection
4. On failure → Show error and return to restore options

### Screen 4B: Backup File Selection

#### Google Drive Path
```
┌─────────────────────────────────────┐
│        Choose Backup File           │
│                                     │
│  Found backups in your Drive:       │
│                                     │
│  ○ Carousel_2025-08-30.backup      │
│    Last updated: Yesterday 11:47 PM │
│                                     │
│  ○ Carousel_2025-08-25.backup      │
│    Last updated: 5 days ago         │
│                                     │
│  [Restore Selected Backup] ─────────│
│  [← Back]                           │
└─────────────────────────────────────┘
```

#### Manual File Path
- Native file picker opens
- Filter for .carousel file extensions
- User selects backup file
- Continues to restore process

### Screen 5B: Restore Process

#### Loading Screen Layout
```
┌─────────────────────────────────────┐
│        Restoring Your Data...       │
│                                     │
│  ████████████░░░ 75%                │
│                                     │
│  ✓ Profiles restored                │
│  ✓ Clothing items restored          │
│  ⏳ Processing images...            │
│  ⏳ Restoring outfit history...     │
│                                     │
│  This may take a few minutes        │
└─────────────────────────────────────┘
```

#### Progress Steps
1. **Profiles restored** - User profiles and relationships
2. **Clothing items restored** - Wardrobe items and metadata
3. **Processing images** - Extracting and saving clothing photos
4. **Restoring outfit history** - Past outfit logs and state history

#### Technical Process
- Download and decrypt backup file
- Validate backup integrity
- Restore database records
- Extract and save image files
- Update progress indicators
- Handle any errors gracefully

### Screen 6B: Restore Success

#### Success Screen Layout
```
┌─────────────────────────────────────┐
│            Welcome Back!            │
│                                     │
│  Successfully restored:             │
│  • 3 profiles                       │
│  • 127 clothing items               │
│  • 45 days of outfit history        │
│                                     │
│  [Continue to Wardrobe] ────────────│
└─────────────────────────────────────┘
```

#### Content
- **Title**: "Welcome Back!" - celebratory tone
- **Summary**: Shows what was restored with specific counts
- **CTA**: "Continue to Wardrobe" - takes user to main wardrobe screen

## Error States

### No Backups Found
```
┌─────────────────────────────────────┐
│         No Backups Found            │
│                                     │
│  We couldn't find any Carousel      │
│  backups in your Google Drive       │
│                                     │
│  [Try Different Account]            │
│  [Choose File Manually]             │
│  [Start Fresh Instead]              │
└─────────────────────────────────────┘
```

### Authentication Failed
```
┌─────────────────────────────────────┐
│       Connection Failed             │
│                                     │
│  Couldn't connect to Google Drive   │
│  Please check your internet         │
│  connection and try again           │
│                                     │
│  [Try Again]                        │
│  [Choose File Manually]             │
│  [Start Fresh Instead]              │
└─────────────────────────────────────┘
```

### Corrupt Backup File
```
┌─────────────────────────────────────┐
│        Invalid Backup File          │
│                                     │
│  This backup file couldn't be       │
│  read. It may be corrupted or       │
│  from an incompatible version       │
│                                     │
│  [Try Different File]               │
│  [Contact Support]                  │
│  [Start Fresh Instead]              │
└─────────────────────────────────────┘
```

## Next Steps After Flow Completion

### After Profile Creation (Path A)
- Navigate to wardrobe onboarding flow
- Guide user through adding first clothing items

### After Successful Restore (Path B)  
- Navigate directly to main wardrobe screen
- User can immediately start using restored wardrobe

## Technical Considerations

### Data Validation
- Validate backup file format before processing
- Check app version compatibility
- Verify data integrity during restore

### Performance
- Show progress indicators for long operations
- Handle large image collections efficiently  
- Provide cancel option for restore process

### Privacy
- Encrypt backup files before upload
- Clear temporary files after restore
- Handle Google Drive permissions appropriately

### Offline Handling
- Detect network connectivity for Google Drive
- Provide appropriate error messages
- Fallback to manual file selection when offline