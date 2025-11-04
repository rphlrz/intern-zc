### Feature Report – October 31, 2025

I returned to the Location Label Report task and focused on refactoring the backend. I started by cloning the EquipmentLabelReportController to create the new LocationLabelReportController. The most significant change was simplifying the search logic. The old controller had a large if/else if block that would call a different service (IEquipmentSubtypeService, IAnotherEquipmentService, etc.) based on the selected equipment type. I removed this entire structure and replaced it with a single, direct call to the ILocationAppService.

This simplification extended to the filter object and DTO. I created a new LocationReportFilter class, removing equipment-specific fields and adding LocationCode. I also created a LocationLabelReportDTO with properties relevant to a location (like Code, Description, EquipmentTypeName, and IsLocationActive).

Finally, I refactored the AutoMapper configuration. The old setup had dozens of mappings (one for each equipment type to the old DTO). I replaced this with a single, clean mapping from the Location entity to the new LocationLabelReportDTO.

```C#

// Old approach (simplified)
cfg.CreateMap<EquipmentTypeOne, EquipmentLabelReportDTO>();
cfg.CreateMap<EquipmentTypeTwo, EquipmentLabelReportDTO>();
cfg.CreateMap<EquipmentTypeThree, EquipmentLabelReportDTO>();

// New approach
cfg.CreateMap<Location, LocationLabelReportDTO>()
    .ForMember(dest => dest.Description, opt => opt.MapFrom(src => src.LocationDescription))
    .ForMember(dest => dest.EquipmentTypeName, opt => opt.MapFrom(src => src.EquipmentType.DisplayName));
    // ... other mappings
```