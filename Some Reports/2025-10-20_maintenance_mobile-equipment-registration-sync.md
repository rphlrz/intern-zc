### **Maintenance Report – October 20, 2025**

On this day, I focused on implementing and adjusting the new *registration date* property across the mobile environment to ensure proper synchronization between corporate and mobile systems. The main objective was to include this information in the synchronization flow so that both systems could maintain consistent registration data for all equipment types.

After configuring the mobile environment (Blazor/Maui), I worked alongside a colleague to debug and fix a synchronization issue in one of the mobile databases. The problem was related to inconsistent and incomplete data, likely caused by a manual database intervention. Once corrected, synchronization resumed as expected.

Returning to the main task, I added the property

```csharp
public DateTime RegistrationDate { get; set; }
```

to all DTOs responsible for synchronization between corporate and mobile systems — such as *Shelter*, *ButtonPanel*, *PumpHouse*, *RespiratorEquipment*, *Extinguisher*, *Hydrant*, *EmergencyKit*, *Light*, and *Hose*. Later, I removed the property from those not directly used in mobile synchronization (inside the `ReceiveSyncServices` class), keeping only the relevant entities.

In the configuration mappings, I handled two inheritance cases. For entities derived from a *BaseEquipment* class, I mapped the field using the nested equipment object:

```csharp
.ForMember(dest => dest.RegistrationDate, opt => opt.MapFrom(src => src.Equipment.RegistrationDate));
```

For entities that inherited directly from *Equipment*, the mapping was simpler:

```csharp
.ForMember(dest => dest.RegistrationDate, opt => opt.MapFrom(src => src.RegistrationDate));
```

To ensure that the registration date was correctly initialized when creating new equipment, I updated the equipment constructor to include a nullable parameter for the registration date. If no date was provided, the current UTC time would be assigned automatically:

```csharp
public Equipment(Guid humanResourceId, Guid locationId, Guid companyId, string code, string typeCode,
                 Guid inspectionModelId, Guid partnerId, string? integrationId = null, DateTime? registrationDate = null)
{
    HumanResourceId = humanResourceId;
    LocationId = locationId;
    EquipmentCode = code.Trim();
    TypeCode = typeCode;
    InspectionModelId = inspectionModelId;
    PartnerId = partnerId;
    CompanyId = companyId;
    RegistrationDate = registrationDate ?? DateTime.UtcNow;
    IntegrationId = integrationId;
}
```

In the synchronization adapter classes, I extended the adaptation logic to handle the new property. For example, when mapping data received from the API, I ensured that each equipment instance correctly received its registration date:

```csharp
public List<Equipment> Adapt(ICollection<HoseResponse> data)
{
    var equipmentList = new List<Equipment>();
    var executionDate = DateTime.UtcNow;

    foreach (var item in data)
    {
        var equipment = new Equipment(GetHumanResourceId(item.HumanResourceId.ToString()),
                                      GetLocationId(item.LocationId.ToString()),
                                      SelectedCompanyId,
                                      item.Number,
                                      EquipmentTypeValueObject.Hose.Code,
                                      GetModelId(item.InspectionModelId),
                                      SelectedPartnerId,
                                      item.EquipmentId,
                                      item.RegistrationDate);
        [...]
    }
}
```

Finally, I updated the response model classes used by the mobile API layer to include the new property:

```csharp
public class HoseResponse
{
    [...]
    public DateTime RegistrationDate { get; set; }
}
```

With these changes, both corporate and mobile environments can now exchange and preserve the registration date of each equipment unit consistently during synchronization.
