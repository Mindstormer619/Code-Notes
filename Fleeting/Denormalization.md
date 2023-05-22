# Denormalization
> Created: 2023-05-18 19:10

In database design, denormalization is a technique that violates the principles of normalization by adding redundant data to a database in order to improve performance. This can be done by storing data in multiple places, or by storing data in a less normalized format.

Denormalization is often used in databases that are heavily read-intensive, as it can improve the performance of queries that access frequently used data. However, it can also make the database more complex and difficult to maintain.

There are two main types of denormalization:

-   **Data duplication:** This involves storing the same data in multiple places. For example, a database might store the customer's address in both the customer table and the order table.
-   **Data summarization:** This involves storing aggregated data in a single place. For example, a database might store the total number of orders placed by each customer in a single table.

Denormalization should be used with caution, as it can have a number of drawbacks, including:

-   **Increased data storage:** Denormalization can increase the amount of data that needs to be stored in the database. This can lead to increased costs for storage and backup.
-   **Reduced data integrity:** Denormalization can make it more difficult to maintain the integrity of the data in the database. This is because changes to the data in one place need to be replicated to all other places where the data is stored.
-   **Increased complexity:** Denormalization can make the database more complex and difficult to understand. This can make it more difficult to troubleshoot problems and to make changes to the database.

Denormalization should only be used when the benefits outweigh the drawbacks. It is important to carefully evaluate the needs of the application before deciding whether or not to denormalize the database.

----

## References
1. Bard