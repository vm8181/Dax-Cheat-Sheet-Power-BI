# 📊 Power BI DAXpedia Dashboard

[![Power BI](https://img.shields.io/badge/Built%20With-Power%20BI-yellow.svg)](https://powerbi.microsoft.com/)
[![Data Source](https://img.shields.io/badge/Data%20Source-DAX.Guide-33b5e5)](https://dax.guide)

This is a dynamic, visually interactive **DAX Cheat Sheet Dashboard** built in **Power BI** using **Power Query**.  
It pulls live data from the [DAX.Guide](https://dax.guide) portal and allows users to search, filter, and reference DAX functions in a structured way.

---

## 🖼️ Dashboard Preview

### 📘DAX Function Types Overview

![DAX_Cheat_Sheet_Dashboard](https://github.com/user-attachments/assets/4f45654e-c0e6-484d-ac5c-712f767c3279)


### 📘Function Explorer, Search & Metadata

![DAX_Cheat_Sheet_Dashboard_Page2](https://github.com/user-attachments/assets/ce18347e-b51e-434a-a2fe-0c8f7744cfc9)




---

## 🔧 Key Features

- 🔁 **Live data source** from [dax.guide](https://dax.guide)
- 📊 **Category-wise function breakdown** (e.g., Aggregation, Logical, Time Intelligence)
- 🧠 **Dynamic filtering** with **Manage Parameters**
- 📅 **Year-wise release tracking** of DAX functions
- 🔍 **Tooltips** and descriptions for quick reference
- 📥 **PDF Export** included

---

## 🛠️ Technologies Used

| Tool             | Description                                    |
|------------------|------------------------------------------------|
| Power BI Desktop | Visual analytics platform for dashboarding    |
| Power Query      | ETL tool for data import and transformation   |
| DAX.Guide        | Official source for DAX function metadata      |

---
## 🧾 Power Query (M) Code – Full DAX Metadata Loader

This combined M code includes:
- A **main function** that loads multiple DAX function categories
- A nested call to `DaxFunctionsDetails`, which scrapes Syntax, Return Values, Remarks, and Release Date from each DAX function page

```m
//-------------------------------------
// Main Loader Code
//-------------------------------------
**Category Level Data Loader**
let
    Source = Web.BrowserContents("https://dax.guide/"),
    ExtractedTable = Html.Table(Source, {
        {"Function Type", "ul.multi-cols li a"}, 
        {"Description", "ul.multi-cols li p"}
    }, [RowSelector = "ul.multi-cols li"]),
    #"Renamed Columns" = Table.RenameColumns(ExtractedTable,{{"Function Type", "DaxFunctionTypes"}})
in
    #"Renamed Columns"

```m
**Functions Level Data Loader**
let
    FunctionCategoriesText = ParamFunctionsName,
    FunctionCategories = Text.Split(FunctionCategoriesText, ","),

    GetFunctionData = (FunctionName as text) =>
    let
        ParamFunctionName = "functions/" & FunctionName,
        Source = Web.BrowserContents(ParamWebPath & ParamFunctionName),

        ExtractedTable = Html.Table(Source, {
            {"Function Name", "table tbody tr td a"},  
            {"Description", "table tbody tr td + td"}
        }, [RowSelector = "table tbody tr"]),

        ExtractedSections = Html.Table(Source, {
            {"Function Type", "h1"}
        }, [RowSelector = "div.entry-content header"]),

        Result = [Type = ExtractedSections, Name = ExtractedTable]
    in
        Result,

    AllFunctionData = List.Transform(FunctionCategories, each GetFunctionData(_)),
    ExpandRecords = Table.FromRecords(AllFunctionData),
    ExpandedName = Table.ExpandTableColumn(ExpandRecords, "Name", {"Function Name", "Description"}),
    ExpandedType = Table.ExpandTableColumn(ExpandedName, "Type", {"Function Type"}),
    Filtered = Table.SelectRows(ExpandedType, each ([Function Name] <> null)),

    Links = Table.AddColumn(Filtered, "Link", each ParamWebPath & Text.Lower([Function Name])),
    AddExtractedData = Table.AddColumn(Links, "Extracted Data", each DaxFunctionsDetails([Link])),

    ExpandedAll = Table.ExpandTableColumn(AddExtractedData, "Extracted Data", {
        "Syntax", "ReturnValues", "Remarks", "ReleaseDate"
    }),
    ExpandedSyntax = Table.ExpandTableColumn(ExpandedAll, "Syntax", {"Syntax"}),
    ExpandedReturn = Table.ExpandTableColumn(ExpandedSyntax, "ReturnValues", {"Return Values"}),
    ExpandedRemarks = Table.ExpandTableColumn(ExpandedReturn, "Remarks", {"Remarks"}),
    ExpandedDate = Table.ExpandTableColumn(ExpandedRemarks, "ReleaseDate", {"FirstReleaseDate"}),

    InsertedDate = Table.AddColumn(ExpandedDate, "Dates", each Text.End([FirstReleaseDate], 10), type text),

    #"Changed Type" = Table.TransformColumnTypes(InsertedDate, {
        {"Dates", type date},
        {"Function Type", type text},
        {"Function Name", type text},
        {"Description", type text},
        {"Remarks", type text},
        {"Return Values", type text},
        {"Syntax", type text}
    })
in
    #"Changed Type"

```m
//-------------------------------------
// DaxFunctionsDetails Function
//-------------------------------------
let
    Source = (url as text) =>
    let
        Source = Web.Contents(url),

        Syntax = Html.Table(Source, {
            {"Syntax", "section#syntax .notation"}
        }, [RowSelector = "section#syntax"]),

        ReturnValues = Table.SelectColumns(
            Table.AddColumn(
                Html.Table(Source, {
                    {"Values", "section#returns div"},
                    {"Description", "section#returns p:last-of-type"}
                }, [RowSelector = "section#returns"]),
                "Return Values",
                each Text.Combine({[Values], [Description]}, " ")
            ),
            {"Return Values"}
        ),

        Remarks = Html.Table(Source, {
            {"Remarks", "section#remarks"}
        }, [RowSelector = "section#remarks"]),
        CleanedRemarks = Table.TransformColumns(Remarks, {{"Remarks", Text.Clean, type text}}),
        ReplaceRemarks = Table.ReplaceValue(CleanedRemarks, "Remarks", "", Replacer.ReplaceText, {"Remarks"}),
        TrimmedRemarks = Table.TransformColumns(ReplaceRemarks, {{"Remarks", Text.Trim, type text}}),

        FirstReleaseDate = Html.Table(Source, {
            {"FirstReleaseDate", "section.first-release p"}
        }),

        Combined = Table.FromRecords({[
            Syntax = Syntax,
            ReturnValues = ReturnValues,
            Remarks = TrimmedRemarks,
            ReleaseDate = FirstReleaseDate
        ]})
    in
        Combined
in
    Source

