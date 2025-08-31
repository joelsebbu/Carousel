# Carousel App - Technology Stack & Architecture Decisions

## Project Overview
Carousel is a personal wardrobe management mobile app for tracking clothing items, managing daily outfits, and organizing laundry efficiently. The app features multi-profile support for families and handles different contexts (indoor/outdoor) and travel scenarios.

**Key Requirements:**
- Personal use only (no need for enterprise-level scalability)
- Mobile-first application
- Offline-first with local data storage
- Daily automated backups to Google Drive
- Cost-effective (minimize ongoing expenses)
- Leverage existing React/JavaScript skills

## Technology Stack

### Mobile Framework: React Native + Expo
**Choice:** React Native with Expo managed workflow

**Why This Choice:**
- **Skill Leverage:** Builds directly on existing React and JavaScript expertise
- **Cross-Platform:** Single codebase for iOS and Android
- **Development Speed:** Expo provides pre-built components for camera, file system, notifications
- **Easy Testing:** QR code scanning for instant device testing
- **No Native Complexity:** Avoids Xcode/Android Studio setup complexity
- **Rich Ecosystem:** Access to extensive React Native community and libraries

**Alternative Considered:** Native iOS/Android development
**Why Rejected:** Would require learning Swift/Kotlin, maintaining separate codebases

### UI Framework: NativeWind (Tailwind for React Native)
**Choice:** NativeWind for styling

**Why This Choice:**
- **Familiar Syntax:** Uses same Tailwind classes as web development
- **Rapid Prototyping:** Utility-first approach speeds up UI development
- **Consistent Design:** Built-in design system prevents inconsistent styling
- **Small Bundle Size:** Only includes used styles
- **Mobile Optimized:** Designed specifically for React Native constraints

**Alternative Considered:** Styled Components, React Native Elements
**Why Rejected:** More verbose than utility classes, less familiar syntax

### State Management: Zustand
**Choice:** Zustand for global state management

**Why This Choice:**
- **Minimal Learning Curve:** Simple API using React hooks patterns
- **Tiny Bundle Size:** Only 1KB, perfect for mobile
- **No Boilerplate:** Unlike Redux, minimal setup required
- **TypeScript Ready:** Excellent TypeScript support out of the box
- **Perfect for App Scale:** Ideal for personal-use app complexity level

**Alternative Considered:** Redux Toolkit, React Context
**Why Rejected:** Redux too complex for personal app, Context causes re-render issues

### Database: SQLite + Drizzle ORM
**Choice:** SQLite database with Drizzle ORM

**Why SQLite:**
- **Offline First:** Works completely without internet connection
- **Mobile Optimized:** Designed for local storage on devices
- **Relational Support:** Handles complex relationships in wardrobe schema
- **ACID Compliance:** Ensures data integrity for clothing state transitions
- **Zero Cost:** No cloud database fees
- **Fast Queries:** Local database means instant response times

**Why Drizzle ORM:**
- **Type Safety:** Full TypeScript integration prevents database errors
- **Schema Evolution:** Automated migration generation for future feature additions
- **Lightweight:** Minimal runtime overhead compared to heavy ORMs
- **SQL Transparency:** Generated SQL is readable and optimizable
- **Migration Management:** Built-in version control for database schema changes

**Alternative Considered:** Prisma, raw SQLite, Supabase
**Why Rejected:** Prisma too heavy for mobile, raw SQL error-prone, Supabase requires internet

### Image Storage Strategy: Local File System
**Choice:** Expo FileSystem for local image storage

**Why This Choice:**
- **Offline Access:** Images load instantly without network requests
- **Cost Effective:** No cloud storage fees for frequently accessed images
- **Privacy:** Images never leave device except in encrypted backups
- **Performance:** No network latency for image loading
- **Simple Implementation:** Straightforward file path storage in database

**Image Processing Approach:**
- **Clothing Photos:** Stored permanently (compressed to ~200KB each)
- **Outfit Photos:** Processed for image recognition then discarded
- **No Redundant Storage:** Outfit history stored as clothing item references, not photos

### Backup Strategy: Google Drive API
**Choice:** Daily encrypted backups to Google Drive

**Why This Choice:**
- **Cost Effective:** Uses free 15GB Google Drive storage
- **Privacy:** Encrypted before upload protects personal data
- **Reliability:** Google Drive's 99.9% uptime ensures backup availability
- **Easy Restore:** Simple process to restore complete app state
- **Automated:** Daily 1 AM backups require no user intervention
- **Version History:** 30-day backup retention for recovery options

**Backup Architecture:**
- **Single File Approach:** Complete database + images in one encrypted file
- **Base64 Encoding:** Images converted to text for unified backup format
- **Automated Cleanup:** Old backups automatically deleted to manage storage

**Alternative Considered:** iCloud, Dropbox, AWS S3
**Why Rejected:** iCloud iOS-only, Dropbox limited free tier, AWS requires payment

## Architecture Decisions

### Local-First Data Architecture
**Decision:** Store all data locally with cloud backup only

**Rationale:**
- **Offline Functionality:** App works completely without internet
- **Performance:** Instant data access and updates
- **Privacy:** Personal wardrobe data stays on device
- **Cost Control:** No ongoing database hosting fees
- **Simplicity:** No API endpoints or server management needed

### Image Recognition Without Storage
**Decision:** Process outfit photos for recognition but don't store them

**Rationale:**
- **Storage Efficiency:** Eliminates 90% of potential photo storage
- **Privacy:** No daily outfit photos stored anywhere
- **Focus on Data:** Emphasizes clothing state tracking over photo collection
- **Backup Size:** Keeps backups manageable for Google Drive free tier
- **Performance:** Faster backups with less data to process

### Single User Development Model
**Decision:** Optimize for personal use rather than multi-user scalability

**Rationale:**
- **Simplified Architecture:** No user authentication or data isolation needed
- **Faster Development:** Can make assumptions about usage patterns
- **Cost Optimization:** No need for scalable infrastructure
- **Feature Focus:** Can prioritize personal workflow over general use cases

### Mobile-Only Approach
**Decision:** Native mobile app without web version

**Rationale:**
- **Camera Integration:** Outfit logging requires mobile camera functionality
- **Daily Usage Pattern:** Wardrobe management is naturally mobile-centric
- **Offline Requirements:** Mobile apps better suited for offline-first architecture
- **Development Focus:** Single platform allows deeper feature development

## Development Workflow Benefits

### Rapid Prototyping
- **Hot Reload:** Instant updates during development with Expo
- **Type Safety:** Compile-time error catching with TypeScript + Drizzle
- **Familiar Tools:** React DevTools and debugging experience

### Feature Evolution
- **Schema Migrations:** Drizzle Kit handles database changes automatically
- **Component Reusability:** React patterns enable rapid UI development
- **State Management:** Zustand simplifies complex state updates

### Testing Strategy
- **Device Testing:** Expo QR codes for instant real-device testing
- **Data Safety:** Local backups allow safe experimentation
- **Rollback Capability:** Google Drive backups provide safety net

## Cost Analysis

### Development Costs
- **React Native + Expo:** Free
- **All Libraries:** Open source, no licensing fees
- **Google Drive API:** Free tier sufficient for personal use
- **App Store Publishing:** $99/year iOS + $25 one-time Android

### Ongoing Operational Costs
- **Database Hosting:** $0 (local SQLite)
- **Image Storage:** $0 (local file system)
- **Backup Storage:** $0 (Google Drive free tier)
- **API Costs:** $0 (no external APIs needed)
- **Total Monthly Cost:** $0

### Long-term Scalability
- **Storage Growth:** Predictable (only new clothing items)
- **Backup Size:** Manageable within free tiers for years
- **Performance:** Local data ensures consistent speed regardless of wardrobe size

## Risk Mitigation

### Data Loss Prevention
- **Daily Automated Backups:** Minimize data loss window
- **Multiple Retention:** 30 days of backup versions
- **Local + Cloud:** Dual storage strategy
- **Encryption:** Protects backups if Google account compromised

### Technology Obsolescence
- **React Native Longevity:** Backed by Meta, strong ecosystem
- **SQLite Stability:** Decades-proven database technology
- **Standard APIs:** Uses platform-standard file systems and APIs
- **Migration Path:** Drizzle enables easy schema evolution

### Development Continuity
- **Skill Alignment:** Builds on existing React/JavaScript expertise
- **Documentation:** All technologies have excellent documentation
- **Community Support:** Large communities for troubleshooting
- **Personal Control:** No dependence on external services for core functionality

## Future Extension Possibilities

### Potential Feature Additions
- **Smart Recommendations:** AI-based outfit suggestions
- **Weather Integration:** Weather-appropriate clothing suggestions
- **Social Features:** Family member coordination for shared items
- **Analytics:** Wardrobe utilization statistics and insights

### Technical Extensibility
- **API Integration:** Easy to add external APIs for weather, fashion trends
- **Machine Learning:** Could integrate TensorFlow.js for advanced image recognition
- **Cloud Sync:** Could add real-time sync between family devices if needed
- **Export Options:** Data easily exportable to other formats

This technology stack provides a solid foundation for personal wardrobe management while maintaining simplicity, cost-effectiveness, and alignment with existing development skills.