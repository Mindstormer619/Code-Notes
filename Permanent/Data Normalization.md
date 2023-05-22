# Data Normalization
> Created: 2023-05-18 19:05

_Normalization_ is the idea of de-duplication -- you don't use free text, and instead use identified objects to represent stored data, with one object for each separate instance of that data. For example, instead of having `town` be a text field, you link it to a `Towns` table which contains the list of allowed towns.

## Advantages

+ Consistent style and spelling
+ Avoids ambiguity
+ Ease of updating if it ever needs to be changed: Bangalore → Bengaluru
+ Localization support: you can translate the entire table at once.
+ Better search.

## Disadvantages

+ Read time performance. If you want data that requires summarizing info in the table, or requires linking through to a foreign table to get the info, the joins / summarization queries can take much longer than just reading from a duplicate, single field that contains the data directly.
+ 

----

## References
1. [[Designing Data Intensive Applications - Data Models and Query Languages]]