---
title: "Entity Framework Table-Valued Functions (TVFs) (EF5 onwards) | Microsoft Docs"
ms.custom: ""
ms.date: "2016-10-23"
ms.prod: "visual-studio-2013"
ms.reviewer: ""
ms.suite: ""
ms.technology: 
  - "visual-studio-sdk"
ms.tgt_pltfrm: ""
ms.topic: "article"
ms.assetid: f019c97b-87b0-4e93-98f4-2c539f77b2dc
caps.latest.revision: 3
---
# Entity Framework Table-Valued Functions (TVFs) (EF5 onwards)
> **EF5 Onwards Only** - The features, APIs, etc. discussed in this page were introduced in Entity Framework 5. If you are using an earlier version, some or all of the information does not apply.

The video and step-by-step walkthrough shows how to map table-valued functions (TVFs) using the Entity Framework Designer. It also demonstrates how to call a TVF from a LINQ query.

TVFs are currently only supported in the Database First workflow.

TVF support was introduced in Entity Framework version 5. Note that to use the new features like table-valued functions, enums, and spatial types you must target .NET Framework 4.5. Visual Studio 2012 targets .NET 4.5 by default.

TVFs are very similar to stored procedures with one key difference: the result of a TVF is composable. That means the results from a TVF can be used in a LINQ query while the results of a stored procedure cannot.

[See the video that accompanies this step-by-step walkthrough.](../ef6/entity-framework-table-valued-functions-tvfs-ef5-onwards-video.md)
 

## Pre-Requisites

To complete this walkthrough, you must install the [School database](../ef6/entity-framework-school-database.md).

You will need to have Visual Studio 2012, Ultimate, Premium, Professional, or Web Express edition installed to complete this walkthrough.

 

## Set up the Project

1.  Open Visual Studio 2012
2.  On the **File** menu, point to **New**, and then click **Project**
3.  In the left pane, click **Visual C\#**, and then select the **Console** template
4.  Enter **TVF** as the name of the project and click **OK**

## Add a TVF to the Database

-   Select **View -&gt; SQL Server Object Explorer**
-   If **(localdb)\\v11.0** is not in the list of servers:
    Right-click on **SQL Server** and select **Add SQL Server**
    Use the default **Windows Authentication** to connect to the **(localdb)\\v11.0** server
-   Expand **(localdb)\\v11.0**
-   Under the Databases node, right-click the School database node and select **New Query…**
-   In T-SQL Editor, paste the following TVF definition

```
CREATE FUNCTION [dbo].[GetStudentGradesForCourse] 
 
(@CourseID INT) 
 
RETURNS TABLE 
 
RETURN 
    SELECT [EnrollmentID], 
           [CourseID], 
           [StudentID], 
           [Grade] 
    FROM   [dbo].[StudentGrade] 
    WHERE  CourseID = @CourseID
```

-   Click the right mouse button on the T-SQL editor and select **Execute**
-   The GetStudentGradesForCourse function is added to the School database

 

## Create a Model

1.  Right-click the project name in Solution Explorer, point to **Add**, and then click **New Item**
2.  Select **Data** from the left menu and then select **ADO.NET Entity Data Model** in the **Templates** pane
3.  Enter **TVFModel.edmx** for the file name, and then click **Add**
4.  In the Choose Model Contents dialog box, select **Generate from database**, and then click **Next**
5.  Click **New Connection**
    Enter **(localdb)\\v11.0** in the Server name text box
    Enter **School** for the database name
    Click **OK**
6.  In the Choose Your Database Objects dialog box, under the **Tables** node, select the **Person**, **StudentGrade**, and **Course** tables
7.  Select the **GetStudentGradesForCourse** function located under the **Stored Procedures and Functions** node
    Note, that starting with Visual Studio 2012, the Entity Designer allows you to batch import your Stored Procedures and Functions
8.  Click **Finish**
9.  The Entity Designer, which provides a design surface for editing your model, is displayed. All the objects that you selected in the **Choose Your Database Objects** dialog box are added to the model.
10. By default, the result shape of each imported stored procedure or function will automatically become a new complex type in your entity model. But we want to map the results of the GetStudentGradesForCourse function to the StudentGrade entity:
    Right-click the design surface and select **Model Browser**
    In Model Browser, select **Function Imports**, and then double-click the **GetStudentGradesForCourse** function
    In the Edit Function Import dialog box, select **Entities** and choose **StudentGrade**

## Persist and Retrieve Data

Open the file where the Main method is defined. Add the following code into the Main function.

The following code demonstrates how to build a query that uses a Table-valued Function. The query projects the results into an anonymous type that contains the related Course title and related students with a grade greater or equal to 3.5.

```
using (var context = new SchoolEntities()) 
{ 
    var CourseID = 4022; 
    var Grade = 3.5M; 
     
    // Return all the best students in the Microeconomics class. 
    var students = from s in context.GetStudentGradesForCourse(CourseID) 
                            where s.Grade >= Grade 
                            select new 
                            { 
                                s.Person, 
                                s.Course.Title 
                            }; 
 
    foreach (var result in students) 
    { 
        Console.WriteLine( 
            "Couse: {0}, Student: {1} {2}", 
            result.Title,  
            result.Person.FirstName,  
            result.Person.LastName); 
    } 
}
```

Compile and run the application. The program produces the following output:

```
Couse: Microeconomics, Student: Arturo Anand
Couse: Microeconomics, Student: Carson Bryant
```

## Summary

In this walkthrough we looked at how to map Table-valued Functions (TVFs) using the Entity Framework Designer. It also demonstrated how to call a TVF from a LINQ query.
