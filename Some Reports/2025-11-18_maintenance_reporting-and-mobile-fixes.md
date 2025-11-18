## Dropdown Styling Refinements and Fix for Missing Sub-Unit Filtering in Export

I continued styling the dropdown, documented progress, and set the next tasks: finalize dropdown styling, review all related code, study widget options further, and synchronize with colleagues.

During debugging I observed that the parent-name stopped appearing; I changed the DTO property name from ParentUnit back to Parent to match how the view consumed the value. Sanitized DTO snippet:

```C#
public class OrgUnitDropdownDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Parent { get; set; } // previously ParentUnit, renamed to match view usage
}
```

Later that same day I paused the dropdown work to address a separate ticket (ID 7854). The client reported that when filtering for a specific organizational unit the summary listing correctly showed 274 items, but exporting the detailed report produced a blank file. The client provided a video demonstrating the problem and I reproduced the issue locally.

Investigation revealed the export query was not considering sub-units. I modified the repository filter to include subunits when the "include subunits" flag is set. The intervention adds retrieval of all descendant unit IDs and filters the query accordingly. Sanitized repository snippet (C# pseudocode):

```C#
if (filter.OrgUnitId.HasValue)
{
    if (filter.IncludeSubUnits)
    {
        var subUnits = orgUnitRepository.GetAllSubUnitIds(filter.OrgUnitId.Value);
        query = query.Where(x => subUnits.Contains(x.Location.OrgUnitId) ||
                                 x.Location.OrgUnitId == filter.OrgUnitId);
    }
    else
    {
        query = query.Where(x => x.Location.OrgUnitId == filter.OrgUnitId);
    }
}
```

I tested the fix against the updated client database and confirmed the export now contains the expected records. I documented the change and noted that two condensed export templates failed in later tests and require further verification.