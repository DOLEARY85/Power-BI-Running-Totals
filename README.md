# Creating Running Totals In Power BI
Running totals are a very common query in Power BI and they are fairly simple to complete, here are a few options to solve this:

**Calculated Column**

You can use calculated column to create a running total based on another field e.g. To get a week on week running total for sales based on the week number column: You would create a column that adds together the sales and filters the data to all sales where the week number is less than or equal to the current week number this can be achieved by using the EARLIER function.

        Running Total= 
        CALCULATE (
            SUM ('Table'[Sales]),
            ALL ('Table'),
           'Table'[Week] <= EARLIER ('Table'[Week]))

![image](https://github.com/DOLEARY85/Power-BI-Running-Totals/assets/126701906/d79231ff-e20a-42b0-bfab-4c46a27ad02b)

**Measure**

The second option is to create a measure. This could work based on a date field by using a similar DAX expression to the one used in the calculated column. The only adjustments needed to this is you would to enclose the ALL Function within a FILTER function and use the MAX function to replace the EARLIER function used in the calculated column:

        Running Total (m)= 
        CALCULATE (
            SUM  ('Table'[Sales]),
            FILTER (
            ALL ('Table'),
            'Table'[Date] <= MAX ('Table'[Date])))
            
 ![image](https://github.com/DOLEARY85/Power-BI-Running-Totals/assets/126701906/a628b0c9-3c76-4d4c-8eef-b5fc66be58e7)

**Power Query List.Sum/List.FirstN**

Your third option is to use Power Query which will avoid runtime calculations so can be quicker. To use the List.Sum with List.FirstN functions you will need to start with your data sorted in the correct order, you'll then need to add an Index column starting from the value 1 instead of the default 0. You will then be able to create a custom column with the following code:

        List.Sum(List.FirstN(#"Name of last step"["Sales"],[Index]))
        
You'll notice the #"Name of last step" in the code above. This will need to changed to the name of your last step in your table. You can find this on the Home Ribbon under Advanced Editor, below you can see the last step in my example before adding the custom column for a Running Total is #”Reorder Columns”:

![image](https://github.com/DOLEARY85/Power-BI-Running-Totals/assets/126701906/9eaaeea5-4b46-415d-b2aa-f8f96d2929a5)

Therefore my code would look like: 

        List.Sum(List.FirstN(#"Reordered Columns"["Sales"],[Index]))
        
This works by creating a list of the values based on the index number along with the previous index numbers, so for Index 2 the list will contain the Sales values for Index 1 & 2 (in the image below this will be {10,20}), it then adds together the values in the list.

Additional thing to note: With very large data sets this method can get extremely slow as it will iterate through each row and its previous rows to generate a list before calculating the total.

Output:

![image](https://github.com/DOLEARY85/Power-BI-Running-Totals/assets/126701906/4286e655-a1c2-469a-bdb0-a2e572bab734)

**Power Query List.Generate**

The fourth option is again to use Power Query, but instead of creating a column you can create a custom function, this can then be used as a repeatable process within your tables. It is also a lot faster that iterating through each row using a custom column so it would be ideal for larger datasets.

First you need to create a your custom function. In Queries pane on the left-hand side of Power Query - right click and select New Query - Blank Query:

![image](https://github.com/DOLEARY85/Power-BI-Running-Totals/assets/126701906/bb5feca4-7206-4fa4-a895-e96a362206bd)

After this right click the Query and select Advanced Editor. You will be presented with your query window with some code in to get you started:

![image](https://github.com/DOLEARY85/Power-BI-Running-Totals/assets/126701906/46ca93c5-8fd3-4329-b6e1-b44e35b44db6)

You can just delete the starting code and replace it with the below:

        = (values as list) as list =>
        
        let
        
            Running_Total = List.Generate
            (
                ()=> [Running_Total = values{0}, counter = 0],
                each [counter] < List.Count (values),
                each [Running_Total = [Running_Total]+ values{[counter]+1},counter = [counter] + 1 ],
                each [Running_Total]
            )
        in
            Running_Total

Finally rename your Query something memorable I’ve used fxRunningTotal.

Next you’ll need to run this function within the table where you're going to create your running total. Select the table in the queries pane and open up the advanced editor for it.

Your code will be divided into a 'let' and 'in' section. Under your last row in the let section you’re going to add some new code to run your function. Make sure to add a comma to current line before continuing.

First we need to buffer a list of the values for the column that you need a running total. You’ll see in the first line of code below  #"Renamed Columns" step is used to pull the Value from, this is the final step from my current Advanced Editor view under the 'let' section, as with the previous Power Query method you’ll need to update this to whatever your current step is:

        BV = List.Buffer( #"Renamed Columns"[Sales]),

Next we run the fxRunningTotal function you just created on the buffered list:

        Running_Total = Table.FromList(fxRunningTotal(BV),Splitter.SplitByNothing(),{"Running_Total"}),
        
Now we create a list of every column from the original table and include the Running total column:

        Columns = List.Combine({Table.ToColumns(#"Renamed Columns"),Table.ToColumns(Running_Total)}),
             
Next convert all the lists to a table, so your original table will now contain a running total. We also name the step here so we can call it in the 'in' section of the code, I’ve used Convert To Table.

        #"Convert To Table" = Table.FromColumns(Columns, List.Combine({Table.ColumnNames(#"Renamed Columns"),{"Running Total"}}))

Finally, we need to change the ‘in’ section of your code to now display the table from your new Convert To Table step.

    in
        #"Converted To Table"

Output below shows the outcome and the steps created by the new code:

![image](https://github.com/DOLEARY85/Power-BI-Running-Totals/assets/126701906/32cd9189-def2-4be7-a50f-2090874dd82d)

Full code for table execution:

        BV = List.Buffer( #"Renamed Columns"[Sales]),
        Running_Total = Table.FromList(fxRunningTotal(BV),Splitter.SplitByNothing(),{"Running_Total"}),
        Columns = List.Combine({Table.ToColumns(#"Renamed Columns"),Table.ToColumns(Running_Total)}),
        #"Convert To Table" = Table.FromColumns(Columns, List.Combine({Table.ColumnNames(#"Renamed Columns"),{"Running Total"}}))

            in
                #"Converted To Table"
