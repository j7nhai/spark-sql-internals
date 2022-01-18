# UserDefinedAggregator

`UserDefinedAggregator[IN, BUF, OUT]` is a [UserDefinedFunction](UserDefinedFunction.md).

## Creating Instance

`UserDefinedAggregator` takes the following to be created:

* <span id="aggregator"> [Aggregator[IN, BUF, OUT]](Aggregator.md)
* <span id="inputEncoder"> [Encoder[IN]](../Encoder.md) ([assumed a ExpressionEncoder](#scalaAggregator))
* <span id="name"> Name (optional)
* <span id="nullable"> `nullable` flag (default: `true`)
* <span id="deterministic"> `deterministic` flag (default: `true`)

`UserDefinedAggregator` is created using [udaf](../spark-sql-functions.md#udaf) standard function.

## <span id="apply"> Creating Column for UDAF Execution

```scala
apply(
  exprs: Column*): Column
```

`apply` creates a [Column](../Column.md) with an [AggregateExpression](AggregateExpression.md) with the following:

* [ScalaAggregator](#scalaAggregator) aggregate function
* `Complete` aggregate mode
* `isDistinct` flag disabled (`false`)

`apply` is part of the [UserDefinedFunction](UserDefinedFunction.md#apply) abstraction.

## <span id="scalaAggregator"> Creating ScalaAggregator

```scala
scalaAggregator(
  exprs: Seq[Expression]): ScalaAggregator[IN, BUF, OUT]
```

`scalaAggregator` assumes the following are all [ExpressionEncoder](../ExpressionEncoder.md)s:

1. [Input Encoder](#inputEncoder)
1. [Buffer Encoder](Aggregator.md#bufferEncoder) of the [Aggregator](#aggregator)

In the end, `scalaAggregator` creates a [ScalaAggregator](ScalaAggregator.md).

`scalaAggregator` is used when:

* `UDFRegistration` is requested to [register a UserDefinedAggregator](../UDFRegistration.md#register)
* `UserDefinedAggregator` is requested to [create a Column (for UDAF execution)](#apply)