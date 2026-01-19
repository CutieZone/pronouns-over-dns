# Pronouns over DNS

This document proposes a standard format for specifying personal pronouns using DNS TXT records, primarily for personal domains.

## Terminology

- `domain`: A domain name associated with a person, typically used for personal websites or email addresses.
- `pronouns`: A set of personal pronouns that a user uses to refer to themself
- `record`: A DNS TXT record that contains information about the pronouns.
- `pronoun set`: A set of associated pronouns, such as "she/her", "he/him", "they/them", etc.
- `user`: The person whose pronouns are being specified.
- `implementation`: A software or service that retrieves and interprets pronoun records from DNS.

## Record Names

The record must be a TXT record at `pronouns.<domain>`. For example, for the domain `example.com`, the record would be located at `pronouns.example.com`. Multiple records may be present to indicate multiple available pronoun sets, their precedence by default follows the [selection process](#selection-process) defined below, but [tags](#tags) may be used to indicate preference.

## Format

The basic syntax for a format is as follows:

```ebnf
record = [ base_record ], [ comment ];

base_record =
    | wildcard
    | none
    | pronoun_set, [ tags ];

pronoun_set = value, "/", value, [ "/", value ], [ "/", value ], [ "/", value ];

tags = ";", tag, { ";", tag }

tag = "preferred" | "plural";

value = [a-z]+;

comment = "#", { any-non-newline-character };

wildcard = "*";
none = "!";
```

The canonical representation of a record's values and tags contains only lowercase letters, with no whitespace. Parsers, however, should normalise input to be lowercase, and should ignore leading or trailing whitespace. The canonical representation of the tags component should contain no duplicate or empty tags, but again parsers should expect these and normalise input to remove duplicates and empty tags.

A pronoun set must include a subject (e.g., "she", "he", "they") and an object (e.g., "her", "him", "them") pronoun at minimum. The possessive determiner (e.g., "her", "his", "their"), the possessive pronoun (e.g., "hers", "his", "theirs"), and reflexive pronoun (e.g., "herself", "himself", "themself") are optional. These components must be provided in the order listed above if they are included.

Some examples of valid and invalid records:

```diff
# Valid:

+ she/her
+ he/him/his/his/himself;preferred
+ they/them/their/theirs/themself
+ they/them;preferred;plural
+ *
+ !
+ ze/zir/zir/zirself

# Valid but non-canonical:

+ SHE/HER # -> she/her
+ SHE /    HER # -> she/her
+ he/him;;;preferred # -> he/him;preferred

# Invalid:

- she/her/
- she
- they/them/their/theirs/themself/extra
- she/her;unknown-tag
```

Note that each example here is an isolated example provided grouped for brevity. In practice, a `!` record may not be provided with other records and must be the sole record if present, per [None Records](#none).

## Tags

The following tags are defined:

### `preferred`

Indicates that this pronoun set is the user's preferred pronoun set. If multiple pronoun sets are tagged with `preferred`, a record is picked in accordance with the [selection process](#selection-process) defined below.

### `plural`

Indicates that this pronoun set uses a plural verb agreement (e.g., "are", "were" rather than "is", "was", as used in "they are", "they were"). For simplicity for users, parsers are expected to recognise that "they/them" has a plural verb agreement even if the tag is not present, but the tag may be used to indicate that other pronoun sets are plural in nature.

## Wildcard and None Records

### Wildcards

A wildcard record (`*`) indicates that the user is open to being referred to using any pronoun set. This is a standalone record, and must not be combined with any other values or tags. Comments are still allowed.

When a wildcard is the only record present, the neutral pronoun set (they/them) is assumed to be the de facto preference.

Other records may be provided alongside a wildcard record to indicate preference. In the case that a wildcard is not the only record:

- If there is one other record, that record has implied preference as the default way to refer to the user.
- If there are multiple other records, the [selection process](#selection-process) is used to determine which record has preference.
- If there are multiple other records, and one is tagged with `preferred`, that record has preference.
- If there are multiple other records, and multiple are tagged with `preferred`, the [selection process](#selection-process) is used to determine which record has preference among those tagged with `preferred`.

For example:

| Record(s) Present                                         | Preferred Pronoun Set                                      |
| --------------------------------------------------------- | ---------------------------------------------------------- |
| `*`                                                       | Any, default they/them                                     |
| `*`, `she/her`                                            | Any, default she/her                                       |
| `*`, `she/her`, `he/him`                                  | Any, default she/her or he/him depending on implementation |
| `*`, `she/her;preferred`, `he/him`                        | Any, default she/her                                       |
| `*`, `she/her;preferred`, `he/him;preferred`, `they/them` | Any, default she/her or he/him depending on implementation |

### None

A none record (`!`) indicates that the user does not wish to be referred to using any pronouns, but would prefer to explicitly be referred to by their name. This is also a standalone record, and must not be combined with any other values or tags. Comments are still allowed.

## Selection Process

When multiple pronoun sets are available, and no more specific means of determining priority is available, record preference is left as intentionally undefined behaviour and is left to the implementation to decide. Possible strategies include:

- Selecting the first record returned by the DNS resolver.
- Randomly selecting one of the available records.
- Returning a pseudo-random but consistent selection based on a hash of the domain name.

## Parser Conversions

Implementations should, in the case of certain common pronoun sets defined below, translate the given input into a more correct form. This is to account for common uses of pronoun sets which do not strictly conform to the format defined above, but are widely used.

| Input  | Converted To         |
| ------ | -------------------- |
| it/its | it/it/its/its/itself |
