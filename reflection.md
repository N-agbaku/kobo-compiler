# Reflection

## 1. Fractional numbers

The scanner decides that a `.` begins the fractional part of a number only when the character immediately after it is also a digit. This decision is made at `src/scanner.rs:147`, where the scanner checks both `self.peek() == '.'` and `self.peek_next().is_ascii_digit()`. If both conditions are true, it consumes the decimal point and continues scanning digits at `src/scanner.rs:148-152`. This means `5.25` is one `NUMBER` token. However, with the input `5.`, the character after the dot is not a digit, so the scanner does not consume the dot as part of the number. The number becomes `5`, and the remaining `.` is then handled as an invalid character. This matches section 1.4 because a fractional part requires digits after the decimal point. It also prevents `5.` from incorrectly becoming a valid number.

## 2. Line counting and EOF

The scanner changes its line counter in two places. A normal newline encountered while scanning source changes the counter at `src/scanner.rs:105`. A newline occurring inside a string changes it at `src/scanner.rs:125`. These are the two places where `line` is incremented. EOF is handled separately at `src/scanner.rs:36-41`: instead of using the scanner's current physical line, the scanner takes the line number from the last real token. Therefore, if a file ends with two blank lines, EOF carries the line of the last real token, not the line number after those blank lines. Section 6.1 requires this because EOF represents the end of the scanned token stream, and trailing whitespace or blank lines should not make the EOF token appear to belong to a later source line.

## 3. What I misunderstood

One test I failed during development was `tests/phase-1/valid/arithmetic.kobo`. Initially, the scanner's EOF token was created using the scanner's current `line` value. That meant trailing newlines caused EOF to receive a later line number than the last real token. The failure showed an expected EOF on line 4 but a reported EOF on line 23. I had misunderstood the EOF requirement by treating EOF as belonging to the physical end of the file rather than to the end of the real token stream. I fixed this in `src/scanner.rs:36-41` by taking the line from `self.tokens.last()` and using line 1 when there is no previous token.

The mistake is visible in commit `2792c1f`, where the EOF was added using `self.add(TokenType::Eof);`. I fixed it in commit `6a896c5`, where EOF instead receives `eof_line`.

### History

* Wrong: `2792c1f` — `self.add(TokenType::Eof);`
* Fixed: `6a896c5` — `line: eof_line,`
