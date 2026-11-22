# Eilak - Categories, Subcategories and Property Fields

**Document:** 04
**Version:** 2.0
**Last updated:** 2026-11-22
**Status:** Consolidated field reference

---

## 1. Purpose

This document defines the broader product classification and the field definitions that can currently be supported by approved project decisions.

It distinguishes:

- Product classification.
- MVP implementation combinations.
- Fixed category fields.
- Transaction-specific fields.
- Listing-level dynamic fields.
- Unresolved fields and form details.

A field appearing in the broader product classification does not imply that it is implemented in the MVP.

## 2. Product classification

The broader product classification contains five main categories and thirteen subcategories.

| Main category | Subcategory |
|---|---|
| Residential Rent | Apartment Rent |
| Residential Rent | HouseVilla Rent |
| Residential Sale | Apartment Sale |
| Residential Sale | HouseVilla Sale |
| Residential Sale | Land and Old Building Sale |
| Office and Commercial Sale | Office Sale |
| Office and Commercial Sale | Shop and Booth Sale |
| Office and Commercial Sale | Industrial, Agricultural and Commercial Property Sale |
| Office and Commercial Rent | Office Rent |
| Office and Commercial Rent | Shop and Booth Rent |
| Office and Commercial Rent | Industrial, Agricultural and Commercial Property Rent |
| Construction Projects | Construction Partnership |
| Construction Projects | Presale |

Short-term rental is outside the current scope.

## 3. MVP implementation scope

The initial end-to-end MVP supports:

| PropertyType | TransactionType | Form |
|---|---|---|
| Apartment | Sale | ApartmentSale |
| Apartment | Rent | ApartmentRent |
| HouseVilla | Sale | HouseVillaSale |
| HouseVilla | Rent | HouseVillaRent |

Other subcategories remain in the broader product classification.

They must not be represented as completed MVP forms until their field definitions and workflows are approved.

## 4. General data conventions

| Concept | Representation |
|---|---|
| Text | String |
| Integer count | Integer |
| Area | Decimal or Integer according to the field definition |
| Money | Integer/Decimal numeric value in Toman |
| Percentage | Integer from 0 to 100 |
| Year | Numeric year |
| Single choice | Enum or predefined option |
| MultiSelect | Set of predefined options |
| Province | Reference |
| City | Province-dependent reference |
| Media | External file plus database metadata |

General rules:

- Optional values may remain empty.
- Validation is applied when a value is entered.
- Counts are nonnegative unless a field defines a different rule.
- Monetary values are numeric, not formatted strings.
- Phone numbers and identifiers are strings.
- Required/Optional is a per-form decision.
- Enum input must use approved options.
- Free text is not allowed for a selection field unless an approved Other option and explanation rule exist.
- Empty is not equivalent to zero or false.

## 5. Common Listing fields

Common fields belong to Listing rather than category-specific tables.

| Field | Type | Notes |
|---|---|---|
| ListingId | BIGINT | Internal identity |
| PropertyTypeId | Reference | Required |
| TransactionTypeId | Reference | Required |
| ProvinceId | Reference | Required |
| CityId | Reference | Required and dependent on Province |
| Neighborhood | String | Optional free text |
| Address | String | Optional free text |
| YearBuilt | SMALLINT | Optional |
| Description | Text | Customer-facing |
| InternalNote | Text | Private, one current value |
| MarkerColor | TINYINT | Nullable, five fixed values |
| IsActive | Boolean | Active/inactive |
| CreatedAt | UTC timestamp | Immutable |
| UpdatedAt | UTC timestamp | Updated after successful edit |

Coordinates and ListingCode are excluded.

## 6. Apartment fields

All fixed fields below belong to the `Apartment` table.

Dynamic fields are listed separately in Section 9.

| Field | Type | Required/Optional | Rule |
|---|---|---|---|
| UnitArea | DECIMAL(10,2) | Required | Greater than zero |
| BedroomCount | TINYINT | Optional | Nonnegative |
| FloorNumber | SMALLINT | Optional | Negative, zero or positive |
| BuildingFloorCount | SMALLINT | Optional | If entered, at least 1 |
| BuildingUnitCount | SMALLINT | Optional | If entered, greater than 0 |
| UnitsPerFloor | TINYINT | Optional | If entered, greater than 0 |
| UnitDirection | NVARCHAR(30) | Optional | North/South/East/West |
| FloorMaterial | NVARCHAR(50) | Optional | Fixed enum |
| KitchenType | NVARCHAR(30) | Optional | Fixed enum |
| ToiletType | NVARCHAR(20) | Optional | Conditional |
| ToiletCount | TINYINT | Optional | Nonnegative |
| BathroomCount | TINYINT | Optional | Nonnegative |
| HasElevator | BIT | Optional | Nullable Boolean |
| HasParking | BIT | Optional | Nullable Boolean |
| ParkingCount | TINYINT | Optional | Conditional |
| HasStorage | BIT | Optional | Nullable Boolean |
| HasBalcony | BIT | Optional | Nullable Boolean |
| BuildingDirection | NVARCHAR(30) | Optional | Fixed enum |
| BuildingFacade | NVARCHAR(50) | Optional | Fixed enum |
| HasBeenRenovated | BIT | Optional | Major renovation |

### 6.1 Apartment semantics

`UnitArea` is the useful interior area of the unit. Parking, storage and balcony area are excluded.

`BedroomCount` counts rooms usable as bedrooms.

`FloorNumber`:

- Negative: below ground.
- Zero: ground floor.
- Positive: above ground.

If BuildingFloorCount is specified, a positive FloorNumber must not exceed it.

`BuildingFloorCount` counts above-ground building floors including the ground floor.

`BuildingUnitCount` is the total number of units in the building.

`UnitsPerFloor` is the usual number of units per floor.

If both UnitsPerFloor and BuildingUnitCount are entered, UnitsPerFloor must not exceed BuildingUnitCount.

### 6.2 Apartment parking

HasParking means at least one dedicated parking space belongs to the unit.

- HasParking = 0 or NULL: ParkingCount must be NULL.
- HasParking = 1: ParkingCount is optional.
- If ParkingCount is entered, it must be greater than zero.

### 6.3 Toilet and bathroom

ToiletCount counts independent toilet spaces, Iranian or Western.

BathroomCount counts independent spaces with bathing facilities.

A combined toilet/bathroom may be counted in both fields.

Rules:

- ToiletCount NULL or zero => ToiletType NULL.
- ToiletCount greater than zero => ToiletType optional.
- BathroomCount is independent of ToiletCount and ToiletType.

### 6.4 Apartment Boolean semantics

For nullable Boolean fields:

- 1 = Yes.
- 0 = No.
- NULL = Unknown/not entered.

HasElevator means an operational elevator is available to the unit.

HasStorage means a specific usable storage unit is assigned.

HasBalcony means a balcony directly connected to the unit exists. An independent terrace does not qualify.

HasBeenRenovated means major renovation, not minor repair.

### 6.5 Apartment enum options

`ToiletType`:

- Iranian
- Western
- Both

`UnitDirection` and `BuildingDirection`:

- North
- South
- East
- West

`FloorMaterial`:

- Ceramic
- Stone
- Parquet
- Laminate
- Mosaic
- Other

`KitchenType`:

- Open
- Closed
- SemiOpen

`BuildingFacade`:

- Stone
- Brick
- Cement
- Composite
- Glass
- Other

Enum identifiers are English PascalCase. Persian labels are provided by the UI.

## 7. HouseVilla fields

All fixed fields below belong to the `HouseVilla` table.

| Field | Type | Required/Optional | Rule |
|---|---|---|---|
| LandArea | INT | Required | Greater than zero |
| BuildingArea | INT | Required | Greater than zero and <= LandArea |
| BedroomCount | TINYINT | Optional | Nonnegative |
| BuildingFloorCount | SMALLINT | Optional | Nonnegative |
| HasYard | BIT | Optional | Nullable Boolean |
| HasGarden | BIT | Optional | Independent from HasYard |
| HasPool | BIT | Optional | Swimmable pool |
| PoolType | NVARCHAR(20) | Optional | Conditional |
| HasSauna | BIT | Optional | Independent |
| HasJacuzzi | BIT | Optional | Independent |
| HasParking | BIT | Optional | Dedicated parking |
| HasStorage | BIT | Optional | Specific usable storage |
| HasBalcony | BIT | Optional | Independent |
| HasTerrace | BIT | Optional | Independent |
| BuildingDirection | NVARCHAR(30) | Optional | Fixed enum |
| BuildingFacade | NVARCHAR(50) | Optional | Fixed enum |
| FloorMaterial | NVARCHAR(50) | Optional | Fixed enum |
| HasBeenRenovated | BIT | Optional | Major renovation |
| HasFireplace | BIT | Optional | Fixed Boolean |
| HasAirConditioning | BIT | Optional | Fixed Boolean |
| HasSecurityGuard | BIT | Optional | Any permanent/shift guard |
| HasCCTV | BIT | Optional | At least one camera |
| HasRemoteControlledGate | BIT | Optional | Remote-controlled entrance |
| HasIntercom | BIT | Optional | Video intercom only |
| HasSmartHomeSystem | BIT | Optional | Any smart equipment |

### 7.1 Area definitions

LandArea is the land area in square meters.

BuildingArea is the building footprint on the land in square meters. It is not the sum of the areas of all floors.

Both are positive integers.

BuildingArea must not exceed LandArea.

### 7.2 Pool

HasPool refers only to a pool usable for swimming.

PoolType:

- OpenAir
- Covered

Rules:

- PoolType is allowed only when HasPool = 1.
- If HasPool = 1, PoolType is optional.
- If HasPool = 0 or NULL, PoolType must be NULL.

### 7.3 Independent facilities

- HasYard and HasGarden are independent.
- HasSauna and HasJacuzzi are independent of HasPool.
- HasBalcony and HasTerrace are independent.
- HasParking means dedicated parking belonging to the property.
- HasStorage means a specific usable storage space.
- ParkingCount is not part of HouseVilla.

### 7.4 HouseVilla enums

BuildingDirection:

- North
- South
- East
- West

BuildingFacade:

- Stone
- Brick
- Cement
- Composite
- Glass
- Other

FloorMaterial:

- Ceramic
- Stone
- Parquet
- Laminate
- Mosaic
- Other

### 7.5 Security and facilities

HasSecurityGuard includes permanent and shift-based guards.

HasCCTV means at least one camera exists.

HasRemoteControlledGate refers to the entrance gate of the parking area or property.

HasIntercom includes video intercom only.

HasSmartHomeSystem is true when at least one smart-home device exists.

HasFireplace and HasAirConditioning remain fixed fields. They are not replaced by dynamic HeatingSystem and CoolingSystem.

## 8. Transaction-specific fields

### 8.1 ApartmentSale

| Field | Type | Required/Optional | Rule |
|---|---|---|---|
| TotalPrice | DECIMAL(18,0) | Optional | If entered, > 0 |
| DeedStatus | Single choice | Optional | One combined deed/transfer option |
| IsExchangeable | BIT | Optional | Nullable Boolean |
| HasLoan | BIT | Optional | Nullable Boolean |

`LoanAmount` is excluded.

DeedStatus must remain one single-choice field. The exact option list is unresolved and must not be invented.

### 8.2 ApartmentRent

| Field | Type | Required/Optional | Rule |
|---|---|---|---|
| DepositAmount | DECIMAL(18,0) | Optional | If entered, > 0 |
| MonthlyRent | DECIMAL(18,0) | Optional | If entered, > 0 |

### 8.3 HouseVillaSale

| Field | Type | Required/Optional | Rule |
|---|---|---|---|
| TotalPrice | DECIMAL(18,0) | Optional | If entered, > 0 |
| DeedStatus | Single choice | Optional | Same conceptual rule as ApartmentSale |

The exact DeedStatus options remain unresolved.

### 8.4 HouseVillaRent

| Field | Type | Required/Optional | Rule |
|---|---|---|---|
| DepositAmount | DECIMAL(18,0) | Optional | If entered, > 0 |
| MonthlyRent | DECIMAL(18,0) | Optional | If entered, > 0 |
| PetsAllowed | BIT | Optional | 1 yes, 0 no, NULL unknown |
| Furnished | BIT | Optional | 1 yes, 0 no, NULL unknown |

SuitableFor is displayed in this form but stored as a Listing-level dynamic value.

## 9. Listing-level dynamic fields

The following fields are dynamic Listing values, not fixed columns in Apartment or HouseVilla.

| Field | Type | Storage | Form context |
|---|---|---|---|
| CabinetMaterial | MultiSelect | Listing | Relevant apartment/house forms |
| CoolingSystem | MultiSelect | Listing | Relevant property forms |
| HeatingSystem | MultiSelect | Listing | Relevant property forms |
| HotWaterProvider | MultiSelect | Listing | Relevant property forms |
| SuitableFor | MultiSelect | Listing | HouseVillaRent |

### 9.1 CabinetMaterial

Options:

- MDF
- HighGloss
- Membrane
- Metal
- Wood

Persian labels:

- ام‌دی‌اف
- های‌گلاس
- ممبران
- فلزی
- چوب

No Other option.

No explanatory text.

### 9.2 CoolingSystem

Options:

- کولر آبی
- کولر گازی
- اسپلیت
- چیلر
- فن‌کوئل
- داکت‌اسپلیت
- سایر

### 9.3 HeatingSystem

Options:

- پکیج
- موتورخانه
- بخاری گازی
- شوفاژ
- گرمایش از کف
- هیتر
- شومینه
- سایر

### 9.4 HotWaterProvider

Options:

- پکیج
- موتورخانه
- آبگرمکن دیواری
- آبگرمکن ایستاده
- آبگرمکن خورشیدی
- سایر

### 9.5 SuitableFor

Options:

- Family
- Event
- MultipleFamilies

### 9.6 Dynamic field rules

- All listed dynamic fields are optional.
- Empty means unknown/not entered.
- Empty does not mean that the facility does not exist.
- MultiSelect supports multiple simultaneous selections.
- Options are seeded by code.
- Options cannot be managed through the application UI.
- Seed must be idempotent and versioned.
- The physical storage for explanatory text associated with Other remains unresolved for CoolingSystem, HeatingSystem and HotWaterProvider.

## 10. Metadata structure

The approved conceptual metadata/value structure consists of:

1. FieldDefinition
2. FieldOption
3. ListingFieldValue
4. ListingFieldValueOption

The purpose is to define dynamic fields, their fixed options and Listing-level values.

The model must not duplicate fixed Apartment/HouseVilla fields.

Transaction-specific fields such as DeedStatus must not be stored as generic Listing-level dynamic values.

DeedStatus belongs to the transaction-specific record even though its exact option list remains unresolved.

## 11. Broader category fields outside MVP

The following information is retained as broader product requirements, not as confirmed MVP schema.

### 11.1 House and villa classification

A legacy form definition lists these options for property type:

- ویلایی مستقل
- شهرکی
- دوبلکس
- آپارتمانی
- تریپلکس

The final relationship between this legacy form field and the approved HouseVilla model must be reviewed before implementation. Do not add it to the fixed HouseVilla schema solely from this legacy list.

### 11.2 Shop and booth use type

Legacy multi-select options:

- تجاری
- خدماتی
- اداری
- صنعتی
- آموزشی
- درمانی
- انبار
- سایر

This is a multi-select field and must not be converted into a single-choice property type.

It is outside the four-combination MVP.

### 11.3 Industrial, agricultural and commercial property type

Legacy single-choice options:

- سوله
- کارگاه
- انبار
- مغازه
- دفتر کار
- زمین کشاورزی
- زمین صنعتی
- زمین تجاری
- کارخانه
- دفتر اداری
- سردخانه
- سالن صنعتی
- دامداری و گاوداری

This is a single-choice field.

It is outside the four-combination MVP.

### 11.4 Construction partnership current property state

Legacy single-choice options:

- ساختمان قدیمی
- زمین خالی
- در حال ساخت
- ساختمان نوساز

This is outside the current MVP.

### 11.5 Other legacy fields

The source material also mentions:

- Construction project duration in months.
- Partner shares as percentages from 0 to 100.
- Presale delivery time as one month/year value.
- Utility checkboxes.
- Occupancy status.
- Completion certificate.
- Positive features and important notes.
- Other commercial property facilities.

These fields must not be promoted to final physical schema without an approved form-level specification.

## 12. Legacy field definitions not approved for current MVP

The earlier reconstructed field document mentioned a number of fields whose precise assignment was not established.

Examples:

- Utility connections.
- Roof garden.
- Lobby.
- Caretaker.
- Occupancy status.
- Completion certificate.
- Price status.
- Payment terms.
- Advance payment.
- Payment schedule.
- Price per square meter.
- Generic deed and transfer status variants.

These must not be added to the four MVP forms by assumption.

## 13. Required/Optional policy

A universal Required/Optional rule does not exist for every field in every one of the thirteen forms.

Confirmed mandatory fields in the current model include:

- Listing.PropertyTypeId
- Listing.TransactionTypeId
- Listing.ProvinceId
- Listing.CityId
- Apartment.UnitArea
- HouseVilla.LandArea
- HouseVilla.BuildingArea
- Customer.Name
- Customer.PhoneNumber
- Owner mobile number during Listing creation

Other fields remain optional unless a specific approved rule states otherwise.

The complete Required/Optional matrix for all thirteen forms remains open.

## 14. Validation summary

### Apartment

- UnitArea > 0.
- BuildingFloorCount is NULL or >= 1.
- BuildingUnitCount is NULL or > 0.
- UnitsPerFloor is NULL or > 0.
- Positive FloorNumber does not exceed specified BuildingFloorCount.
- UnitsPerFloor does not exceed BuildingUnitCount when both are present.
- ToiletType is NULL when ToiletCount is NULL or zero.
- ParkingCount is NULL when HasParking is false or unknown.
- ParkingCount is positive when specified.
- Enum values belong to their approved option sets.

### HouseVilla

- LandArea > 0.
- BuildingArea > 0.
- BuildingArea <= LandArea.
- Counts are nonnegative when specified.
- PoolType is NULL unless HasPool = 1.
- Enum values belong to their approved option sets.

### Transaction amounts

- TotalPrice, DepositAmount and MonthlyRent are optional.
- When entered, they must be greater than zero.

## 15. Display and input conventions

- Persian labels in UI.
- English PascalCase identifiers for enums.
- Numeric storage for counts and monetary values.
- String storage for phone numbers.
- Optional selection fields can be cleared.
- Required selection fields cannot be cleared.
- MultiSelect supports individual removal and Clear All.
- NULL is not false.
- Empty is not zero.

## 16. Unresolved form-level information

The following information must be obtained from an approved source before the corresponding forms are treated as complete:

1. Exact DeedStatus option list.
2. Required/Optional status of every field in all thirteen forms.
3. Exact field order and grouping for each form.
4. Full fixed field sets for LandOldBuilding, Office, Shop and IndustrialCommercial.
5. Exact legacy form-to-model mapping for HouseVilla property type.
6. Exact storage for Other explanations.
7. Full utility connection definitions.
8. Occupancy and completion certificate definitions.
9. Full construction partnership model.
10. Full presale model.

## 17. Source precedence

When sources disagree, apply this precedence:

1. Explicitly confirmed decisions in the current project decision history.
2. Current approved conceptual model.
3. Current approved requirements.
4. Reconstructed or legacy field documents.
5. Inference from naming or visual similarity.

A reconstructed legacy field list must never override an explicit confirmed decision.
