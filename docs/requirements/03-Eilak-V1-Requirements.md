# Eilak - V1 Product Requirements

**Document:** 03
**Version:** 2.0
**Last updated:** 2026-11-22
**Status:** Consolidated requirements baseline

---

## 1. Product definition

Eilak is an Android application for managing real-estate listings and customers.

The initial product is local-first and offline-capable.

The system is designed around Listing as the central business record and Customer as the person record.

The application must support structured listing information, customer relationships, media, notes, search, filtering, sharing and backup/restore.

## 2. Scope boundaries

### 2.1 Broader product classification

The broader product classification includes five main categories and thirteen subcategories.

The complete classification is documented in:

`04-Eilak-Categories-Subcategories-Fields.md`

### 2.2 End-to-end MVP

The initial end-to-end implementation supports four combinations:

1. Apartment Sale
2. Apartment Rent
3. HouseVilla Sale
4. HouseVilla Rent

This MVP boundary does not redefine or remove the broader product categories.

The remaining categories and scenarios are retained for future implementation.

### 2.3 Architecture constraints

- Listing is the central entity.
- Property is not a separate V1 entity or table.
- PropertyType and TransactionType are independent.
- Category-specific data and transaction-specific data are separate.
- Dynamic Listing-level values are stored through the approved metadata model.
- Domain logic must not depend on Android UI or persistence implementation details.

## 3. Application architecture

The intended separation is:

1. Presentation
2. Application
3. Domain
4. Data
5. Infrastructure

Responsibilities:

### Presentation

- Screens and navigation.
- Forms and field controls.
- Display formatting.
- Input interaction and validation feedback.

### Application

- User workflows.
- Use-case orchestration.
- Save and restore operations.
- Lifecycle transitions.

### Domain

- Business entities and rules.
- Validation semantics.
- Relationship behavior.
- Search/filter semantics.

The Domain layer must not depend on Android, Room, Contacts APIs or UI components.

### Data and Infrastructure

- Persistence.
- Repository implementations.
- Media file access.
- Contacts integration.
- Backup and restore implementation.

The inspected project does not yet confirm a Room implementation. The persistence technology must be selected and integrated separately.

## 4. Listing management

The application must support:

- Creating a Listing.
- Viewing a Listing.
- Editing a Listing.
- Archiving a Listing.
- Restoring an archived Listing.
- Moving a Listing to trash.
- Restoring a trashed Listing.
- Searching Listings.
- Filtering Listings.
- Sharing a Listing.
- Managing Listing media.
- Managing Listing notes.
- Connecting Customers.

A new Listing creation workflow must create the corresponding category-specific and transaction-specific records.

The UI must not allow an independent Property record or independent category record.

### 4.1 Required common information

The Listing model includes:

- PropertyType
- TransactionType
- Province
- City
- YearBuilt where applicable
- Description
- InternalNote
- Neighborhood
- Address
- MarkerColor
- CreatedAt
- UpdatedAt
- IsActive

The exact Required/Optional behavior of individual form fields must be documented per form. It must not be inferred from similarity between forms.

### 4.2 Description and private note

Description may be included in customer-facing sharing.

InternalNote is private and must never be shared.

V1 supports one current InternalNote without history.

## 5. Listing lifecycle

The user-facing lifecycle includes:

- Active
- Archived
- Trash

### 5.1 Archive

- Confirmation is required.
- Archived Listings remain viewable.
- Archived Listings may be edited.
- New Customer relationships cannot be added to archived Listings.
- Media is retained.

### 5.2 Trash

- Confirmation is required when moving an item to trash.
- Trash records are read-only except for restore.
- Media is retained.
- Permanent Listing deletion is outside V1.

### 5.3 Restore

- Restore does not require an additional confirmation.
- Relationships are preserved.
- Effective relationship activity follows the lifecycle of both sides.

## 6. Listing forms

The form model has four conceptual field groups:

1. Common Listing fields.
2. Property-category fields.
3. Transaction-type fields.
4. Category-transaction intersection fields.

The form must show only fields applicable to the selected combination.

### 6.1 Input rules

- Required fields must be validated.
- Optional fields may remain empty.
- Empty, zero and false are different values.
- Numeric fields must not be stored as formatted text.
- Phone numbers must be strings.
- Enum values must come from approved options.
- MultiSelect values must be stored as selections, not concatenated display text.
- User-facing labels are Persian.
- Internal enum identifiers use stable technical values.

### 6.2 Save behavior

- Save is explicit.
- Autosave is not used.
- All validation errors are collected and presented together.
- Invalid input must remain available for correction.
- A successful save updates UpdatedAt.
- CreatedAt must not change after creation.

## 7. Customer management

The application must support:

- Manual Customer creation.
- Customer creation from device contacts.
- Viewing Customer.
- Editing Customer.
- Archiving Customer.
- Restoring Customer.
- Moving Customer to trash.
- Restoring Customer from trash.
- Customer notes.
- Listing relationships.
- Loyalty membership.
- Calling and SMS handoff.

### 7.1 Customer identity

- Name is required.
- Mobile number is required.
- Customer has a stable internal ID.
- Mobile numbers are strings.
- One Customer has one primary mobile number in V1.
- Normalized phone is unique across active, archived and trashed Customers.
- Archiving or soft-deleting does not release the phone number.
- Automatic merging is prohibited.

### 7.2 Phone normalization

Normalization must account for:

- Persian digits.
- Arabic digits.
- English digits.
- Spaces.
- Hyphens.
- Formatting differences.

The same logical phone number must not be treated as multiple customers merely because its display formatting differs.

### 7.3 Restore collision

If a Customer cannot be restored because another record owns the normalized phone number, the application must not silently overwrite or merge records.

The exact collision-resolution workflow remains open.

## 8. Contacts integration

The application may import a name and phone number from Android contacts.

The device contact name and Eilak customer name remain independent.

Display-name priority:

1. Contact name when available and permission allows.
2. Eilak customer name.
3. Phone number.

A device contact may be linked to at most one Customer.

Changing or deleting a device contact must not delete or corrupt the Eilak Customer.

Revoking Contacts permission must not make Customer records unusable.

## 9. Customer loyalty

The application supports loyal-customer membership.

The user can toggle membership.

Archiving or soft-deleting a Customer does not automatically remove loyalty membership.

Removing a Customer from the loyal list removes membership only; it does not delete the Customer.

The physical implementation must preserve these semantics.

## 10. Listing-Customer relationships

A Listing may be associated with multiple Customers.

A Customer may be associated with multiple Listings.

Only one relationship may exist for a given Listing-Customer pair.

### 10.1 Owner

The owner is a Customer with the `Owner` role in ListingCustomer.

The owner's mobile number is mandatory when creating a Listing.

### 10.2 Relationship eligibility

Only an active Customer may be newly connected to an active Listing.

### 10.3 Lifecycle interaction

Archiving or soft-deleting either side does not delete the relationship.

The relationship's effective active state depends on both entities.

Restoring both entities makes the preserved relationship eligible to become active again.

Manual disconnection is allowed.

Disconnecting the relationship deletes its relationship note, but does not delete independent notes on either entity.

## 11. Media management

Media files are stored outside the database.

The database stores metadata and file references.

The application supports multiple photos and videos per Listing.

At most one media item is the cover.

The cover may be either a photo or a video.

If the current cover is deleted, another eligible item is selected automatically when available.

Archiving or moving a Listing to trash must not delete media files.

### 11.1 Sharing

Sharing includes:

- All registered Listing fields.
- All photos.
- Photo ordering.

Sharing excludes:

- Videos.
- InternalNote.
- Private information not intended for customers.

Android Sharesheet is used.

## 12. Notes

There are three note scopes:

1. Listing notes.
2. Customer notes.
3. ListingCustomer relationship note.

Listing and Customer may each have multiple independent notes.

A ListingCustomer relationship may have at most one note.

Note text is mandatory. Title is optional.

Notes do not have a trash lifecycle.

Deleting a note is permanent and requires confirmation.

Deleting a relationship note must not delete independent Listing or Customer notes.

## 13. Search

Search must support Persian text normalization.

Normalization includes:

- ي to ی.
- ك to ک.
- Persian, Arabic and English digits.
- Relevant spaces and half-space differences.

Search supports partial matching.

When a query contains multiple terms, all terms must match.

Numeric fields must also be searchable through their textual numeric representation.

Internal IDs are not included in ordinary text search.

## 14. Filtering

Supported conceptual filter dimensions include:

- Property category/subcategory.
- City.
- Price.
- Area.

Within a single filter, multiple selected values use OR.

Across independent filters, conditions use AND.

For Rent:

- DepositAmount is an independent filter.
- MonthlyRent is an independent filter.

Construction partnership must not be forced into the generic price filter.

Only filters relevant to the implemented form combination should be shown.

## 15. Listing details

The detail screen uses expandable/accordion sections.

Conceptual sections:

1. Main information.
2. Category-specific information.
3. Transaction-specific information.
4. Media.
5. Customers.
6. Listing notes.
7. Creation and update information.

The private InternalNote must be visually and semantically distinguished from customer-facing Description.

## 16. Editing

The application supports complete and partial edits.

Both edit modes must use consistent validation and save rules.

Only one edit workflow may be active for a Listing at a time.

Failed validation must preserve user-entered values.

A successful edit updates UpdatedAt.

## 17. Home screen

The home screen includes:

- Add Listing action.
- Search.
- Five recently created or updated Listings according to the agreed recent-list rule.
- Customer access.
- Loyal-customer access.

The home screen is not an analytics dashboard.

## 18. Communication

Calling and SMS actions open the corresponding Android application.

The user performs the final call or message action.

The application does not implement in-app calling or SMS.

WhatsApp integration and call-log functionality are outside V1.

## 19. UI and localization

- UI language is Persian.
- Layout direction is RTL.
- Persian digits are used for appropriate user-facing numeric display.
- Shamsi date display is required.
- User-facing strings are maintained through Android string resources.
- Theme, palette and font require a separate approved design decision.
- Technical identifiers remain stable and independent of Persian display labels.

### 19.1 Optional selection controls

Optional selection fields can be cleared.

Required selection fields cannot be cleared.

MultiSelect controls support:

- Removing individual options.
- Clear All.

The behavior should be implemented through reusable shared components.

### 19.2 Count display

Selected count fields display single-digit values in Persian words in public-facing contexts.

Values 10 and above display numerically.

Database values and form inputs remain numeric.

The rule excludes:

- Price.
- Area.
- Year.
- Mobile.
- Identifiers.

It includes ToiletCount and BathroomCount.

## 20. Backup and restore

Backup and restore are explicitly user-triggered.

Backup includes:

- Database.
- Media.
- Settings.

Every backup operation creates a new backup file.

Restore is a full restore.

The backup must represent a consistent snapshot.

An incomplete or failed restore must not destroy the user's existing data.

The following details remain open:

- File format.
- Encryption.
- Destination.
- File naming.
- Version compatibility.
- Recovery behavior.

## 21. Performance and reliability

The application must remain usable offline.

Persistence operations must avoid partial business records.

A Listing creation operation must not leave an incomplete category/transaction combination without an explicit recovery mechanism.

Media file operations and database metadata operations must be coordinated so that failed operations do not silently orphan data.

The implementation must handle application interruption and process recreation safely.

## 22. Acceptance criteria for MVP

The MVP is acceptable when:

1. A user can create an Apartment Sale Listing.
2. A user can create an Apartment Rent Listing.
3. A user can create a HouseVilla Sale Listing.
4. A user can create a HouseVilla Rent Listing.
5. Each Listing has the correct category-specific record.
6. Each Listing has the correct transaction-specific record.
7. Listing has no independent Property entity.
8. Owner is represented through Customer and ListingCustomer.
9. Listing and Customer lifecycle behavior preserves relationships.
10. Listing media is retained across archive and trash.
11. Photos can be shared in order and videos are excluded.
12. Search normalization works for Persian text and digits.
13. Filters combine with the specified OR/AND semantics.
14. Explicit save and aggregate validation work.
15. Private InternalNote is excluded from sharing.
16. Backup and restore do not silently destroy existing data on failure.
17. The app remains usable without network access.

The thirteen-subcategory product classification is not an assertion that all thirteen forms are implemented in MVP.

## 23. Out of scope

- Advanced customer need profiles.
- Matching engine.
- Visit timeline.
- Negotiation timeline.
- Transaction timeline.
- Advanced reminders.
- Unattached notes.
- Activity ranking.
- Saved filters.
- Bulk operations.
- Permanent Listing deletion.
- Short-term rent.
- In-app calling and SMS.
- Final buyer identification.
- WhatsApp integration.

## 24. Open requirements

1. Exact Required/Optional matrix for all thirteen forms.
2. Exact display order and grouping for all thirteen forms.
3. Exact DeedStatus options.
4. Customer restore collision workflow.
5. Duplicate-listing detection algorithm and threshold.
6. Contact synchronization debounce.
7. Backup format and encryption.
8. Font, theme and palette.
9. Lifecycle timestamp semantics.
10. Physical persistence technology.
11. Full field specification for non-MVP categories.
12. Exact storage for optional descriptions associated with dynamic Other options.
