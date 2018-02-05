% Types for 
% Changlin Li

# Assumptions For This Talk

- Far more data processing code than algorithmic code
- Internal libraries/application code rather than external facing code

# What Does It Mean to Be Correct?

- Make sure the ordering of data processing steps is correct
- Make sure all necessary steps are included
- Make sure no extraneous steps are included
- Make sure that all edge cases are handled

# Example

```scala
sanitizeString : String => String
properlyCapitalizeString : String => String
readFromUserInput : String
readInputFromFile : String => String
generateDBRecord : String => DBRecord
persistDBRecord : DBRecord => Unit
```

# Is this correct?

Are files already sanitized?
```scala
readFromFile
  .andThen(properlyCapitalizeString)
  .andThen(generateDBRecord)
```

# Is this correct?
Maybe defensively just always sanitize?
```scala
if (userFlag) {
  readInputFromFile
    .andThen(properlyCapitalizeString)
    .andThen(sanitizeString)
    .andThen(generateDBRecord)
} else {
  readFromUserInput 
    .andThen(properlyCapitalizeString)
    .andThen(sanitizeString)
    .andThen(generateDBRecord)
}
```

# Is this correct?
Maybe go all out and capitalize both before and after for maximal defensiveness?
```scala
readFromUserInput 
  .andThen(properlyCapitalizeString)
  .andThen(sanitizeString)
  .andThen(properlyCapitalizeString)
  .andThen(generateDBRecord)

```
- Are you supposed to capitalize then sanitize or sanitize then capitalize?
- What about canonicalizing file names for `readFromFile`?
- And many more fun things...

# Types as Arbitrary Labels

- Don't try to encode complex runtime pre and post-condition checks in types
- Mark that they've been performed
- Provide escape hatch to remove mark

# Implementation Via Value Classes
```scala
// This isn't private in the way you'd expect
// final case class FulfillsCondition private (
//   underlying: Raw
// )
final class FulfillsCondition private (
  val underlying: Raw
) extends AnyVal

checkForCondition : Raw => Option[FulfillsCondition]
reshapeToCondition : Raw => FulfillsCondition

// Upcoming opaque types
```

# Implementation Via Tagging
```scala
// Lacks private
// trait FulfillsCondition
Tagged[Raw, FulfillsCondition]
Raw @@ FulfillsCondition
```

# Functions Should Be Total

- `Option` and `Either[E, _]` over exceptions
- But prefer pre-conditions over `Option` and `Either`

# Functions Should Be Surjective

- ```scala
  stringifyInt : Int => Option[String]
  stringifyInt : Int => String
  ```
- ```scala
  upperCase : String => String
  upperCase : String => UpperCaseString
  ```

# Be Suspicious of `f: A => A`

- :( `f : String => String`
- :) `f : String => AllCapitalsString`

# Hierarchy of Functions

- :( No help from types
  ```scala
  sanitizeString : String => String
  properlyCapitalizeString : String => String
  generateDBRecord : String => DBRecord 
  ```
- :| Totality by encoding errors
  ```scala
  sanitizeString : String => String
  properlyCapitalizeString : String => String
  generateDBRecord : String => Either[InvalidStringForDBRecord, DBRecord]
  ```
- :) Encoding pre-conditions and post-conditions
  ```scala
  sanitizeString : String => SanitizedString
  properlyCapitalizeString : SanitizedString => ProperlyCapitalizedString
  generateDBRecord : ProperlyCapitalizedString => DBRecord
  ```

# On Defensive Programming

- Defensive Programming is a sign of incomplete type usage
- Types should guide when and what checks are necessary


# Example revisited

```scala
// New functions
absoluteFilePathFromString : String => Either[StringIsNotValidFilePath, AbsoluteFilePath]

sanitizeString : String => SanitizedString
readFromUserInput : String
readInputFromFile : AbsoluteFilePath => Either[FileDoesNotExist, String]
properlyCapitalizeString : SanitizedString => ProperlyCapitalizedString
generateDBRecordFromComment : ProperlyCapitalizedString => DBRecord
persistDBRecord : DBRecord => Unit
```
# Is this correct? (revisited)

Types guide you to the functions you need

```scala

// Are files already sanitized?
readFromFile
  .andThen(properlyCapitalizeString)
  .andThen(generateDBRecord)

// Maybe defensively just always sanitize?
if (userFlag) {
  readInputFromFile
    .andThen(properlyCapitalizeString)
    .andThen(sanitizeString)
    .andThen(generateDBRecord)
} else {
  readFromUserInput 
    .andThen(properlyCapitalizeString)
    .andThen(sanitizeString)
    .andThen(generateDBRecord)
}

// Maybe go all out and sanitize both before and after for maximal defensiveness?
readFromUserInput 
  .andThen(sanitizeString)
  .andThen(properlyCapitalizeString)
  .andThen(sanitizeString)
  .andThen(generateDBRecord)

// Are you supposed to extract then sanitize or sanitize then extract?
// What about canonicalizing file names for `readFromFile`?
// And many more fun things...
```

# Notes

- Types may change as your notions of pre and post-conditions change (maybe
  you want to distinguish among different kinds of `SanitizedString`s)

# Conclusion

- Simple usage of types as labels of pre and post-conditions can ensure the
  integrity of your data processing pipeline through refactors and additions of
  new features
