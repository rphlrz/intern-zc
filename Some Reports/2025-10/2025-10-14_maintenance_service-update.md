### **Maintenance Report – October 14, 2025**

During this maintenance, my main objective was to implement and stabilize the new property `RegistrationDate` within the generic entity `Equipment`, ensuring that it was correctly persisted during both creation and update operations.

The work began with debugging a `NullReferenceException` that occurred during insertion in services such as `StationService`. The issue was caused by trying to assign `entity.Equipment.RegistrationDate = DateTime.Now;` before the nested `Equipment` object was instantiated. To fix this, I adjusted the execution order so that the `UpdateEquipment(entity)` method was invoked first — ensuring that the `Equipment` object existed — and only afterward assigning the registration date.

Once the insertion logic was corrected, I focused on the update flow. At that stage, a `SqlException` was thrown when editing any record, indicating that converting a `datetime2` value to `datetime` produced an out-of-range error. Upon inspection, I confirmed that the update process was recreating a new `Equipment` entity instead of loading the existing one. This caused the `RegistrationDate` property (now non-nullable in C#) to assume its default value (`01/01/0001`), which is invalid for SQL Server’s `datetime` type.

To solve this properly, I refactored the update strategy to use a **“load-first”** pattern. The updated method now retrieves the complete original entity from the database using the repository method `GetByIdWithAllIncludes`. After loading, the system applies only the fields allowed to change, ensuring that immutable fields — such as `RegistrationDate` — remain consistent.

However, this change initially caused a side effect: the many-to-many relationship between `Station` and `Hydrant` stopped persisting. The issue occurred because the logic responsible for handling this collection was lost in the refactor. To resolve this cleanly, I added a new method `GetByIds(IEnumerable<int> ids)` to the `IHydrantRepository` interface and its implementation. Then, within the service layer, I reintroduced the relationship management logic by retrieving the selected hydrants and reassigning them to the station before saving. This preserved data integrity and respected separation of concerns between the service and repository layers.

Later, after reviewing the approach with a colleague, I simplified the update logic to a more streamlined version. The new implementation restored part of the original structure while maintaining the necessary improvements. Specifically, it uses an array of `Expression<Func<Station, object>>` to define properties that should be ignored during updates — such as `RegistrationDate`, `LastVisualInspectionResultCode`, and `ActiveCode`. The service also retrieves the corresponding `Equipment` entity via `repository.GetEquipmentByStationId(entity.Id)` before updating, ensuring proper linkage and consistency.

The final structure of the update flow looks like this (simplified version with illustrative names):

```csharp
public new void Update(Station entity)
{
    var ignoredProperties = new Expression<Func<Station, object>>[3];
    ignoredProperties[0] = item => item.RegistrationDate;
    ignoredProperties[1] = item => item.LastInspectionResultCode;
    ignoredProperties[2] = item => item.StatusCode;

    var selectedHoseIds = entity.Hoses?.Select(x => x.Id).ToArray();
    entity.Hoses = null;

    var location = factoryMethod.GetInstance<ILocationService>().GetByIdWithAllIncludes(entity.LocationId);
    entity.Location = location;
    entity.CompanyId = location.Unit.CompanyId;

    entity.LastUpdatedBy = userSessionService.GetAuthenticatedUserId();
    entity.LastUpdatedAt = DateTime.Now;

    entity.Equipment = repository.GetEquipmentByStationId(entity.Id);
    entity.EquipmentId = entity.Equipment.Id;

    entity.UpdateEquipment(entity);
    entity.ValidateUpdate();

    base.Update(entity, EnumTypeUpdate.IgnoreProperty, ignoredProperties);

    var hoses = hoseService.GetByStationId(entity.Id);
    hoseService.UnlinkStation(hoses.Select(x => x.Id).ToArray());

    if (selectedHoseIds != null)
        hoseService.LinkStation(entity.Id, selectedHoseIds);
}
```

In summary, this maintenance involved:

* Adding the new `RegistrationDate` field to the `Equipment` entity and database.
* Ensuring correct initialization during creation and preservation during updates.
* Fixing SQL date conversion errors caused by invalid default values.
* Refactoring the update logic to adopt a load-first approach and maintain data consistency.
* Reestablishing relationship persistence between entities (`Station`, `Hydrant`, and `Hose`).
* Creating repository methods for efficient entity retrieval.

After thorough testing — including creating, editing, and reassigning associated entities — the system now correctly handles equipment registration dates across the entire data flow.
