### **Maintenance Report – October 21, 2025**

On this date, I continued working on the implementation of the *equipment registration date* feature, focusing on completing the synchronization between systems and improving dashboard queries to handle retroactive data more accurately.

I began by creating a migration named `AddRegistrationDate`, responsible for adding the new registration date field to the equipment table. To validate the migration, I performed several tests — first sending the database to a colleague before inserting new equipment records, then repeating the process after the insertions. The synchronization behaved as expected.

In the `AdapterToReceiveService` class, I updated the mapping logic for all equipment entities (e.g., `Equipment`, `Shelter`, `ControlPanel`, `RespiratorDevice`, `Extinguisher`, `Hydrant`, `EmergencyKit`, `DiphoterineKit`, and `Hose`). Each entity now includes the new property in its parameters:

```csharp
RegistrationDate = equipment.RegistrationDate.ToLocalTime()
```

After finalizing synchronization adjustments, I started improving the dashboard logic to ensure it only considers equipment registered before the selected month’s end. To achieve this, I added a filter based on the registration date to several repository methods responsible for generating dashboard data. The added code was:

```csharp
DateTime registrationLimitDate = filter.MonthYear.AddMonths(1).AddMilliseconds(-1);
query = query.Where(x => x.RegistrationDate < registrationLimitDate);
```

This modification ensures that reports and preventive inspection dashboards display only equipment existing before the filter’s cutoff date, maintaining consistency when analyzing previous months.
The update was applied to multiple repositories related to preventive and visual inspections — including `EquipmentRepository`, `PreventiveInspectionRepository`, and others handling hose, hydrant, extinguisher, and shelter visual inspections.

Next, I planned to review and analyze the “Not Inspected” report to confirm that the same filtering logic would apply consistently across all relevant reporting modules.
