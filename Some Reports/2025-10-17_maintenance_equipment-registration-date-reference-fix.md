### **Maintenance Report – October 17, 2025**

During this maintenance task, I worked on adjusting the implementation of the new *registration date* field across the repositories and services of several equipment modules. The goal was to ensure that the dashboard queries correctly filtered equipment data based on their registration date, maintaining consistency when working with retroactive monthly reports.

I started by modifying the repository filters for multiple modules — such as *Shelter*, *Extinguisher*, *Hydrant*, and *Hose* — so that the queries would exclude equipment registered after the reporting period. For example, in the query that lists uninspected items, I added a condition to limit the results to equipment with a registration date earlier than the filter’s end date:

```csharp
if (filter.IncludeNewEquipment == EnumYesNo.No.Code)
{
    var registrationLimitDate = new DateTime(filter.Year, filter.Month, 1).AddMonths(1);
    query = query.Where(x => x.Equipment.RegistrationDate <= registrationLimitDate);
}
```

In the **configuration layer**, I made mapping adjustments to ensure that DTOs and ViewModels properly received and formatted the registration date.
For example, in DTO mappings where there was no direct navigation to the `Equipment` entity, I used the following structure:

```csharp
.ForMember(dest => dest.ExtRegDate, opt => opt.MapFrom(src => src.Extinguisher.Equipment.RegistrationDate));
```

For ViewModels, I ensured proper date formatting and handled cases where the date might not be initialized:

```csharp
(src => src.Equipment.RegistrationDate != default(DateTime)
    ? src.Equipment.RegistrationDate.ToString("dd/MM/yyyy HH:mm")
    : null)
```

During testing, I found an issue affecting 26 equipment types that directly inherited from the `Equipment` base class — these were returning *date out of range* errors. To solve this, I added logic to retrieve the existing registration date directly from the database during the update process. The service layer was updated to include the property in the update expression and explicitly assign the value from the repository:

```csharp
entity.RegistrationDate = repository.GetRegistrationDate(entity.Id);
```

This required extending the interface and repository with a new method:

```csharp
public DateTime GetRegistrationDate(string id)
{
    return base.FilterAsNoTracking()
               .Where(x => x.Id == id)
               .Select(x => x.RegistrationDate)
               .SingleOrDefault();
}
```

After implementing these fixes, further testing revealed that some modules — such as *Shelter*, *Hydrant*, and *Hose* — were not displaying the registration date correctly on the edit screens. This was due to missing entity includes in their query configurations. I resolved the issue by explicitly including the `Equipment` navigation property within the repository queries:

```csharp
.Include(x => x.Equipment)
```

This adjustment ensured that all dependent data was properly loaded, allowing the registration date to be correctly displayed and updated through the interface.