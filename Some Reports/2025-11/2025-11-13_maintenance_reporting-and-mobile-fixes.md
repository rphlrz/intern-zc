## End-to-End Report Flow Stabilization, Parameter Order Fix, and Long-URL Mitigation

After recovering connectivity, I met with a colleague to review all changes. I stabilized the print flow and focused on the integration between the corporate MVC app and the report service. During this review I found a semantic mismatch in the mapping DTO: the DTO’s Id field was incorrectly populated with the QrIdentifier (GUID). Integration tests with the legacy .aspx endpoint raised SqlException and JsonSerializationException due to a mismatch in the expected parameter order inside the encrypted parameter string.

The controller built the encrypted string as {Company}#{Partner}#{JSON} while the legacy code-behind expected {Company}#{JSON}#{Partner}. I corrected the assembly order in the controller’s Encrypt method so that the code-behind deserializes parameters correctly.

A separate issue occurred when attempting to print many labels: URLs exceeded the IIS default length limit because identifiers were sent via GET. To avoid a large refactor to POST immediately, I adjusted the selection strategy to send numeric IDs (smaller payload) instead of GUIDs. That required changes in the repository to return numeric sequence IDs and an update in the code-behind to filter by SequenceNumber instead of QrIdentifier. The GUID remained available server-side for other operations, but not in the GET payload to prevent long-URL errors.

I also hardened the client-side validation: the print button now prevents submission when the selection is empty or exceeds 100 items; visual feedback is provided via the toastr-like notification system. Messages were moved to a resource file for localization and are accessed via hidden fields on the page. While performing database inspections with a colleague we observed duplicated organizational unit names in some client data; we recorded this for later data-cleanup work.

Sanitized example SQL used in investigation:
```SQL
SELECT SUM(sub.tot) AS TotalDuplicates
FROM (
  SELECT org.Name, COUNT(*) AS tot
  FROM OrganizationalUnit org
  GROUP BY org.Name
  HAVING COUNT(*) > 1
) sub;
```