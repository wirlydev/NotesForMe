---
title: Pivoting Data with Pivot Operator
date: "2019-12-26T22:12:03.284Z"
description: "Pivoting Data with Pivot Operator"
---

##What am I tryig to accomplish?

I would like to return a table that lists all the D1 Wrestling National Champions for a specified year and the number of losses they have had in their collegiate career. 

The column names should be the wrestler's name and the data row should have the number of losses.

The result should look something like...

|Wrestler 1 Name |Wrestler 2 Name |Wrestler 3 Name | etc. |
|----------------|----------------|----------------|------|
|3			     |5               |0               |5     |

The wrestler who won the National Championship with the most career losses will win the ficticious "Foster Resilience Award". 

##What does the original data look like?

To keep things simple for this example all the data is in a single table that looks like this...

|AthleteName     |Wins  |Losses|CompetitionYear|NationalPlacement|
|----------------|------|------|---------------|-----------------|
|Anthony Ashnault|32    |0     |2019           |1                |
|Bo Nickal       |30    |0     |2019           |1                |
|Drew Foster     |28    |5	   |2019	       |1                |


If you are interested in trying this yourself with the same data here is the sql to create and populate the source table. https://gist.github.com/wirlydev/919178ac61ea57b5315fa29e524eef57

##How do I do this?

Since I want to "pivot" the source table from vertical to horizontal I can use the [TSQL Pivot Operator](https://docs.microsoft.com/en-us/sql/t-sql/queries/from-using-pivot-and-unpivot?view=sql-server-ver15)

Here is an example of a sproc that will do the job.

```sql
Alter PROCEDURE GetCareerLosses 
	@CompetitionYear int,
	@NationalPlacement int
AS
BEGIN
	SET NOCOUNT ON;
	
	declare @sql nvarchar(max) = '';
	declare @names nvarchar(max) = '';

	Select @names += QUOTENAME(AthleteName) + ',' from Athletes where CompetitionYear = @CompetitionYear and NationalPlacement = @NationalPlacement order by AthleteName
	Set @names = Left(@names, len(@names) -1)

	set @sql = 'Select * from 
				(Select 
					a1.AthleteName,
					Sum(a1.Losses) Losses
				from Athletes a1 inner join 
					Athletes a2 on (a1.AthleteName = a2.AthleteName)
					where a2.CompetitionYear = @competitionYear and a2.NationalPlacement = @nationalPlacement
					group by a1.AthleteName) as SourceTable
				Pivot (
					Sum(Losses) 
					For AthleteName in (' + @names + ')
					)
				as PivotedTable';

				print @sql

				exec sp_executesql @sql,  N'@competitionYear int, @nationalPlacement int, @names nvarchar(max)', @competitionYear, @nationalPlacement, @names 

END
GO
```

##What is happening here?

If I run the above sproc for the competition year 2019 like...

```sql
exec GetCareerLosses @CompetitionYear= 2019, @NationalPlacement = 1
```

This is the value of the variable @sql

```sql
Select * from 
				(Select 
					a1.AthleteName,
					Sum(a1.Losses) Losses
				from Athletes a1 inner join 
					Athletes a2 on (a1.AthleteName = a2.AthleteName)
					where a2.CompetitionYear = @competitionYear and a2.NationalPlacement = @nationalPlacement
					group by a1.AthleteName) as SourceTable
				Pivot (
					Sum(Losses) 
					For AthleteName in ([Anthony Ashnault],[Anthony Cassar],[Bo Nickal],[Drew Foster],[Jason Nolf],[Mekhi Lewis],[Nick Suriano],[Spencer Lee],[Yianni Diakomihalis],[Zahid Valencia])
					)
				as PivotedTable
```

If I run it for 2018, @sql is this...

```sql
Select * from 
				(Select 
					a1.AthleteName,
					Sum(a1.Losses) Losses
				from Athletes a1 inner join 
					Athletes a2 on (a1.AthleteName = a2.AthleteName)
					where a2.CompetitionYear = @competitionYear and a2.NationalPlacement = @nationalPlacement
					group by a1.AthleteName) as SourceTable
				Pivot (
					Sum(Losses) 
					For AthleteName in ([Bo Nickal],[Jason Nolf],[Kyle Snyder ],[Michael Macchiavello],[Seth Gross],[Spencer Lee],[Vincenzo Joseph],[Yianni Diakomihalis],[Zahid Valencia],[Zain Retherford])
					)
				as PivotedTable

```

The big difference and the only reason we have to do this as dynamic sql is the list of names in the "For AthleteName in (..." part. Without dynamic sql we would need to manually update the query with the list of names for the competition year we are querying.

The subquery that populates the SourceTable provides the data the we are going to pivot so for example if we run it for 2019 we get 

|AthleteName	 |Losses|
|----------------|------|
|Anthony Ashnault|22    |
|Anthony Cassar	 |11    |
|Bo Nickal	     |5     |
|Drew Foster	 |44    |
|Jason Nolf		 |4     |
|Mekhi Lewis	 |4     |
|Nick Suriano	 |7     |
|Spencer Lee	 |5     |
|Yianni Diakomihalis|1  |
|Zahid Valencia	 |5     |















