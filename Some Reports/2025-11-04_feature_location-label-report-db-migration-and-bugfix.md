### Feature Report – November 04, 2025

I started by fixing a bug in the "Select All" feature. It was selecting all records in the database, not just the filtered results. The executeAjax call in the JavaScript was missing the filter parameters. I fixed it by passing the current filter values (like locationCode and isActive) to the controller action, which now correctly returns only the IDs from the filtered set.

```JavaScript

// Fixing the 'Select All' AJAX call
$('#btn-select-all').on('click', function (event) {
    event.preventDefault();
    executeAjax($(this).attr('data-jquery-url'), undefined, 'POST', 'returnSelectAll', undefined, 'JSON',
    {
        // Parameters to match the controller action
        companyId: $("#CompanyId").val(),
        unitId: $("#unitId").val(),
        locationCode: $("#locationCode").val(),
        equipmentType: $("#equipmentType").val(),
        isActive: $('#isActive').val()
    }, true);
});
```

Next, I implemented the database change for the QR code feature. I added a new QrIdentifier property (public Guid QrIdentifier { get; set; }) to the Location entity. In the Entity Framework LocationConfiguration, I mapped this property, but new records were being saved with all-zero GUIDs. I fixed this by adding .HasDatabaseGeneratedOption(DatabaseGeneratedOption.Identity) to let the database handle GUID generation.

I then created the database migration. To handle existing data, the migration runs in three steps: 1) AddColumn as nullable, 2) Sql("UPDATE dbo.Location SET QrIdentifier = NEWID() WHERE QrIdentifier IS NULL") to backfill all existing locations, and 3) AlterColumn to make it non-nullable and set defaultValueSql: "NEWID()" for future inserts. Finally, I updated the AutoMapper profile to map this new QrIdentifier to the Id property of the LocationLabelReportDTO.

```C#

// The final migration 'Up' method
public override void Up()
{
    // 1. Add column as nullable
    AddColumn("dbo.Location", "QrIdentifier", c => c.Guid(nullable: true));

    // 2. Backfill existing data
    Sql("UPDATE dbo.Location SET QrIdentifier = NEWID() WHERE QrIdentifier IS NULL");

    // 3. Alter to be non-nullable with a default value
    AlterColumn("dbo.Location", "QrIdentifier", c => c.Guid(nullable: false, defaultValueSql: "NEWID()"));
}
```