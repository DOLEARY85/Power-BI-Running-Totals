# Power-BI-Running-Totals
Running totals are a very common query in Power BI and they are fairly simple to complete, there are a few options to solve this:

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

        List.Sum(List.FirstN(#"Name of last step"[#"Sales "],[Index]))
        
You'll notice the #"Name of last step" in the code above. This will need to changed to the name of your last step in your table. You can find this on the Home Ribbon under Advanced Editor, below you can see the last step in my example before adding the custom column for a Running Total is #”Reorder Columns”:

![image](https://github.com/DOLEARY85/Power-BI-Running-Totals/assets/126701906/9eaaeea5-4b46-415d-b2aa-f8f96d2929a5)

Therefore my code would look like: 

        List.Sum(List.FirstN(#"Reordered Columns"[#"Sales "],[Index]))

Output:

![image](https://github.com/DOLEARY85/Power-BI-Running-Totals/assets/126701906/4286e655-a1c2-469a-bdb0-a2e572bab734)

