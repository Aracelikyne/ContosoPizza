# W01 Assignment Notes

## 1. Existing pizzas and my additional pizza

Results from the end of the *Create a web API with ASP.NET Core controllers* module.

The two starting pizzas (`GET /pizza`):

```json
[
  { "id": 1, "name": "Classic Italian", "isGlutenFree": false },
  { "id": 2, "name": "Veggie", "isGlutenFree": true }
]
```

After adding my pizza with POST (`GET /pizza`):

```json
[
  { "id": 1, "name": "Classic Italian", "isGlutenFree": false },
  { "id": 2, "name": "Veggie", "isGlutenFree": true },
  { "id": 3, "name": "Hawaii", "isGlutenFree": false }
]
```

## 2. Requests, responses, and status codes

All requests were sent to the running API (`dotnet run`) using `ContosoPizza.http` / curl.

### GET (all pizzas)

**Request**
```http
GET /pizza
Accept: application/json
```

**Response:** `200 OK`
```json
[{"id":1,"name":"Classic Italian","isGlutenFree":false},{"id":2,"name":"Veggie","isGlutenFree":true}]
```

### GET (one pizza by id)

**Request**
```http
GET /pizza/3
Accept: application/json
```

**Response:** `200 OK`
```json
{"id":3,"name":"Hawaii","isGlutenFree":false}
```

A pizza that doesn't exist (`GET /pizza/5`) returns `404 Not Found`.

### POST

**Request**
```http
POST /pizza
Content-Type: application/json

{
    "name": "Hawaii",
    "isGlutenFree": false
}
```

**Response:** `201 Created` (header `Location: /Pizza/3`)
```json
{"id":3,"name":"Hawaii","isGlutenFree":false}
```

### PUT

**Request**
```http
PUT /pizza/3
Content-Type: application/json

{
    "id": 3,
    "name": "Hawaiian",
    "isGlutenFree": false
}
```

**Response:** `204 No Content` (empty body)

Checking with `GET /pizza/3` returned `200 OK` with `{"id":3,"name":"Hawaiian","isGlutenFree":false}`.

### DELETE

**Request**
```http
DELETE /pizza/3
```

**Response:** `204 No Content` (empty body)

Checking with `GET /pizza` returned `200 OK` with only the two original pizzas.

## 3. Sales summary function

From the *Work with files and directories in a .NET app* module. `GenerateSalesSummary` writes a sales summary report to `salesTotalDir/salesSummary.txt`: the overall total, followed by each file's total.

```csharp
void GenerateSalesSummary(IEnumerable<string> salesFiles, string storesDirectory, string reportPath)
{
    var details = new List<string>();
    double grandTotal = 0;

    foreach (var file in salesFiles)
    {
        // Reuse CalculateSalesTotal to get the total for just this one file
        double fileTotal = CalculateSalesTotal(new[] { file });
        grandTotal += fileTotal;

        // Show the path relative to the stores folder, e.g. "201/sales.json"
        var fileName = Path.GetRelativePath(storesDirectory, file);
        details.Add($"  {fileName}: {fileTotal:C}");
    }

    var report = new List<string>
    {
        "Sales Summary",
        "----------------------------",
        $" Total Sales: {grandTotal:C}",
        "",
        " Details:"
    };
    report.AddRange(details);

    // Overwrite the report each run so it always reflects the current files
    File.WriteAllLines(reportPath, report);
}
```

It reuses the module's `CalculateSalesTotal` function to get each file's total:

```csharp
double CalculateSalesTotal(IEnumerable<string> salesFiles)
{
    double salesTotal = 0;
    
    // Loop over each file path in salesFiles
    foreach (var file in salesFiles)
    {      
        // Read the contents of the file
        string salesJson = File.ReadAllText(file);
    
        // Parse the contents as JSON
        SalesData? data = JsonConvert.DeserializeObject<SalesData?>(salesJson);
    
        // Add the amount found in the Total field to the salesTotal variable
        salesTotal += data?.Total ?? 0;
    }
    
    return salesTotal;
}

record SalesData (double Total);
```

It's called from the top of `Program.cs`:

```csharp
GenerateSalesSummary(salesFiles, storesDirectory, Path.Combine(salesTotalDir, "salesSummary.txt"));
```

Output after running `dotnet run` (`salesTotalDir/salesSummary.txt`):

```
Sales Summary
----------------------------
 Total Sales: $2,012.20

 Details:
  sales.json: $88.88
  204/sales.json: $88.88
  204/salestotals.json: $0.00
  203/sales.json: $99.00
  203/salestotals.json: $0.00
  202/sales.json: $1,234.22
  202/salestotals.json: $0.00
  201/sales.json: $501.22
  201/salestotals.json: $0.00
```
