### **Maintenance Report – October 22, 2025**

During this maintenance, I worked on resolving inconsistencies in the dashboard caused by the system not considering equipment registration dates when generating historical reports. This issue affected retroactive month filters, as newly registered equipment appeared in past-period results.

I began by analyzing the filter logic in the *Non-Inspected Equipment Report*, comparing its results with those displayed on the dashboard. Through SQL queries, I confirmed that some pieces of equipment created after the selected filter date were incorrectly included in the results.

Upon reviewing the filters, I identified a dropdown field named *New Equipment*, which determined whether or not to include equipment added after the selected date. However, this logic existed only for hoses and hydrants. I extended this functionality to other equipment types such as extinguishers, cabinets, and generic items.

In the repository, I updated the filtering methods `getQueryNonInspectedVisual` and `getQueryNonInspectedPreventive` to ensure that, when the user selected “No” in the *Include New Equipment* option, only equipment registered before the end of the selected period would be returned. The simplified C# logic was as follows:

```csharp
if (nonInspectedFilter.IncludeNewEquipment == EnumYesNo.No.Code)
{
    var registrationLimitDate = new DateTime(nonInspectedFilter.Year, nonInspectedFilter.Month, 1)
        .AddMonths(1)
        .AddMilliseconds(-1);
    query = query.Where(x => x.RegistrationDate <= registrationLimitDate);
}
```

Later, after aligning with the mobile and backend teams, we decided to standardize the dropdown to contain only “Yes” and “No” options (removing “Select an option”) and to preselect “No” by default.

Next, I created a data migration script to populate the new `RegistrationDate` field in the *Equipment* table. For extinguishers and hoses, the script prioritized the earliest date among the specific registration date, first inspection, or first maintenance request. For other equipment types, it used the earliest available date between the specific registration date or the first inspection, defaulting to the current date if none existed.

The SQL script used multiple `COALESCE` expressions to enforce this hierarchy and ensure that placeholder dates (e.g., `'1900-01-01'`) were properly replaced. Additionally, I ensured the synchronization timestamp (`LastSyncDate`) was updated for all related tables.

After running several tests, the data migration was completed successfully, and the results were validated against control queries to confirm consistency across all equipment categories. The fix ensured that the dashboard and reports now correctly exclude equipment registered after the filtered date, eliminating discrepancies in historical data.
