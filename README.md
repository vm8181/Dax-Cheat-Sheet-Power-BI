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
## 🧾 Power Query (M) Code – DAX Function Scraper

The following M function extracts **Syntax**, **Return Values**, **Remarks**, and **Release Date** from a given DAX function URL using HTML scraping from [DAX.Guide](https://dax.guide):

```m
let
    Source = (url as text) =>
    let
        Source = Web.Contents(url),

        // Extract Syntax
        Syntax = Html.Table(Source, {
            {"Syntax", "section#syntax .notation"}
        }, [RowSelector = "section#syntax"]),

        // Extract Return Values
        Return_Values = Html.Table(Source, {
            {"Return Values", "section#returns div"},
            {"Description", "section#returns p:last-of-type"}
        }, [RowSelector = "section#returns"]),
        ReturnValues = Table.SelectColumns(Table.AddColumn(Html.Table(Source, { // Extracts "Return values"
            {"Values", "section#returns div"},
            {"Description", "section#returns p:last-of-type"}// Extracts the description under Return values
        }, [RowSelector = "section#returns"]), "Return Values", each Text.Combine({[Values], [Description]}, " ")), {"Return Values"}),


        // Extract Remarks
        Remarks = Html.Table(Source, {
            {"Remarks", "section#remarks"}
        }, [RowSelector = "section#remarks"]),
        CleanedRemarks = Table.TransformColumns(Remarks, {{"Remarks", Text.Clean, type text}}),
        ReplaceRemarks = Table.ReplaceValue(CleanedRemarks,"Remarks","",Replacer.ReplaceText,{"Remarks"}),
        TrimmedRemarks = Table.TransformColumns(ReplaceRemarks, {{"Remarks", Text.Trim, type text}}),
        // Release Date
        FirstReleaseDate = Html.Table(Source, {
            {"FirstReleaseDate", "section.first-release p"}}),



        // Combine results into a single table
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
