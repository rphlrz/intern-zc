### Bugfix Report – October 29, 2025

After feeling unwell this morning, I spent time helping a colleague with unrelated mobile application issues. We identified and fixed a critical bug that caused a Foreign Key constraint violation during data synchronization. The root cause was that the GetEquipmentForDeletion method was only fetching the parent Equipment entity, without its child dependencies (like FireExtinguisher, Hydrant, etc.). When the sync process tried to delete the parent, the database blocked it. The solution was to modify the Entity Framework query to use eager loading by adding multiple .Include() statements for the dependent entities. This ensured the EF Change Tracker was aware of the full object graph and could correctly cascade the delete.

```C#

// Anonymized example of the Eager Loading fix
public IQueryable<Equipment> GetEquipmentForDeletion(int... ids)
{
    return _repository.GetQuery()
        .Include(x => x.ChildTypeOne)
        .Include(x => x.ChildTypeTwo)
        .Include(x => x.ChildTypeThree)
        // ...and so on for all dependencies
        .Where(x => ids.Contains(x.Id));
}
```
I also fixed two other UI bugs. First, a save icon wasn't appearing because the logic was flawed. I refactored it to set _canSave = true by default and then explicitly set it to false only if the user lacked the correct permission (ActionType.Update for edits or ActionType.Create for new entries). Second, I corrected a DatePickerComponent display issue in a Razor file by setting its PickerVariant to Dialog and hiding the toolbar, as recommended by the component's documentation.