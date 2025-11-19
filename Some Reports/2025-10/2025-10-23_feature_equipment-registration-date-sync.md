### **Feature Report – October 23, 2025**

During this maintenance, I worked on validating and refining the registration date synchronization between the corporate system and the mobile application. The objective was to ensure that all equipment correctly receives its registration date, whether created on the corporate system or directly on the mobile app, and that this information remains consistent across both platforms after synchronization.

I started by adjusting the SQL migration script responsible for defining registration dates for existing records. The script now evaluates the earliest available date between the first inspection and the first maintenance request for each equipment type, ensuring that a realistic and consistent value is stored. The simplified structure looks like this:

```sql
CASE 
	WHEN eqp.Type = 'EXT' THEN 
		CASE 
			WHEN insp.FirstInspectionDate < mnt.FirstMaintenanceDate THEN insp.FirstInspectionDate
			ELSE mnt.FirstMaintenanceDate
		END
	WHEN eqp.Type = 'MAN' THEN 
		CASE 
			WHEN insp.FirstInspectionDate < req.FirstRequestDate THEN insp.FirstInspectionDate
			ELSE req.FirstRequestDate
		END
	ELSE insp.FirstInspectionDate
END,
@CurrentDate
```

After running the script in test environments, I performed several validations to confirm that registration dates were being generated correctly. I tested scenarios such as registering equipment on the corporate system, synchronizing to mobile, and verifying whether the registration date persisted correctly when editing records on both sides.

During testing, I discovered that some equipment types were not updating the synchronization timestamp. After debugging, I found that in certain service methods responsible for synchronization, the `Equipment` object was arriving as `null` because the method responsible for fetching and updating the entity was not being called properly.

To fix this, I modified the `UpdateSynchronization` method to ensure that each record retrieves the corresponding equipment entity from the repository before applying updates. The relevant excerpt is shown below:

```csharp
public void UpdateSynchronization(ICollection<SynchronizationDTO<GenericEquipment>> data)
{
    [...]

    item.Data.Equipment = repository.GetEquipment(item.Data.Id);
    item.Data.UpdateEquipment(item.Data);

    [...]

    dbContext.EnableAutoSaveChanges();
}
```

After applying this change, most synchronization cases worked as expected, maintaining consistent registration dates and timestamps across systems. Some equipment types still required further investigation, which I planned to continue in the following day’s maintenance session.