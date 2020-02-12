# Sentiment Analysis in Power BI via SQL Server Machine Learning Services 

The database you need to recreate the example used in the blog can be found in the database folder in this repo. The example was done using R but if you want to use Python instead then you need to replace step 3 with the instructions given below:

&nbsp;&nbsp;&nbsp;&nbsp;&nbspSet the values of the @Query and @RScript variables.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp ou need to set the @Query variable with the string that represents the T-SQL for your input dataset. You &nbsp;&nbsp;&nbsp;&nbsp;&nbsp lso need to set the @RScript variable to hold the R code that will be used to perform the sentiment &nbsp;&nbsp;&nbsp;&nbsp;&nbsp nalysis. The T-SQL script for the @Query variable is as follows:

'''
      SET @Query = 'SELECT [id], [text] FROM [dbo].[SentimentData]'
'''

It is a simple SELECT statement that grabs the data from the [dbo].[ SentimentData] table needed for our R script. Next, we define the R code needed to score the data. Here is the code:

'''
SET @RScript = '
              dfInput$text = as.character(dfInput$text)
              sentimentScores <- rxFeaturize(
                                       data = dfInput, 
                                       mlTransforms = getSentiment(vars = list(SentimentScore = "text"))
                                                  )
              sentimentScores$text <- NULL
              dfOutput <- cbind(dfInput, sentimentScores)
              ‘
'''

The script is only four lines long because the pre-trained model does the heavy lifting. In the above R script, the first line changes the text field in the dfInput data frame to a character data type. By default, R converts any character-based fields into a data type called a “factor” when creating data frames. Factor fields are used for categorical data. Underneath the hood, the data is stored using a method similar to the dictionary encoding used in DAX (Data Analysis Expressions). Each unique element in the field is replaced with an integer value, and a map is created that maps the index value to the unique element it replaces. This technique is beneficial because integers are more efficient to work with, and have a lower memory footprint than long strings.

Factors are good for data analysis, but not for the sentiment analysis we are doing. We need access to the actual text in the text field, and that is not easily done when the text field is stored as a factor. Converting the text field to a character data type alleviates the problem.

The next line (actually the next four, due to formatting) performs the sentiment analysis. The workhorse functions are the rxFeaturize() and getSentiment() RevoScaleR functions. The rxFeaturize() function enables us to access data that has undergone a machine learning data transformation via MicrosoftML. It specifies the machine learning transformation that is being performed in the mlTransforms argument; which, in this example, is a getSentiment() transformation. The vars argument in getSentiment() is used to specify the fields in your dataset that you want to score. If you specify a named list, as we did in this example, then the name of each element in your list represents the name of the column that will hold the sentiment score, and the value represents the field you want to perform the sentiment on. Note: You can perform sentiment analysis on multiple fields at once. You just need to add an element to your named list using the method previously describe. The getSentiment() function will return a numeric value between 0 and 1 for each sentiment analysis it performs. The closer to 0 the value is, the more negative the sentiment, and the closer to 1 the value is, the more positive the sentiment.

