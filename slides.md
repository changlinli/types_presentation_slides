% Moving Beyond Defensive Programming
% Changlin Li (mail@changlinli.com)

## What this talk is about

Using types to solve many of the same problems classic defensive programming
tries to solve, but better

## Assumptions for this talk

- Far more data processing code than algorithmic code
- Internal libraries/application code rather than external facing code

## What does it mean to be correct?

- Make sure the ordering of data processing steps is correct
- Make sure all necessary steps are included
- Make sure no extraneous steps are included
- Make sure that all edge cases are handled

## Central Thesis

- Pre and post conditions are an important tool in thinking about correctness
- Types can serve as machine-checked labels for conditions
- There are some rules of thumb that ensure you're fully utilizing types to
  label conditions

## Example

```scala
sanitizeString : String => String
properlyCapitalizeString : String => String
readFromUserInput : String
generateDBRecord : String => DBRecord
persistDBRecord : DBRecord => Unit
```

## Is this correct?

Did we forget to sanitize?
```scala
readFromUserInput
  .andThen(properlyCapitalizeString)
  .andThen(generateDBRecord)
  .andThen(persistDBRecord)
```

## Is this correct?

Maybe defensively just always sanitize?
```scala
readFromUserInput
  .andThen(properlyCapitalizeString)
  .andThen(sanitizeString) // <--- DIFF
  .andThen(generateDBRecord)
  .andThen(persistDBRecord)
```

## Is this correct?

- Are you supposed to capitalize then sanitize or sanitize then capitalize?
- Maybe just sanitize then capitalize and then sanitize again
- And many more fun things...

## Defensive Programming (in practice)

- Don't know if it conforms to a pre-condition so check
- Don't have a way of signaling to downstream code that post-conditions hold
- Redundant checks and cargo culting of other check patterns in the codebase (null checks, exceptions, etc.)

## Types as simple labels

- Don't try to encode complex runtime pre and post-condition checks in types
- Constrain ways of marking that they've been performed
- Provide escape hatch to remove mark

## Implementation via value classes

```scala
// This isn't private in the way you'd expect
// final case class SanitizedString private (
//   underlying: String
// )
final class SanitizedString private (
  val underlying: String
) extends AnyVal

object SanizitedString {
  def sanitizeString(str: String): SanitizedString = {
    // ...
    new SanizitedString(...)
    // ...
  }
}

```

## Functions should handle their entire source type

- A.k.a. totality
- This ensures pre-conditions
- `Option` and `Either[E, _]` over exceptions
- But prefer stricter source type over `Option` and `Either`

## Functions should occupy their entire target type

```scala
// Bad
upperCase : String => String
// Good
upperCase : String => UpperCaseString
```

- This captures post-conditions
- A.k.a. "surjectivity" or "onto"

## Frown on `f: A => A`

- :( `f : String => String`
- :) `f : String => AllCapitalsString`

## Think hard about () for validation

```scala
def validateItem(item: SomeType): Unit = {
  if (itemIsNotValid(item)) {
    throw new Exception("This is now valid")
  } else {
    ()
  }
}
```

- `Option[Unit]`
- `Either[Something, Unit]`

## Think hard about () for validation

```scala
val result = {
  validateItemWithCheck0(item)
  validateItemWithCheck1(item)
  doActualThingWithItem(item)
}

for {
  _ <- validateItemWithCheck0(item)
  _ <- validateItemWithCheck1(item)
  result <- doActualThingWithItem(item)
} yield result
```

## Bad Functions

```scala
properlyCapitalizeString : 
  String => String

generateDBRecord : 
  String => DBRecord 
```

## Okay Functions

```scala
properlyCapitalizeString : 
  String => String

generateDBRecord : 
  String => Either[InvalidString, DBRecord]
```

## Good Functions

```scala
properlyCapitalizeString : 
  SanitizedString => ProperlyCapitalizedString

generateDBRecord : 
  ProperlyCapitalizedString => DBRecord
```

## Example revisited

```scala
sanitizeString : String => SanitizedString
readFromUserInput : String
properlyCapitalizeString : SanitizedString => ProperlyCapitalizedString
generateDBRecordFromComment : ProperlyCapitalizedString => DBRecord
persistDBRecord : DBRecord => Unit
```
## Is this correct? (Revisited)

Can you tell from the types whether this will work?
```scala
// String
readFromUserInput
  // SanitizedString => ProperlyCapitalizedString
  .andThen(properlyCapitalizeString)
  // String => SanitizedString
  .andThen(sanitizeString)
  // ProperlyCapitalizedString => DBRecord
  .andThen(generateDBRecord)
  // DBRecord => Unit
  .andThen(persistDBRecord)
```
## Is this correct? (Revisited)

This is the correct sequence
```scala
// String
readFromUserInput
  // String => SanitizedString
  .andThen(sanitizeString)
  // SanitizedString => ProperlyCapitalizedString
  .andThen(properlyCapitalizeString)
  // ProperlyCapitalizedString => DBRecord
  .andThen(generateDBRecord)
  // DBRecord => Unit
  .andThen(persistDBRecord)
```

## What have we gained?

<!-- if a piece of code gives me back a `SanitizedString` or a `class` holds a
`SanitizedString` field I'm pretty sure I can use it in the downstream parts of
my data processing pipeline without fear -->
+ Re-use other code with less fear

## Additional techniques and references

- Smart constructors (variation of factories)
- `Tagged` and opaque types
- Phantom types
- Path dependent types
- Refinement types (`refined`)
- Bill Venners' `scalactic` library
- _But you don't NEED to use these!_
- Talk to me afterwards/email me if you want to know more about these

## Conclusion

- Simple usage of types as labels of pre and post-conditions can ensure the
  integrity of your data processing pipeline through refactors and additions of
  new features
- Move institutional knowledge of what checks need to be performed to the type
  system

## Questions?

Example questions below (if you can't think of any but would really like your
voice heard/way to let me sneak in content past the 15 min mark):

- What about pre-conditions that involve relationships among different
  arguments?
- When should you NOT use these techniques?
- This isn't a panacea, what classes of errors aren't covered by these
  techniques?
