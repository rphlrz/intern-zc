### **Maintenance Report – October 13, 2025**

During this maintenance, I implemented registration dates for all equipment records and updated the dashboard queries to ensure data consistency when working with retroactive months.

I started by adding a new property called `RegistrationDate` to the `Equipment` entity. I also updated the creation logic so that whenever a new equipment record is inserted, the current date is automatically assigned to this field. After that, I prepared a migration called `AddRegistrationDateToEquipment` to update the database schema accordingly.

While testing, I encountered a mapping issue related to the AutoMapper configuration. To validate and identify the problem, I used a snippet like this:

```csharp
try
{
    config.AssertConfigurationIsValid();
}
catch (AutoMapperConfigurationException ex)
{
    Debug.WriteLine("AutoMapper error: " + ex.ToString());
    throw;
}
```

This helped me detect inconsistencies between the domain and application layers. I corrected the mappings by explicitly ignoring the `RegistrationDate` field in certain conversions, using the following pattern:

```csharp
.ForMember(dest => dest.RegistrationDate, opt => opt.Ignore());
```

I also set the default registration date in the entity constructor:

```csharp
RegistrationDate = DateTime.Now;
```

After stabilizing the mappings, I worked on the data migration script. The goal was to populate the new column in the `Equipment` table with registration dates derived from the specific equipment tables.
For equipment that already had their own registration or inspection date fields, I used those values.
If neither existed, I assigned the date of the first inspection, and as a last resort, the current system date.

An excerpt from the migration script looked like this:

```sql
UPDATE E
SET E.RegistrationDate = COALESCE(S.RegistrationDate, S.LastInspectionDate, GETDATE())
FROM Equipment E
JOIN SpecificTable S ON E.Id = S.EquipmentId
WHERE E.EquipmentType = 'XXX' AND E.RegistrationDate IS NULL;
```

For equipment types that did not have any associated date columns, I assigned the current date to ensure no null values remained.

During testing, I encountered a `NullReferenceException` related to the `BaseEquipment` class when inserting certain types of equipment. After debugging, I identified and fixed the issue in the domain service layer, ensuring the initialization logic was applied consistently across all equipment types.

Once the migration was completed and validated, the next step was to refactor the system to use this unified field as the **single source of truth** for registration dates. That means all equipment-specific date fields will eventually be removed from their respective tables and replaced by references to `Equipment.RegistrationDate`.

Finally, I aligned the SQL queries that use date filters, ensuring that dashboards now return only equipment with a registration date earlier than or equal to the selected filter range. This guarantees data integrity in historical analyses and corrects the previous inconsistency when querying past months.
