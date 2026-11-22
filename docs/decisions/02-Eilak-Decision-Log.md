# Eilak - Architecture and Product Decision Log

**Document:** 02
**Version:** 2.0
**Last updated:** 2026-11-22
**Status:** Consolidated decisions

---

## 1. Purpose

This document records decisions that constrain the architecture, domain model and V1 implementation.

A decision marked Confirmed must not be reopened without new evidence or an explicit change request.

A decision marked Open is not to be completed by assumption.

## 2. Decision status definitions

| Status | Meaning |
|---|---|
| Confirmed | Explicitly decided and approved |
| Constrained | Direction is known, but implementation details remain |
| Open | No reliable final decision is available |
| Deferred | Intentionally outside current implementation scope |
| Superseded | Replaced by a newer decision |

## 3. Confirmed architecture decisions

### DEC-001 - Listing is the central entity

**Status:** Confirmed

`Listing` is the central business entity.

The `Property` entity and table are removed from V1.

There is no independent Property creation workflow.

### DEC-002 - No PropertyId

**Status:** Confirmed

No `PropertyId` is introduced into the V1 model.

Category and transaction records reference `ListingId`.

### DEC-003 - Independent classification dimensions

**Status:** Confirmed

`PropertyType` and `TransactionType` are independent dimensions.

They must not be merged into a single combined type.

### DEC-004 - Official HouseVilla identifier

**Status:** Confirmed

The official category identifier is `HouseVilla`, not `Villa`.

### DEC-005 - Six category tables

**Status:** Confirmed

The official category tables are:

- Apartment
- HouseVilla
- LandOldBuilding
- Office
- Shop
- IndustrialCommercial

Each uses `ListingId` as its one-to-one key and reference to Listing.

### DEC-006 - Category record creation

**Status:** Confirmed

A category record is created only within the workflow for creating a new Listing.

Independent category-record creation is not exposed in V1.

### DEC-007 - Four conceptual form groups

**Status:** Confirmed

Form fields belong to one of four conceptual groups:

1. Common Listing fields.
2. Property-category fields.
3. Transaction-type fields.
4. Category-transaction intersection fields.

The intersection group represents fields specific to a combination such as ApartmentSale.

### DEC-008 - Dynamic field metadata

**Status:** Confirmed

The conceptual metadata/value model uses:

- `FieldDefinition`
- `FieldOption`
- `ListingFieldValue`
- `ListingFieldValueOption`

The application has no management interface for editing these definitions or options.

### DEC-009 - Owner is a Customer

**Status:** Confirmed

The owner is represented by a Customer.

A separate Owner entity is not introduced.

### DEC-010 - Owner relationship

**Status:** Confirmed

The owner is connected to a Listing through `ListingCustomer` with the `Owner` role.

### DEC-011 - Owner mobile is mandatory

**Status:** Confirmed

The owner's mobile number is required when creating a Listing.

### DEC-012 - Loyalty

**Status:** Confirmed at product behavior level

Customer loyalty is represented as membership in the loyal-customer set.

Archiving or deleting a customer does not automatically remove loyalty membership.

The physical representation must preserve this behavior.

### DEC-013 - Six official categories

**Status:** Confirmed

The official category identifiers are:

- Apartment
- HouseVilla
- LandOldBuilding
- Office
- Shop
- IndustrialCommercial

### DEC-014 - No Details suffix

**Status:** Confirmed

Category table names do not use a `Details` suffix.

### DEC-015 - Renovation field

**Status:** Confirmed

`RenovationStatus` is removed.

The field is `HasBeenRenovated`, a nullable Boolean.

### DEC-016 - Province and City reference tables

**Status:** Confirmed

Province and City are reference entities.

City must belong to the selected Province.

### DEC-017 - Coordinates excluded

**Status:** Confirmed

Geographical coordinates are excluded from the current Listing model.

### DEC-018 - InternalNote

**Status:** Confirmed

Listing has one current `InternalNote`.

It is private, not customer-facing and has no history in V1.

### DEC-019 - Four end-to-end MVP combinations

**Status:** Confirmed

The MVP implements:

- Apartment + Sale
- Apartment + Rent
- HouseVilla + Sale
- HouseVilla + Rent

This does not limit the future conceptual architecture.

### DEC-020 - No automatic conversion of old transaction scenarios

**Status:** Confirmed

The thirteen legacy subcategories must not automatically become thirteen TransactionType values.

Construction partnership and presale require separate domain decisions.

## 4. Listing decisions

### DEC-021 - Listing identity

**Status:** Confirmed

`ListingId BIGINT IDENTITY(100,1)` is the conceptual identifier.

### DEC-022 - Required classification

**Status:** Confirmed

`PropertyTypeId` and `TransactionTypeId` are mandatory.

### DEC-023 - YearBuilt ownership

**Status:** Confirmed

`YearBuilt` belongs to Listing.

### DEC-024 - Lifecycle flag

**Status:** Confirmed

`IsActive` is the active/inactive flag in the V1 Listing model.

`Status` and `PublicationStatus` are not introduced as alternative Listing flags.

Archive and trash lifecycle behavior must still be represented in a way consistent with the product requirements.

### DEC-025 - Timestamps

**Status:** Confirmed

`CreatedAt` and `UpdatedAt` are UTC timestamps.

`CreatedAt` is immutable.

`UpdatedAt` is updated after every successful edit.

At creation, both timestamps are equal.

### DEC-026 - Description and InternalNote

**Status:** Confirmed

- `Description` is customer-facing.
- `InternalNote` is private.

### DEC-027 - Listing location

**Status:** Confirmed

Listing stores ProvinceId, CityId, Neighborhood and Address.

Coordinates are excluded.

### DEC-028 - MarkerColor

**Status:** Confirmed

`MarkerColor TINYINT NULL` represents five fixed UI colors.

Values 1-5 map to the approved palette. NULL means no color.

### DEC-029 - ListingCode

**Status:** Confirmed

`ListingCode` is excluded.

## 5. Dynamic field and option decisions

### DEC-030 - Seeded immutable options

**Status:** Confirmed

Options are seeded by application code.

There is no runtime management interface for creating, editing or deleting options.

### DEC-031 - Versioned idempotent seed

**Status:** Confirmed

Seed logic must be idempotent and version-aware.

It must not create duplicate options or overwrite user data unexpectedly.

### DEC-032 - CabinetMaterial

**Status:** Confirmed

MultiSelect options:

- MDF
- HighGloss
- Membrane
- Metal
- Wood

No Other option and no dependent explanation field.

Stored at Listing level.

### DEC-033 - SuitableFor

**Status:** Confirmed

MultiSelect options:

- Family
- Event
- MultipleFamilies

Displayed in HouseVillaRent but stored at Listing level.

### DEC-034 - CoolingSystem

**Status:** Confirmed

MultiSelect, Listing-level, optional.

Options:

- کولر آبی
- کولر گازی
- اسپلیت
- چیلر
- فن‌کوئل
- داکت‌اسپلیت
- سایر

### DEC-035 - HeatingSystem

**Status:** Confirmed

MultiSelect, Listing-level, optional.

Options:

- پکیج
- موتورخانه
- بخاری گازی
- شوفاژ
- گرمایش از کف
- هیتر
- شومینه
- سایر

### DEC-036 - HotWaterProvider

**Status:** Confirmed

MultiSelect, Listing-level, optional.

Options:

- پکیج
- موتورخانه
- آبگرمکن دیواری
- آبگرمکن ایستاده
- آبگرمکن خورشیدی
- سایر

### DEC-037 - Other explanation

**Status:** Open

The exact storage structure for an optional explanation associated with Other in CoolingSystem, HeatingSystem and HotWaterProvider is not finalized.

No extra table or column should be introduced without a separate decision.

### DEC-038 - Empty dynamic values

**Status:** Confirmed

An empty optional dynamic field means unknown/not entered.

It does not mean that the facility does not exist.

## 6. Apartment decisions

### DEC-039 - Apartment field set

**Status:** Confirmed

The approved fixed field set is:

- UnitArea
- BedroomCount
- FloorNumber
- BuildingFloorCount
- BuildingUnitCount
- UnitsPerFloor
- UnitDirection
- FloorMaterial
- KitchenType
- ToiletType
- ToiletCount
- BathroomCount
- HasElevator
- HasParking
- ParkingCount
- HasStorage
- HasBalcony
- BuildingDirection
- BuildingFacade
- HasBeenRenovated

CabinetMaterial, CoolingSystem, HeatingSystem and HotWaterProvider are Listing-level dynamic fields, not Apartment columns.

### DEC-040 - UnitArea

**Status:** Confirmed

`UnitArea DECIMAL(10,2) NOT NULL`, greater than zero.

It represents useful interior area only.

### DEC-041 - Apartment parking

**Status:** Confirmed

`HasParking` means dedicated parking assigned to the unit.

`ParkingCount` is NULL if HasParking is false or unknown.

If HasParking is true, ParkingCount is optional and must be positive when entered.

### DEC-042 - ToiletType dependency

**Status:** Confirmed

If ToiletCount is NULL or zero, ToiletType must be NULL.

If ToiletCount is greater than zero, ToiletType is optional.

### DEC-043 - Apartment direction

**Status:** Confirmed

UnitDirection and BuildingDirection are independent.

### DEC-044 - Floor semantics

**Status:** Confirmed

FloorNumber may be negative, zero or positive.

Zero represents the ground floor.

A positive floor number must not exceed BuildingFloorCount when the latter is known.

### DEC-045 - Apartment enum values

**Status:** Confirmed

ToiletType: Iranian, Western, Both.

Direction: North, South, East, West.

FloorMaterial: Ceramic, Stone, Parquet, Laminate, Mosaic, Other.

KitchenType: Open, Closed, SemiOpen.

BuildingFacade: Stone, Brick, Cement, Composite, Glass, Other.

## 7. HouseVilla decisions

### DEC-046 - HouseVilla field set

**Status:** Confirmed

The approved fixed fields are:

- LandArea
- BuildingArea
- BedroomCount
- BuildingFloorCount
- HasYard
- HasGarden
- HasPool
- PoolType
- HasSauna
- HasJacuzzi
- HasParking
- HasStorage
- HasBalcony
- HasTerrace
- BuildingDirection
- BuildingFacade
- FloorMaterial
- HasBeenRenovated
- HasFireplace
- HasAirConditioning
- HasSecurityGuard
- HasCCTV
- HasRemoteControlledGate
- HasIntercom
- HasSmartHomeSystem

### DEC-047 - HouseVilla area

**Status:** Confirmed

LandArea and BuildingArea are positive integers.

BuildingArea is the footprint of the building and must not exceed LandArea.

### DEC-048 - Pool semantics

**Status:** Confirmed

HasPool refers only to a pool usable for swimming.

PoolType is OpenAir or Covered.

PoolType may be specified only when HasPool is true.

### DEC-049 - Independent facilities

**Status:** Confirmed

HasSauna and HasJacuzzi are independent of HasPool.

HasYard and HasGarden are independent.

HasBalcony and HasTerrace are independent.

### DEC-050 - No HouseVilla ParkingCount

**Status:** Confirmed

ParkingCount is excluded from HouseVilla.

### DEC-051 - HouseVilla direction and enums

**Status:** Confirmed

BuildingDirection: North, South, East, West.

BuildingFacade: Stone, Brick, Cement, Composite, Glass, Other.

FloorMaterial: Ceramic, Stone, Parquet, Laminate, Mosaic, Other.

### DEC-052 - HouseVilla security

**Status:** Confirmed

The following fields remain independent:

- HasSecurityGuard
- HasCCTV
- HasRemoteControlledGate
- HasIntercom
- HasSmartHomeSystem

HasIntercom means video intercom only.

### DEC-053 - Fireplace and air conditioning

**Status:** Confirmed

HasFireplace and HasAirConditioning remain fixed HouseVilla fields even though dynamic HeatingSystem and CoolingSystem exist.

## 8. Transaction decisions

### DEC-054 - ApartmentSale

**Status:** Confirmed

Fields:

- TotalPrice
- DeedStatus
- IsExchangeable
- HasLoan

`LoanAmount` is excluded.

### DEC-055 - DeedStatus structure

**Status:** Confirmed at conceptual level; options open

DeedStatus is one single-choice field.

The selected option combines deed type and transfer condition.

The concept must not be split into separate deed-type and transfer-readiness fields without a new decision.

The exact option list is not reliably recoverable from the currently inspected sources and must not be invented.

### DEC-056 - ApartmentRent

**Status:** Confirmed

Fields:

- DepositAmount
- MonthlyRent

Both are optional positive monetary values when specified.

### DEC-057 - HouseVillaSale

**Status:** Confirmed

Sale-specific data includes TotalPrice and DeedStatus.

### DEC-058 - HouseVillaRent

**Status:** Confirmed

Fields:

- DepositAmount
- MonthlyRent
- PetsAllowed
- Furnished

SuitableFor is a Listing-level dynamic field, not a HouseVillaRent column.

## 9. Customer and relationship decisions

### DEC-059 - Customer identity

**Status:** Confirmed

Customer has a stable identifier, required name and required mobile number.

### DEC-060 - Phone uniqueness

**Status:** Confirmed

Normalized phone uniqueness applies across active, archived and trashed customers.

Archive and soft delete do not release a phone number.

### DEC-061 - No automatic merge

**Status:** Confirmed

Customers are never merged automatically.

### DEC-062 - Contacts independence

**Status:** Confirmed

The device contact name and Eilak customer name are separate.

Loss of device contact access must not destroy the Eilak customer record.

### DEC-063 - ListingCustomer relationship

**Status:** Confirmed

Customer and Listing have a many-to-many relationship.

A pair may have only one relationship.

### DEC-064 - Relationship lifecycle

**Status:** Confirmed

Archiving or soft-deleting either side does not delete the relationship.

Only active customers can be newly connected to active listings.

Restore must preserve and reactivate eligible relationships.

### DEC-065 - Relationship note

**Status:** Confirmed

At most one note belongs to a ListingCustomer relationship.

Disconnecting the relationship deletes that note.

Independent notes remain untouched.

## 10. Media and notes decisions

### DEC-066 - External media files

**Status:** Confirmed

Media files are stored outside the database. Database records contain metadata and paths/references.

### DEC-067 - One cover

**Status:** Confirmed

At most one media item is the cover. It may be a photo or video.

### DEC-068 - Cover replacement

**Status:** Confirmed

Removing the current cover causes another eligible media item to be selected automatically when available.

### DEC-069 - Media retention

**Status:** Confirmed

Archiving or soft-deleting a Listing does not delete media files.

### DEC-070 - Sharing media

**Status:** Confirmed

Sharing includes all photos in order and excludes videos.

### DEC-071 - Three note scopes

**Status:** Confirmed

Notes exist independently for Listing, Customer and ListingCustomer.

Listing and Customer may have multiple independent notes.

ListingCustomer has at most one relationship note.

### DEC-072 - Note deletion

**Status:** Confirmed

Note deletion is permanent and requires confirmation.

Notes do not have a trash lifecycle.

## 11. UI and data conventions

### DEC-073 - Clearable optional selections

**Status:** Confirmed

Optional selection fields can be cleared.

Required selection fields cannot be cleared.

MultiSelect supports removing individual selections and Clear All.

A shared component should implement this behavior.

### DEC-074 - Count display

**Status:** Confirmed

Selected count fields use Persian words for single-digit values in public-facing display.

Values 10 and above are displayed numerically.

Database values and form inputs remain numeric.

Price, area, year, mobile number and identifiers are excluded.

ToiletCount and BathroomCount are included.

### DEC-075 - Null semantics

**Status:** Confirmed

NULL means unknown/not entered where nullable fields are used.

NULL must not be silently converted to zero or false.

### DEC-076 - Persian search normalization

**Status:** Confirmed

Search normalizes:

- ي to ی
- ك to ک
- Persian, Arabic and English digits
- Relevant spacing and half-space differences

Partial matching is supported.

For multiple search words, all terms must match.

Numeric fields must also be searchable through their textual numeric representation.

Internal identifiers are excluded from ordinary text search.

### DEC-077 - Search filters

**Status:** Confirmed

Multiple values within one filter use OR.

Independent filters use AND.

Rent uses separate Deposit and MonthlyRent filters.

Construction partnership is not forced into the general price filter.

### DEC-078 - Explicit save

**Status:** Confirmed

Forms use explicit save.

Autosave is not used.

Validation errors are collected and displayed together.

Failed validation must not destroy entered data.

## 12. Product requirements decisions

### DEC-079 - Local-first

**Status:** Confirmed

V1 is local-first and offline-capable.

### DEC-080 - Listing lifecycle

**Status:** Confirmed

Active, archive and trash behavior must follow the lifecycle requirements.

Permanent Listing deletion is outside V1.

### DEC-081 - Sharing

**Status:** Confirmed

Android Sharesheet is used.

Sharing includes registered fields and photos, but excludes private InternalNote and videos.

### DEC-082 - Contacts and communication

**Status:** Confirmed

Contact selection is supported.

Calling and SMS open the relevant Android application. The user performs the final action.

In-app calling, SMS handling and WhatsApp integration are outside V1.

### DEC-083 - Backup and restore

**Status:** Confirmed at behavior level

Backup and restore are explicitly user-triggered.

Backup includes database, media and settings.

A new backup file is created for each operation.

Restore must be consistent and must not destroy existing data when an incomplete restore occurs.

Format, encryption and destination remain open.

## 13. Open decisions

The following items must not be guessed:

1. Exact DeedStatus option list.
2. Required/Optional matrix for every field in all thirteen subcategories.
3. Exact display order of every field in all thirteen forms.
4. Full field matrices for non-MVP categories.
5. Storage of explanatory text for dynamic Other options.
6. Restore collision workflow for Customer phone uniqueness.
7. Backup format and encryption.
8. Exact font, theme and palette.
9. Duplicate-listing algorithm and threshold.
10. Contact synchronization debounce.
11. Lifecycle timestamps and restore semantics.
12. Physical database engine and persistence implementation.
13. Cascade deletion policies.
14. Final physical representation of loyalty membership and relationship status.

## 14. Deferred scope

The following are outside the initial V1 implementation unless separately approved:

- Advanced customer need profiles.
- Automatic property/customer matching engine.
- Visit and negotiation timeline.
- Transaction timeline.
- Advanced reminders.
- Unattached notes.
- Activity ranking.
- Saved filters.
- Bulk operations.
- Permanent Listing deletion.
- Short-term rental.
- In-app calling and SMS.
- Final buyer identification workflow.
- WhatsApp integration.

## 15. Change control

A confirmed decision may be changed only when:

1. New evidence is available, or
2. The product owner explicitly requests a change.

When a decision changes:

- Update this log.
- Update the conceptual model.
- Update affected requirements.
- Update category and field definitions.
- Identify implementation and migration impact.

No document should retain a superseded rule as though it were still current.
