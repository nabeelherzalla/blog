<!--{Title:"LINQ To SQL Changes in Visual Studio 2010", PublishedOn:"2009-10-12T05:43:16", Intro:"Damien Guard was nice enough to blog about changes coming to L2S in VS 2010. Rather, the changes are"} -->

[Damien Guard was nice enough to blog about changes coming to L2S in VS 2010](http://damieng.com/blog/2009/06/01/linq-to-sql-changes-in-net-40). Rather, the changes are coming in the .NET Framework 4.0.
The whole rumour within the development community/blogs about "LINQ To SQL is being unsupported, Entity Framework is the new coolness" was just plain wrong, I believe. Microsoft is too big, and has too many projects on the go. They'll gauge the momentum of both technologies. Have you heard [Damien as a guest on Herding Code Episode 50](http://herdingcode.com/?p=187)? In this blog post, Damien says that the focus for Microsoft will be on EF, and that's fine. L2S is definitely not dead!
I still believe that if you're needing an ORM, and working with SQL Server, then use LINQ To SQL. I've tried EF, and it worked fine. I've ended up with 4 projects using L2S, and haven't found any real need for EF.

###Welcome Defect Fixes in coming in .NET 4.0 for Linq-To-Sql###

For me, the most interesting changes within Damien's post are:

* `Contains()` with enums automatically casts to int or string depending on column type**

* `String.StartsWith()`, `EndsWith()` and `Contains()` now correctly handles `~` in the search string (regular &amp; compiled queries) - Here's a small defect. I've not needed to search for tildes very much, but I decided to give it a shot just in case! It's true, the behavior is just as described! 

![tilde2.png](img/tilde2.png)

 
![tilde_thumb.png](img/tilde_thumb.png)
 

**Now detects multiple active result sets (MARS) better** - I am not the heaviest user of L2S, and definitely haven't needed to specify MARS myself. Here's the [MultipleActiveResultSets defect as reported on MS Connect](https://connect.microsoft.com/VisualStudio/feedback/ViewFeedback.aspx?FeedbackID=366444). It's a simple issue where the connection string property `MultipleActiveResultSets` is only picked up when CamelCased exactly as shown above. Any deviation will ignore the option! 

**DeleteDatabase no longer fails with case-sensitive database servers** - Interesting that this functionality even exists. I had to research this method - `DataContext.DeleteDatabase()`. I can't recall actually seeing it in the IntelliSense method list, but indeed it's there! Most blogpost or articles that I read in that 5 minute span were talking about using this method for tear-down during "Unit Testing". I'd call that integration testing, and ill-informed as well. Unit tests should not include databases! 

![DeleteDatabase.png](img/DeleteDatabase.png)

**VarChar(1) now correctly maps to string and not char** - This one has bitten me before. The column was called Gender. Of course it was storing M, F, T, or U for unknown. The core of the problem was that some rows were having a blank stored in this field, rather than null. StackOverflow to the rescue! [http://stackoverflow.com/questions/1190328/linq-to-sql-exception-string-must-be-exactly-one-character-long](http://stackoverflow.com/questions/1190328/linq-to-sql-exception-string-must-be-exactly-one-character-long). After some thought, I'd agree that storing this value as `char(1)` would be semantically more correct, more performant, and consume just one byte per tuple. 

![varchar.png](img/varchar.png)
![lenzero.png](img/lenzero.png)
**Decimal precision and scale are now emitted correctly in the DbType attributes for stored procedures &amp; computed columns** - I couldn't reproduce this defect, and perhaps I misunderstood.  I defined a `decimal(18,5)` attribute on the table, and L2S brought it back without any problems. Then I realized the key to this defect was probably the 'computed' bit. So I went and created a simple decimal return type. I ran the query, and still no defect. 

![computed.png](img/computed.png)
![decimalsOK.png](img/decimalsOK.png)

Then I clued in - the defect was under the LINQ To SQL Designer heading. So upon further inspection, here's the defect in the `myL2S.designer.cs`. The return type is calculated as `decimal(0,0)`. Ouch! :)

![decimalzero_thumb.png](img/decimalzero_thumb.png)

**Foreign key changes will be picked up when bringing tables back into the designer without a restart** - This defect has hit me a few times as well. It appears as such: 

* Edit a FK in SQL Server. If you've got an open L2S file, deleting + dragging and dropping those tables back onto the L2S surface DO NOT show your FK changes. 
* Clicking the Refresh button in Server Explorer doesn't help. 
* The only solution is to close your L2S file, and re-open. 

**Changing a FK for a table and re-dragging it to the designer surface will show new FK's** - this is very much related to the item above. 
**Opening a DBML file no longer causes it to be checked out of source control** - this has appeared to me a few times. Simply opening/viewing the L2S file creates a 'check out'. Nothing earth shattering here, and glad to see this is fixed. 
**Can edit the return value type of unidentified stored procedure types** - This feature is great! It's very helpful when you've got a sproc that shapes data just the exact perfect way that you'd like to show on a custom report. Perhaps you're binding to an `asp:GridView` or including as part of an MVC FormViewModel. The normal course of action is: 

* Create your proc to shape your data as you like[![Sproc-To-Custom-Object](img/SprocToCustomObject_thumb.png)](http://devtxt.com/blog/blogimg/LinqToSqlChangesinVisualStudio2010_BE1A/SprocToCustomObject.png)
* Drag a proc onto the Methods section of the L2S designer. 
* Try change its return type to a class you've created solely for the purpose of binding
* Oops, it's locked!
[![locked](img/locked_thumb.png)](http://devtxt.com/blog/blogimg/LinqToSqlChangesinVisualStudio2010_BE1A/locked.png)

The work-around is a bit time-consuming. You have to:
    
* open up the `myL2S.designer.cs` file       
* find your method marked with the attribute containing your proc name
* replace the default return type `int` with `ISingleResult<t>` - in my simple example here, it's `ISingleResult<customerreport>` 
        
[![modifydbml_thumb.png](img/modifydbml_thumb.png)](http://devtxt.com/blog/blogimg/LinqToSqlChangesinVisualStudio2010_BE1A/modifydbml.png)
The frustrating bit here is that this behavior isn't predictable (to me at least). I CONSTANTLY have to go through this process to properly set the return type of 2 procs in one particular project. This defect fix in particular by Microsoft will be welcome!