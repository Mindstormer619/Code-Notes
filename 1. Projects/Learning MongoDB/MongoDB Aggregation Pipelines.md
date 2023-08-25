---
created: 2023-08-25 14:51
alias: 
---
#### Summary
+ 

----
# [[MongoDB]] Aggregation Pipelines

An aggregation pipeline consists of one or more stages that process documents:
- Each stage performs an operation on the input documents. For example, a stage can filter documents, group documents, and calculate values.
- The documents that are output from a stage are passed to the next stage.
- An aggregation pipeline can return results for groups of documents. For example, return the total, average, maximum, and minimum values.

> 💡 Aggregation pipelines run with the [`db.collection.aggregate()`](https://www.mongodb.com/docs/manual/reference/method/db.collection.aggregate/#mongodb-method-db.collection.aggregate) method do not modify documents in a collection, unless the pipeline contains a [`$merge`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge) or [`$out`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out) stage.

## Examples

### Quantities of medium pizza types

```js
db.orders.aggregate( [
   // Stage 1: Filter pizza order documents by pizza size
   {
      $match: { size: "medium" }
   },
   // Stage 2: Group remaining documents by pizza name and calculate total quantity
   {
      $group: { _id: "$name", totalQuantity: { $sum: "$quantity" } }
   }
] )
```

The [`$match`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/match/#mongodb-pipeline-pipe.-match) stage:
- Filters the pizza order documents to pizzas with a `size` of `medium`.
- Passes the remaining documents to the [`$group`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group) stage.

The [`$group`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/group/#mongodb-pipeline-pipe.-group) stage:
- Groups the remaining documents by pizza `name`.
- Uses [`$sum`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/sum/#mongodb-group-grp.-sum) to calculate the total order `quantity` for each pizza `name`. The total is stored in the `totalQuantity` field returned by the aggregation pipeline.

### Total order value per date, and average order quantity:

```js
db.orders.aggregate( [
   // Stage 1: Filter pizza order documents by date range
   {
      $match:
      {
         "date": { $gte: new ISODate( "2020-01-30" ), $lt: new ISODate( "2022-01-30" ) }
      }
   },
   // Stage 2: Group remaining documents by date and calculate results
   {
      $group:
      {
         _id: { $dateToString: { format: "%Y-%m-%d", date: "$date" } },
         totalOrderValue: { $sum: { $multiply: [ "$price", "$quantity" ] } },
         averageOrderQuantity: { $avg: "$quantity" }
      }
   },
   // Stage 3: Sort documents by totalOrderValue in descending order
   {
      $sort: { totalOrderValue: -1 }
   }
 ] )
```

## Details

+ A stage does not have to output 1:1 for each input document -- it may produce more documents or filter them out.
+ The same stage may appear multiple times in the pipeline, except [`$out`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/out/#mongodb-pipeline-pipe.-out), [`$merge`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge), and [`$geoNear`](https://www.mongodb.com/docs/manual/reference/operator/aggregation/geoNear/#mongodb-pipeline-pipe.-geoNear).
+ To calculate averages and perform other calculations in a stage, use [aggregation expressions](https://www.mongodb.com/docs/manual/meta/aggregation-quick-reference/#std-label-aggregation-expressions) that specify [aggregation operators](https://www.mongodb.com/docs/manual/reference/operator/aggregation/#std-label-aggregation-expression-operators).

## Aggregation Pipeline Expressions[![](https://www.mongodb.com/docs/manual/assets/link.svg)](https://www.mongodb.com/docs/manual/aggregation/#aggregation-pipeline-expressions "Permalink to this heading")

Some aggregation pipeline stages accept an [aggregation expression](https://www.mongodb.com/docs/manual/meta/aggregation-quick-reference/#std-label-aggregation-expressions), which:
- Specifies the transformation to apply to the current stage's input documents.
- Transform the documents in memory.
- Can specify [aggregation expression operators](https://www.mongodb.com/docs/manual/reference/operator/aggregation/#std-label-aggregation-expression-operators) to calculate values.
- Can contain additional nested [aggregation expressions.](https://www.mongodb.com/docs/manual/meta/aggregation-quick-reference/#std-label-aggregation-expressions)

## Limitations

+ Each document in the result set is subject to the 16MB BSON document size limit. The limit only applies to the returned documents. During the pipeline processing, the documents may exceed this size.
+ 1000 stages maximum in a single pipeline.
+ If [`allowDiskUseByDefault`](https://www.mongodb.com/docs/manual/reference/parameters/#mongodb-parameter-param.allowDiskUseByDefault) is set to `true`, pipeline stages that need more than 100MB memory will write temp files to disk by default. To disable, use `{allowDiskUse: false}` option within the stage. If `allowDiskUseByDefault` is set to `false`, >100MB memory stages will raise an error unless overridden by the stage.

## See More

More info can be found in the book: [Practical MongoDB Aggregations by Paul Done](https://www.practical-mongodb-aggregations.com/front-cover.html).

----

## References
+ https://www.mongodb.com/docs/manual/core/aggregation-pipeline/#std-label-aggregation-pipeline