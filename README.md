# Tejaswi-bigdata-final-project
- This repo consists of processing text data using Spark and Python.

## Author:
- Tejaswi Reddy Kandula 

## Text Data
- Text Data Source: (TheLittleWarriorbyP.G.Wodehouse)[https://www.gutenberg.org/files/6837/6837-0.txt]

## Tools and Languages:
- Language: Python
- Tools: Pyspark, Databricks Notebook, Pandas, MatPlotLib, Regex, Urllib

## Databrick Community:
[Databricks](https://community.cloud.databricks.com/?o=738325624314186#notebook/123542383545008/command/900976778029719)

## Process
## Gathering Data
urllib - It is used to open URL
1.urllib.request library is used to request the data or pull data from the text data's url. Once the data is pulled, it is stored in a temporary file called 'tejaswi.txt' and will get the text data from 'The Little Warrior' from gutenberg.org site.
```
# Read the data from the URL and print it
import urllib.request
# Open a connection URL using urllib
urllib.request.urlretrieve("https://github.com/Teju2404/tejaswi-bigdata-final-project/blob/main/TheLittleWarrior.txt" , "/tmp/tejaswi.txt")
```
2.The data has been saved,The dbutils is used to work with Filesystems.dbutils.fs.mv which is used to transfer the data in the temporary data to a new site called data.
-- dbutils.fsmv(from: String, to: String, recurse: boolean = false): boolean - Which moves a file or directory, across the FileSystems.
```
dbutils.fs.mv("file:/tmp/tejaswi.txt","dbfs:/data/tejaswi.txt")
```
3.To transfer the data file into Spark, using sc.textfile into TejaswiRDD(Resilient distributed Systems) as shown below,The elements will run and operate on multiple nodes to do a parallel processing on a cluster.
--  textFile = sc.textFile("/my/directory/*.txt"),
    textFile2 = sc.wholeTextFiles("/my/directory/")
Used to Read either one text file from HDFS, a local file system or or any Hadoop-supported file system URI with textFile(), or read in a directory of text files with      wholeTextFiles().
```
tejaswiRDD = sc.textFile("dbfs:/data/tejaswi.txt")
```
## Cleaning the Data

4. The data above consists of capitalized words, sentences, punctuations, and stopwords.
Steps for cleaning the data:
We need to split each line by its spaces, changing the capitalized to lower case,filtering out empty lines and splitting the sentences into words.
```
# flatmap each line to words
wordsRDD=tejaswiRDD.flatMap(lambda line : line.lower().strip().split(" "))
```
5.Removing all the punctuation.Regular expression is used It is done by using library re.It is used to view anything that is not a letter.
```
import re
cleanTokensRDD = wordsRDD.map(lambda w: re.sub(r'[^a-zA-Z]','',w))
```
6.We need to remove the stopwords. PySpark by default knows about the stopwords.We need to import the library StopWordsRemover from pyspark.After importing filter out the words from the library.
```
from pyspark.ml.feature import StopWordsRemover
remover =StopWordsRemover()
stopwords = remover.getStopWords()
cleanwordRDD=cleanTokensRDD.filter(lambda w: w not in stopwords)
```
## Processing the data

7. Next step after cleaning data is processing data.we will map our words into intermediate key-value pairs. we will create a pair consisting of ('<word>', 1) for each word element in the RDD. We can create the pair RDD using the map() transformation with a lambda() function to create a new RDD.
This will look like: (word,1), once we map it.

```
IKVPairsRDD= cleanwordRDD.map(lambda word: (word,1))
```
8. In this step we will perform Reduce by key operation. The key means word.Everytime we will keep the first word count.If the word appears again we will be removing the latest one and keep the first word count.
```
wordCountRDD = IKVPairsRDD.reduceByKey(lambda acc, value: acc+value)
```
9.In this step we will retrieve the elements.collect() action function is used to retrieve all elements from the dataset(RDD/DataFrame/Dataset) as a Array(row) to the driver program.
```
results = wordCountRDD.collect()
```
## Finding Useful Data
- Sorting a list of tuples by the second value which will be used to reposition of tuples position in the list.The second values of tuples are in ascending order.Top 12 words are displayed below using print.

```results.sort(key=lambda x:x[1])
results.reverse()
print(results[:12])
```
- To Graph the data we will use the library mathplotlib.We can display any type of graph(bar,scatter,pie etc) by plotting x axis and y axis.
```
mostCommon=results[1:5]
word,count = zip(*mostCommon)
import matplotlib.pyplot as plt
fig = plt.figure()
plt.bar(word,count,color="Lavender")
plt.xlabel("Words")
plt.ylabel("Number of times used")
plt.title("Most used words in Little Warrior")
plt.show()
```
# Charting Results
-![Sorting](https://github.com/Teju2404/tejaswi-bigdata-final-project/blob/main/sort.PNG)
-![Results](https://github.com/Teju2404/tejaswi-bigdata-final-project/blob/main/results.PNG)

# References
- [Matplotlib](https://dzone.com/articles/types-of-matplotlib-in-python)
- [Python](https://www.analyticsvidhya.com/blog/2020/02/beginner-guide-matplotlib-data-visualization-exploration-python/)
- [Dbutils](https://docs.microsoft.com/en-us/azure/databricks/dev-tools/databricks-utils)
