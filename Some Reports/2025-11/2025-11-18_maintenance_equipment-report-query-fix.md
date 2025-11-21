### Maintenance Report – November 18, 2025

I started working on ticket 7854 regarding an inconsistency in the Safety Equipment Report. The client reported that when filtering by the "PIER EQUIPMENT" organizational unit, the on-screen grid correctly displayed 274 records, but the "Detailed Version" export resulted in a file with no data.

After downloading the updated database and testing the scenario locally, I successfully reproduced the issue. Upon inspecting the data access layer, I identified that the Filter method in the repository was manually constructing the query predicates but failed to account for the sub-unit logic. While the screen listing used a centralized method to handle organizational hierarchy, the export query was missing this specific check.

To resolve this, I refactored the Filter method to utilize the shared ApplyOrganizationalUnitFilter method. This ensures that the logic for including or excluding sub-units is applied consistently across both the grid view and the Excel export.

```C#

ICollection<SafetyEquipment> ISafetyEquipmentRepository.Filter(EquipmentFilter filter, string userId)
{
    var query = base.Filter()
        .Include(x => x.Equipment)
        // ... other includes ...
        .Include(x => x.Standard);

    if (filter.CompanyId.HasValue)
        query = query.Where(x => x.CompanyId == filter.CompanyId);

    // ++++++ START OF INTERVENTION ++++++
    // Replaced manual check with the centralized filter method to include sub-units correctly
    query = ApplyOrganizationalUnitFilter(query, filter);
    // ++++++ END OF INTERVENTION ++++++

    if (filter.EmployeeId.HasValue)
        query = query.Where(x => x.EmployeeId == filter.EmployeeId);

    // ... remaining filters ...

    return query.ToList();
}
```
This adjustment aligned the dataset of the export with the one shown on the user interface.