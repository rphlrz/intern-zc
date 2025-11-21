### Bugfix Report – November 19, 2025

While validating the fixes from the previous day, I noticed a regression in the Summary Report models (Types 1 and 2). When the filter criteria resulted in zero records, the application generated a corrupted Excel file. This occurred because the Excel generation library attempted to build a workbook based on data groups; with an empty dataset, no groups were formed, resulting in a workbook with zero worksheets, which is invalid.

To address this, I implemented a guard clause in the export service methods (GenerateDetailedReport, GenerateSummaryReportModel1, and GenerateSummaryReportModel2). The system now checks if the result list is empty before attempting to process the data. If no records are found, it explicitly creates a single sheet named "Empty Report" containing a user-friendly message.

```C#

private byte[] GenerateSummaryReportModel2(EquipmentFilter filter)
{
    var list = equipmentService.Filter(filter);

    var exporter = new ExcelDataExporter(Language.ReportTitle, Language.ReportTitle);
    exporter.CreateDocument();

    // ++++++ START OF INTERVENTION ++++++
    // Prevents file corruption by ensuring at least one sheet exists
    if (list == null || list.Count == 0)
    {
        var sheet = exporter.Workbook.CreateSheet("Empty Report");
        var row = sheet.CreateRow(0);
        row.CreateCell(0).SetCellValue(Language.Message_NoRecordsFound);
        sheet.AutoSizeColumn(0);

        return exporter.GetMemoryStream().ToArray();
    }
    // ++++++ END OF INTERVENTION ++++++

    var groups = list
        .GroupBy(x => new { /* Grouping logic */ })
        .OrderBy(g => g.Key.FantasyName);

    foreach (var g in groups)
    {
       // ... Sheet generation logic ...
    }

    return exporter.GetMemoryStream().ToArray();
}
```

I applied this pattern to all export variations to ensure consistency. The solution was tested, documented, and deployed to the production environment.