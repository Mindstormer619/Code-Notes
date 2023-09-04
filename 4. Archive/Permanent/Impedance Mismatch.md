# Impedance Mismatch
> Created: 2023-05-18 18:59

A program requires _in memory objects_ to represent data. The database might be representing data in a different way -- for example, via relational structures in a [[Database Data Models#Relational|SQL DB]] .

This means that we need to have a layer in the middle which handles the translation to and from the DB to the program. This is traditionally called an [[ORM|Object Relational Mapping]]. The problem itself of actually requiring an ORM is called an impedance mismatch.

----

## References
1. [[Designing Data Intensive Applications - Data Models and Query Languages]]