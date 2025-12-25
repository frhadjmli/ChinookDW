# Chinook Data Warehouse (SSIS)
A data warehouse sample for the Chinook database, demonstrating full-load dimensional and fact modeling, ETL processes, and analytical querying for reporting and business intelligence use cases.

---

## Overview

This repository contains the SSIS project and supporting artifacts used to build the Chinook Data Warehouse (DW). The DW is a snowflake schema designed for analytics and reporting on music sales, customers, playlists and related metadata. The warehouse consolidates OLTP tables (`Invoice`, `InvoiceLine`, `Track`, `Album`, `Artist`, `Genre`, `MediaType`, `Customer`, `Employee`, `Playlist`, etc.) into one central fact table and a set of dimensions.

**Primary design goals:**

* Simple snowflake example
* Surrogate-keyed dimensions for slowly changing attributes (basic SCD support)
* A single grain `FactInvoice` that represents invoice line items (one row per sold track-line)

---

## Prerequisites

* Microsoft SQL Server (required for DW database)
* SQL Server Integration Services (SSIS) / Visual Studio with SSIS extension (for the SSIS project)
* Sufficient disk space and permissions to run packages that read the source DB and write to the DW
* (Optional) Power BI or other BI tool if you plan to connect and visualize

---

## How to deploy

1. Restore or make available the source Chinook OLTP database (if needed for reloading).
2. Open the SSIS project in Visual Studio (SSDT) or SSIS designer.
3. Configure package-level and project-level connection managers:

   * Source connection (Chinook OLTP)
   * Destination connection (Chinook_DW)
4. Update any configuration values (e.g., schema names, truncate vs incremental flags, file paths) in the project configuration or environment-specific configurations.
5. Execute the packages in the recommended order (if packages are split):

   * Load dimensions (order matters for referential keys)
   * Load fact table
6. Verify results using the validation queries supplied in `sql/` or by running the checks below.

**Note:** The DW in this repository is *already full-loaded* (the data files/tables were loaded during development). If you need to refresh, run the SSIS packages after adjusting connections.

---

## Data model (snowflake schema)

### Fact table

* **FactInvoice** (single fact storing invoice header + invoice line details)

  * Purpose: stores sales transactions at the invoice-line grain (a sold track or line item).
  * Typical columns (examples): `InvoiceSK` (surrogate key), `InvoiceDateKey`, `CustomerSK`, `GeographySK`, `TrackGenreMediaSK`, `AlbumArtistSK`, `Quantity`, `UnitPrice`, `LineTotal`
* Notes: Fact was built by joining `Invoice` and `InvoiceLine` from the source.

### Dimension tables

* **DimEmployee** — employee metadata (salesperson, support reps)

  * Key: `EmployeeSK` (surrogate), business key e.g. `EmployeeId`
  * Common attributes: fullname, title, hire date, SalaryGroup, etc.

* **DimCustomer** — customer metadata

  * Key: `CustomerSK`
  * Attributes: fullname, company, supportRepEmployeeSK, etc.

* **DimGeography** — geography / location (city / region / country)

  * Key: `GeographySK`
  * Attributes: city, state, country.

* **DimDate** — date dimension used for invoice dates, load dates, etc.

  * Key: `DateKey` (YYYYMMDD integer or surrogate), `Date` (date type)
  * Attributes: year, quarter, month, day, weekday, fiscal flags, etc.

* **DimTrackGenreMedia** — denormalized dimension joining Track, Genre and Media Type

  * Key: `TrackGenreMediaSK`
  * Attributes: `TrackId` (business), `TrackName`, `GenreName`, `MediaTypeName`, `Composer`, `DurationGroup`, `SizeGroup`, `AlbumArtistSK`, etc.

* **DimAlbumArtist** — denormalized dimension joining Album and Artist

  * Key: `AlbumArtistSK`
  * Attributes: `AlbumArtistSK`, `AlbumTitle`, `ArtistName`.

* **DimPlaylist** — playlist metadata

  * Key: `PlaylistSK`
  * Attributes: `PlaylistSK`, `PlaylistName`.

* **DimPlaylistTrack** — junction dimension / bridge table for Playlist ↔ Track (many-to-many)

  * Key: `PlaylistTrackSK` (surrogate)
  * Attributes: `PlaylistSK`, `TrackSK`

---

## Keys & Relationships

* Dimensions use surrogate keys (`*SK`) assigned during ETL.
* `FactInvoice` references dimension surrogate keys.
* `DimDate[DateKey]` joins `FactInvoice[InvoiceDateKey]`.
* All foreign keys in the fact should have referential integrity to corresponding dimensions after a successful load.

---

## Performance tips

* Ensure indexes on foreign key columns in the fact (e.g., `AlbumArtistSK`, `TrackGenreMediaSK`, `InvoiceDateKey`, `CustomerSK`).
* Keep `LineTotal` as a computed and persisted numeric column in the fact if queries frequently aggregate on it.
* Consider partitioning `FactInvoice` by date key if the table grows large.

---

## Contact

- Project maintainer: `Farhad Jamali`
    
- Email: farhad080.jamali@gmail.com
  
---

## License

This project is licensed under the **MIT License**. See `LICENSE` for details.

---
