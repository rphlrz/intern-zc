# intern-zc

Study repository with C#, .NET, Blazor, and SQL Server snippets and mini‑POCs from an internship, focused on reusable patterns, clean code, and practical problem‑solving, with all content anonymized and free of sensitive information.

### Overview
This repository collects focused code snippets, small utilities, and proof‑of‑concepts across backend and frontend using the .NET ecosystem. Each example is intentionally scoped, readable, and easy to copy into real projects without bringing along unrelated dependencies or company‑specific details.

### Tech stack
- C# and .NET (APIs, services, testing, CLI utilities)
- Blazor (components, forms, validation, service integration)
- SQL Server (queries, scripts, indexing, routines, modeling)
- Optional: Entity Framework, ASP.NET Core, xUnit or MSTest for tests

### Repository structure
- csharp/ — language features, patterns, utilities
- dotnet/ — services, middlewares, validation, testing samples
- blazor/ — components, pages, state, API calls
- sql/ — DDL/DML scripts, indexes, views, stored procedures, tuning examples
- docs/ — quick notes, conventions, references for the repo

### Getting started
1. Install a compatible .NET SDK (the sample folders indicate required versions; some examples may target .NET Framework 4.8 and others .NET 6+).  
2. Clone the repository and open the desired folder in Visual Studio or VS Code.  
3. Read the local README in each sample for specific prerequisites and run instructions.  

Typical commands:
```bash
# Build and run .NET samples
dotnet restore
dotnet build
dotnet run --project path/to/sample

# Run tests (if present)
dotnet test
```
For .NET Framework samples, open the .sln in Visual Studio and use the standard build/run workflow.

### How to use the SQL scripts
- Scripts under sql/ are safe to run on a local dev instance; they avoid destructive operations unless explicitly documented.  
- Always review a script before executing and prefer running inside a transaction when experimenting locally.  

### Conventions
- Prefer small, self‑contained examples with clear naming and minimal dependencies.  
- Keep configuration examples public but never include real secrets, certificates, or connection strings.  
- Use comments sparingly to clarify intent when the code alone is not obvious.  

### Commits and versioning
- Commit style: Conventional Commits with gitmoji where meaningful.  
- Example:  
  ```
  ✨ feat(repo,docs,build): bootstrap intern-zc with README (EN) and .gitignore
  ```
- Versioning: semantic version tags starting at v0.1.0 for navigable milestones.

### Security and privacy
- No internal project names, real datasets, or sensitive information.  
- appsettings.json may exist for illustration; local or secret variants should be ignored by Git.  

### Roadmap (optional)
- More Blazor component patterns and testing examples  
- EF performance snippets (tracking vs no‑tracking, projections, pagination)  
- SQL Server indexing checklists and parameter sniffing demos  

### License
MIT — see LICENSE for details.

### Tags
dotnet, csharp, blazor, sql-server, snippets, internship, patterns, clean-code, best-practices