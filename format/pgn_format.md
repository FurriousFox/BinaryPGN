# pgn format

## specs

### character set

- subset of Latin 1 charset
- includes the standard seven bit ASCII character set for the 32 control character code values from zero to 31.
- The 95 printing character code values from 32 to 126 are also equivalent to seven bit ASCII usage.
- Code value 127, the ASCII DEL control character, is not used for PGN data representation.
- The 32 ISO 8859/1 code values from 128 to 159 are non-printing control characters. They are not used for PGN data representation.
- The 32 code values from 160 to 191 are mostly non-alphabetic printing characters and their use for PGN data is discouraged as their graphic representation varies considerably among other ISO Latin sets.
- Finally, the 64 code values from 192 to 255 are mostly alphabetic printing characters with various diacritical marks; their use is encouraged for those languages that require such characters.
<br>
- Only four of the ASCII control characters are permitted in PGN import format; these are the horizontal and vertical tabs along with the linefeed and carriage return codes.
<br>
- tab characters may not appear inside of string data.

### formats

**export format** should be byte equivalent when parsed and (re-)exported in export format, this doesn't have to be the case with **import format** (generally written by a human)

### newlines

note baroque representations for a newline (i.e. make sure to handle \n \r\n and \r correctly)

### comments

(note: comments cannot appear inside any token)

- "rest of line" comment: starts at ";" and continues to the end of the line.
example:

  ```pgn
  1. d4; closed system
  1...d5 2. c4
  ```

- left brace character and continues to the next right brace character

### escaping

- "%" as **first** character, ignoring everything (excluding the newline)

### tokens

- string token
  - format: `"{text}"`
  - notable cases:
    - `""` (empty string)
    - `"\""` (an quote escaped using a backslash)
    - `"\\"` (a backslash escaped using a backslash)

  - commonly used as tag pair values (e.g. `Black "mzhadi"`)
  - may not contain non-printing characters (like newline and tab)
  - (currently!) a string is limited to a maximum of 255 characters of data.
<br>
- integer token
  - sequence of one or more decimal digit characters
  - format `123`
  - used to help represent move number indications
  - terminated just prior to the first character after the integer digit sequence
<br>
- period token
  - literally `.`, it's a token on itself
  - used for move number indications
<br>
- asterisk token
  - literally `*`
  - one of the game termination markers: incomplete game or a game with an unknown or otherwise unavailable result
<br>
- bracket token
  - `[` and `]`
  - around tag pair values (e.g. `[` (bracket token) `Black "mzhadi"` (tag pair value) `]` (bracket token))
<br>
- parenthesis token
  - `(` and `)`
  - delimit "Recursive Annotation Variations" (e.g. `(8... b5)`)
  - notable cases:
    - `(8... Qc7 (subvariation (subsubvar...) ))`
    (rav-begin/data/rav-end structures can be nested)
<br>
- angle bracket token
  - `<` and `>`
  - reserved for future expansion
  (not needed to implement ig, as there isn't any implementation yet)
<br>
- Numeric Annotation Glyph / NAG token
  - format: `$` + one or more digit characters (uint8; 0-255)
  - example: `$142`
  - example in context: `1. d4!? $18 $132 *`
  meaning: first move, to d4, interesting move (!?), white is winning (\$18), white has moderate counterplay (\$132), the other hasn't done a move (\*)
<br>
- symbol token
  - starts with a letter or digit character and is immediately followed by a sequence of zero or more symbol continuation characters
  - continuation characters are: A-Za-z 0-9 `_` `+` `#` `=` `:` `-`
  - symbol is limited to 255 characters
  - terminated just prior to the first non-symbol character following the symbol character sequence

<br>

### PGN database file

a sequential collection of zero or more PGN games,
an empty file is still a pgn database (of zero games)

### PGN game

two sections:

- tag pair section
- movetext section

### tag pair section

#### tag pair specification

- **zero** or more tag pairs
- format: `[`<sub><sup>(zero or more white spaces (space / newline / tab))</sup></sub> symbol token <sub><sup>(zero or more white spaces (space / newline / tab))</sup></sub> string token <sub><sup>(zero or more white spaces (space / newline / tab))</sup></sub>`]`
- export format: `[`symbol token, " " (1 space), string token`]`
- tag names (symbol) is case sensitive (like all symbols)
- notable cases:
  - multiple tag pairs on one line
  `[White "Pirate91"][Black "mzhadi"]`
  - multiple lines on one tag pair

    ```pgn
    [
          White"Pirate91"

            ]
    ```

- (in export format) a single empty line follows the last tag pair
- `:` in the string token in a tag pair means a sequence of items
for example: `[White "Bob:Alice"]`
\
The above example means that both Bob and Alice both represented white.
- the colon should not otherwise appear in the string

#### all tag pair tags

32 (7 (STR) + 2 (common) + 2 lichess specific) tags[^4]
\
the Seven Tag Roster (in (export) order) (7 tags): [^9]

- Event
`[Event "name of event"]`
- Site
`[Site "location of event"]`
- Date (starting date of the game)
`[Date "YYYY.MM.DD"]`
Question marks are used in place of unknown digits
for example: `[Date "2005.01.??"]`
- Round (either a hyphen (-) or round or multipart round)
examples:
  - `[Round "-"]`
  - `[Round "1"]`
  - `[Round "2.4.7.5.2"]`
- White (may not contain non-printing characters (like newline and tab))
If name is unknown: `[White "?"]`
If name is known: `[White "name of white player"]`
If names are known: `[White "name of first white player:name of second white player"]`
- Black (may not contain non-printing characters (like newline and tab))
If name is unknown: `[Black "?"]`
If name is known: `[Black "name of black player"]`
If names are known: `[Black "name of first black player:name of second black player"]`
- Result
White won: `[Result "1-0"]`
Black won: `[Result "0-1"]`
Game drawn: `[Result "1/2-1/2"]`
No result (yet): `[Result "*"]`

\
common (but non-official) tags:[^5] [^6]

- WhiteTitle
`[WhiteTitle "IM"]` (or any other title)
`[WhiteTitle "-"]` (no title)
- BlackTitle
`[BlackTitle "IM"]` (or any other title)
`[BlackTitle "-"]` (no title)
- WhiteElo
followed by an int value [^1] [^2]
`[WhiteElo "1284"]` (or any other FIDE Elo rating)
`[WhiteElo "-"]` (no (known) FIDE Elo rating)
- BlackElo
followed by an int value [^1] [^2]
`[BlackElo "1284"]` (or any other FIDE Elo rating)
`[BlackElo "-"]` (no (known) FIDE Elo rating)
- WhiteUSCF
followed by an int value [^1] [^2]
`[WhiteUSCF "1284"]` (or any other USCF rating)
`[WhiteUSCF "-"]` (no (known) USCF rating)
- BlackUSCF
followed by an int value [^1] [^2]
`[BlackUSCF "1284"]` (or any other USCF rating)
`[BlackUSCF "-"]` (no (known) USCF rating)
- WhiteNA
`[WhiteNA "example@example.com"]` (or any other e-mail or network address(es))
`[WhiteNA "-"]` (no (published) e-mail or network addresses)
- BlackNA
`[BlackNA "example@example.com"]` (or any other e-mail or network address(es))
`[BlackNA "-"]` (no (published) e-mail or network addresses)
- WhiteType (whether the player is a human or a computer)
`[WhiteType "human"]`
`[WhiteType "computer"]`[^3]
- BlackType (whether the player is a human or a computer)
`[BlackType "human"]`
`[BlackType "computer"]`[^3]
<br>

- EventDate (starting date of the event)
_often_ similar to Date tag
`[EventDate "YYYY.MM.DD"]`
- EventSponsor
`[EventSponsor "Acme Corporation"]`
- Section (playing section in tournament)
`[Section "Open"]`
`[Section "Reserve"]`
- Stage (stage in tournament (if applicable))
`[Stage "Preliminary"]`
`[Stage "Semifinal"]`
- Board (number to identify physical board played at)
`[Board "17"]`
- Opening (which opening was played)
`[Opening "Benoni Defense: Old Benoni"]`
- Variation (which variation was played)
`[Variation "Old Benoni"]`
- SubVariation (which sub-variation was played)
`[SubVariation "Old Benoni"]`
- ECO
reference corresponding to the opening described in the Encyclopedia of Chess Openings
  - format: A/B/C/D/E + 2 digits
  - example: `[ECO "B01"]`
- NIC
similar to ECO, but for New in Chess, [a list of NIC keys is located at the German Wikipedia](https://de.wikipedia.org/wiki/NIC-Schl%C3%BCssel)
`[NIC "FR4"]`
`[NIC "QP14"]`
- Time (**local** start time)
`[Time "HH:MM:SS"]`
`[Time "17:41:13"]`
`[Time "17:41:??"]`
- UTCTime (start time in UTC timezone)
`[UTCTime "15:41:13"]`
- UTCDate (start date in UTC timezone)
`[UTCDate "2024.10.21"]`
- TimeControl [^8]
  - Unknown time control `[TimeControl "?"]`
  - No time control (e.g. unlimited time) `[TimeControl "-"]`
  - Sudden death time control (no increment) `[TimeControl "seconds"]`
  - Incremental time control (with increment) `[TimeControl "initial time in seconds+increment in seconds"]`
  - Sandclock time control (used for semi-accurate time control, like a sandclock)
(format: `*time in seconds`) `[TimeControl "*180"]`
- SetUp
`[SetUp "0"]` The game has started from the usual initial position
`[SetUp "1"]` The game has started from the inital position defined in the "FEN" tag pair
- FEN (defines a different initial position)
`[FEN "Forsyth-Edwards Notation for the starting position"]`
`[FEN "rnbqkbnr/pppppppp/8/8/4P3/8/PPPPP1PP/RNBQKBNR w KQkq - 0 1"]`
- Termination (reason for game termination)
example: `[Termination "rules infraction"]`
The 6 reasons for termination used by Lichess are "Unterminated", "Abandoned", "Time forfeit", "Normal", "Rules infraction" and "Unknown"
- Annotator (the name(s) of the annotator(s) of the game)
`[Annotator "Eric Rosen"]`
`[Annotator "Eric Rosen:Magnus Carlsen"]`
- Mode (the way the game was played)
includes but isn't limited to:
`[Mode "OTB"]` (over the board)
`[Mode "EM"]` (electronic mail)
- PlyCount (number of ply (half-moves) in the game)
`[PlyCount "17"]`

\
tags used by Lichess: [^10]

[^1]: still using a string token, not integer token, as a string token is required for tag pairs per spec
[^2]: though the value commonly represents an int, it may be any other value (including ones representing non-numerical values)
[^3]: though "human" and "computer" are common values, other values are allowed too
[^4]: tag pairs may include but are not limited to these tags!
[^5]: these tag pairs are recommended to be used like this, but might be used differently (assume that these specifications will be violated)
[^6]: the STR (Seven Tag Roster) is in a predefined order, other tags should be ordered in ASCII order by tag name [^7]
[^7]: this means `B` (ASCII 66) comes before `a` (ASCII 97)
[^8]: this definition is missing some nuances [to be found in the official documentation](https://www.saremba.de/chessgml/standards/pgn/pgn-complete.htm#c9.6.1)
[^9]: these tags are required for archival storage, but not strictly required in either export or import format
[^10]: these tag pairs are used currently like this by Lichess, but might be used differently in the future or by other people (assume that these specifications will be violated)
