# Jetpack Compose Modifiers
> Created: 2022-01-12 00:57

For modifiers, the order in which they are applied is the *reverse* of their declaration.

```kotlin
modifier = Modifier
	.border(width = 10.dp, color = Color.Green)
	.padding(10.dp)  
	.clickable {  }  
	.align(Alignment.CenterHorizontally)
```

In the above example, the element is:
1. made to align to the center horizontally
2. made clickable
3. padding is added around the clickable element (the padding area itself is non-clickable)
4. border is added *inwards* for 10dp towards the element.

Result (see the "Click Me" text element):
![[Pasted image 20220112010135.png]]

The reason for this is -- consider `modifier` is passed in from a parent element. In that case, the modifications that the parent has made must be applied *last*, and the modifications on a child element must be applied first.

```kotlin
@Composable
fun MyFancyButton(modifier: Modifier = Modifier) {
  Text(
    text = "Ok",
    modifier = modifier
      .clickable(onClick = { /*do something*/ })
      .background(Color.Blue, RoundedCornerShape(4.dp))
      .padding(8.dp)
  )
}
```

In the above example, any modifiers already present from the parent composable will be applied at the *end*, because the ones closest to the "root" are applied last (reverse order).

----

## References
1. https://developer.android.com/jetpack/compose/modifiers
2. https://stackoverflow.com/questions/64206648/jetpack-compose-order-of-modifiers