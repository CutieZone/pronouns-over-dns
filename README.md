# Pronouns over DNS

This document proposes a standard way to specify personal pronouns using DNS TXT records, primarily for personal websites.

## Record Format

Create a TXT record with the name `pronouns`. This will apply to the subdomain `pronouns.{DOMAIN}` (e.g., `pronouns.example.com`).

## Record Contents

The record's content should be a slash-separated list of pronouns: words used to refer to someone in place of their name (e.g., "she/her" or "they/them").

Each pronoun in the list is called a "segment". A segment is a string of characters without any forward slashes (`/`). Each segment should be:

- At least one character long
- Trimmed (no leading or trailing whitespace, spaces can exist within)
- Lowercased

To ensure consistency and ease of parsing, segments should appear in this order (examples provided from a `she/her` base, as those are my pronouns so it was easiest to work with):

1. **Subject:**  (Required) The pronoun used when referring to the person as the subject of a sentence (e.g., "**She** went to the store").
2. **Object:** (Required)  The pronoun used when referring to the person as the object of a verb or preposition (e.g., "Give the book to **her**").
3. **Possessive Determiner:**  (Optional) A pronoun indicating possession before a noun (e.g., "**Her** book").
4. **Possessive Pronoun:** (Optional) A pronoun indicating possession that stands alone (e.g., "The book is **hers**").
5. **Reflexive:** (Optional) A pronoun referring back to the subject (e.g., "She looked at **herself** in the mirror").

At least the subject and object pronouns are required.

**Examples:**

- `they/them` (subject, object)
- `she/her/her/hers` (subject, object, possessive determiner, possessive pronoun)
- `he/him/his/his/himself` (subject, object, possessive determiner, possessive pronoun, reflexive)

### Consistency

It is recommended to use the same number of segments for each record, however that is not mandatory.

## Multiple Records

If you use or are comfortable with multiple sets of pronouns, you can create additional TXT records with the name `pronouns`. To indicate your preferred or default set, create another TXT record named `primary.pronouns`. The content of this record should exactly match the content of one of your `pronouns` records.

Parsers should prioritize the `primary.pronouns` record if it exists. If no `primary.pronouns` record is found, parsers may choose any of the `pronouns` records or present all options to the user.

**Example:**

- `pronouns`: `she/her`
- `pronouns`: `they/them`
- `primary.pronouns`: `she/her`

## Explicit support for `any/all` pronouns

To specify that you go by `any/all` pronouns, one should set their `pronouns.` to an asterisk (`*`).

Additionally, if you wish to signify that you primarily go by one set of pronouns, however are comfortable with any pronouns, one should follow the multiple records section with one of the `pronouns.` records set to `*` with a `primary.pronouns.` of the preferred primary.

Furthermore, if you have a `(primary.)pronouns.` with `*`, you are able to provide an optionally preferred fallback pronoun set option in the form of `TXT (primary.)pronouns *; fall/back/pronouns` (example: `*; they/them`).

If no fallback pronouns are provided in the event of a `primary.pronouns.` `*`, the client parsing is encouraged to pick a random set of `pronouns.` provided within the same domain space (i.e. if a user has multiple pronoun records, and their primary is `*` a random `pronouns.` set should be selected).

If no fallback pronouns are provided, and the only existing pronoun set on the domain space is `*`, the client is encouraged to pick suitable random pronouns.

## Selection of `pronouns.` in the event of no `primary.pronouns.`

In an event such that there are multiple `pronouns.` specified yet no `primary.pronouns.`, a client is encouraged to randomly select from the available `pronouns.` sets available.

## Extra

This originated, as far as I can tell, from [this blog post by fasterthanlime](https://fasterthanli.me/articles/state-of-the-fasterthanlime-2024#all-the-personal-news). If someone can find an earlier source, I'd be happy to amend it, but this is an attempt at formalizing the format.
