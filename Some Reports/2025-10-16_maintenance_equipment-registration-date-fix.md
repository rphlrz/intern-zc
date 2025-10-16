### **Maintenance Report – October 16, 2025**

During this maintenance task, I investigated why the migration script responsible for updating the equipment registration dates had no effect after execution. After analyzing the issue, I discovered that the query filter `WHERE registration_date IS NULL` was no longer valid, since the column had been changed to **NOT NULL** in a previous migration. As a result, older records contained the default value `1900-01-01 00:00:00.000`, which is automatically inserted by SQL Server when a `NOT NULL` datetime field has no assigned value.

Once I identified the issue, I adjusted the filter to target records with the placeholder date `1900-01-01`, ensuring that the update would correctly affect legacy records. To validate the update process and visualize the results, I designed a transactional query that performed the update while logging all changes in a temporary in-memory table. This allowed me to compare **old and new registration dates**, along with the **data source** used to populate each record.

The update logic followed a hierarchical approach using the `COALESCE()` function to determine the most reliable registration date for each piece of equipment. The function prioritized, in order:

1. The registration date from the specific equipment-type table (e.g., housing, button panel, extinguisher, hydrant, etc.),
2. The date of the first recorded inspection, and
3. The current system date (`GETDATE()`), if neither of the previous values was available.

For example, the simplified version of the logic looked like this:

```sql
UPDATE e
SET e.registration_date = COALESCE(
        CASE e.type_code
            WHEN 'A' THEN a.created_at
            WHEN 'B' THEN b.created_at
            WHEN 'C' THEN c.created_at
        END,
        i.first_inspection_date,
        GETDATE()
    ),
    e.last_sync_date = GETDATE()
FROM Equipment e
LEFT JOIN EquipmentTypeA a ON e.id = a.equipment_id
LEFT JOIN EquipmentTypeB b ON e.id = b.equipment_id
LEFT JOIN EquipmentTypeC c ON e.id = c.equipment_id
LEFT JOIN (
    SELECT equipment_id, MIN(inspection_date) AS first_inspection_date
    FROM Inspection
    GROUP BY equipment_id
) i ON e.id = i.equipment_id
WHERE e.registration_date = '1900-01-01';
```

To ensure data consistency, I also updated the synchronization date fields (`last_sync_date`) across all related tables. After confirming the results through several validation queries — including counts by equipment type and comparisons between new and original data sources — I prepared the final version of the migration script for execution in the production-like environment.

This change ensures that all equipment records now have coherent and accurate registration dates, improving the reliability of dashboard reports that depend on temporal filtering and preventing inconsistencies in retroactive analyses.
