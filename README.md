# Building-a-Medallion-Architecture-in-Microsoft-Fabric

![Medallion Architecture Diagram](https://github.com/user-attachments/assets/8d843251-022c-4d1d-9f2e-65c01c737185)

## Project Summary
This project describes how to implement a medallion lakehouse architecture in Microsoft Fabric. The Medallion architecture consists of three layers: `Bronze Layer` (the Raw Data Layer), `Silver Layer` (validated), and `Gold Layer` (enriched). Each layer has a specific purpose and is designed to support efficient data governance and performance. 

* Created Lakehouse
* Data Ingestion. 
* Silver Data Transformation.
* Gold Data Transformation.
* Data Modelling.
* Data Visualization in Power BI Service.

This project was presented by [Musili Adebyo](https://www.linkedin.com/in/musili-adebayo/) at the [Microsoft Fabric Nigeria](https://community.fabric.microsoft.com/t5/Microsoft-Fabric-Nigeria/gh-p/MicrosoftFabricNigeria) and [dbrownconsulting](https://www.linkedin.com/posts/dbrownconsulting_analyticsmeetup-dbrownconsulting-microsoftfabric-activity-7255201470994673668-hX6G?utm_source=share&utm_medium=member_desktop) physical meet-up on 26th October 2024

## Data Sources:
Datasource: Data was sourced from the popular adventure works data you can [download Here](https://github.com/AbdurRahman-Olaniyan/Building-a-Medallion-Architecture-in-Microsoft-Fabric/tree/main/Sample_Adventureworks_Dataset)

## Creating Lakehouse:
In Synapse Data Engineering, create three Lakehouses called Sales_Bronze, Sales_Silver and Sales_Gold.

![Creating Bronze, Silver and Gold Lakehouse](https://github.com/user-attachments/assets/8485c128-1599-46da-b7f6-ed52dc33c798)

## Data Ingestion:
* In the Sales_Bronze Lakehouse, Upload the: [Sample Adventure Works dataset](https://github.com/AbdurRahman-Olaniyan/Building-a-Medallion-Architecture-in-Microsoft-Fabric/tree/main/Sample_Adventureworks_Dataset)

![Bronze_Layer](https://github.com/user-attachments/assets/77703939-f7d2-4304-ad9b-b5dee199a40b)

## Silver Data Transformation.
In the Sales_Silver Lakehouse, click on new Notebook to transform data for the Sales_Silver Lakehouse or you can import the [Silver_Data_Transformation Notebook](https://github.com/AbdurRahman-Olaniyan/Building-a-Medallion-Architecture-in-Microsoft-Fabric/blob/main/Silver_Data_Transformation.ipynb). Remember to replace the ABFS File Path of each table from the Sales_Bronze lakehouse before you Run ALL.

![Sales_Silver_Notebook](https://github.com/user-attachments/assets/9cc32938-6738-4ba7-9b39-10d5ef45bc94)

### Step-by-Step Transformation
1. **Customers Table Transformation**
   - Defined a schema (`customers_schema`) to ensure consistent data types for key columns like `CustomerID`, `FirstName`, `LastName`, etc.
   - Loaded customer data from the Bronze layer using `spark.read`, specifying `header` and schema options.
   - Added new columns:
     - `DateInserted`: current timestamp to mark the record insertion time.
     - `SourceFilename`: name of the source file to aid data lineage tracking.
   - Saved the transformed data to the Silver layer using the Delta format with schema merge enabled for compatibility.

2. **Employees Table Transformation**
   - Loaded employee data from Bronze without a schema definition and then casted columns individually.
   - Added metadata columns (`DateInserted`, `SourceFilename`) for data validation.
   - Saved the dataframe to the Silver layer in Delta format, with an option to overwrite if the schema changes.

3. **Orders Table Transformation**
   - Loaded orders data from Bronze, cast columns for numerical data, and converted date columns (`OrderDate`, `DueDate`, `ShipDate`) using `to_date`.
   - Added validation columns for tracking (`DateInserted`, `SourceFilename`).
   - Saved the data in Delta format, merging schema for future updates.

4. **Products Table Transformation**
   - Loaded product data from Bronze, cast columns such as `ProductID` and `MakeFlag`.
   - Added validation fields to capture insertion details.
   - Saved the transformed dataframe to Silver Lakehouse in Delta format.

5. **Product Categories Table Transformation**
   - Loaded data, casted columns like `CategoryID` and `CategoryName` as specified types.
   - Included metadata for record tracking.
   - Saved to Silver Lakehouse in Delta format.

6. **Product Subcategories Table Transformation**
   - Loaded subcategory data, cast `SubCategoryID` and `CategoryID` as integers.
   - Captured source details and insertion timestamps.
   - Saved the result to Silver layer in Delta format.

### Data Validation and Lineage Tracking
The inclusion of `DateInserted` and `SourceFilename` columns ensures that each record’s origin and insertion time are traceable, aiding in data validation and simplifying cleanup operations.

## Gold Data Transformation.
In the Sales_Gold Lakehouse, click on new Notebook to transform data for the Sales_Gold Lakehouse or you can import the [Gold_Data_Transformation Notebook](https://github.com/AbdurRahman-Olaniyan/Building-a-Medallion-Architecture-in-Microsoft-Fabric/blob/main/Gold_Data_Transformation.ipynb). Remember to replace the ABFS File Path of each table from the Sales_Bronze lakehouse before you Run ALL.

![Gold_Layer_Notebook](https://github.com/user-attachments/assets/becf04f1-5da3-435c-9f31-91380e8e760b)

### Step-by-Step Transformation
This setup ensures that your Gold layer contains refined, cleansed, and business-ready data for analytics. It creates and saves dimension and fact tables in the Gold layer for a lakehouse architecture.

1. **Customer Dimension (DimCustomer)**:
   - Loads relevant customer data from the Silver layer (`Sales_Silver.Customers`), renames `CustomerID` to `CustomerKey`, and writes it to the Gold layer Delta Lake path `Tables/DimCustomer`.

2. **Employee Dimension (DimEmployee)**:
   - Creates a temporary view (`tmpEmployee`) from the `Sales_Silver.Employees` table with selected columns. The data is saved in Delta format at `Tables/DimEmployee`.

3. **Product Dimension (DimProduct)**:
   - Defines a temporary view (`tmpProducts`) by selecting specific fields from `Sales_Silver.Products`, and saves the result to `Tables/DimProduct`.

4. **Product Category Dimension (DimProductCategory)**:
   - Selects fields from `Sales_Silver.ProductCategories` into a temporary view (`tmpProductCategory`) and writes it to `Tables/DimProductCategory`.

5. **Product SubCategory Dimension (DimProductSubCategory)**:
   - Creates a view for product subcategories (`tmpProductSubCategory`) using `Sales_Silver.ProductSubCategories` and writes it to `Tables/DimProductSubCategory`.

6. **Sales Fact Table (FactSales)**:
   - Constructs the fact table `tmpSales` from `Sales_Silver.Orders`, joining with dimension tables in the Gold layer. It uses left joins for customer, employee, product, and product category details, with `-1` as the default key for any missing references.
   - Writes the fact table to `Tables/FactSales`.

## Data Modelling.
Now, we have the final data in the Sales_Gold Lakehouse, Next is to model the semantic data before visualization in Power BI.
![image](https://github.com/user-attachments/assets/069cd62c-d9da-4e5b-b0fa-730004f9b8ac)

## Data Visualization in Power BI.
In Microsoft Fabric, you can create Power BI reports for your analysis.
![Adventure-Works-Sales-Dashboard](https://github.com/user-attachments/assets/c5d11ec9-271e-4b48-a9d5-3d6caba5098f)

### create a date  table in powerBI service
you may create a date table within dataflow gen2 using m language or Use SQL SCript to create it
Then publish the table to either your lakeHouse or warehouse
- Open the semantic Data Model to create relationship with fact table
![image](https://github.com/user-attachments/assets/92de6862-ec57-4164-a12b-d7b8c1b83ec3)

- you can use the method to create a seperate table to encapsulate your Dax Measure, read more about [Edit data models in the Power BI service (preview)](https://learn.microsoft.com/en-us/power-bi/transform-model/service-edit-data-models)


