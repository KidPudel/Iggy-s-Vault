# Base64 Encoding

An encoding scheme that represents binary data as a string of 64 printable ASCII characters.

## What it does

Converts binary to ASCII in three steps:
1. Break binary into 6-bit chunks
2. Map each 6-bit chunk to one of 64 characters (`A-Z`, `a-z`, `0-9`, `+`, `/`)
3. Pad output with `=` if the input length is not a multiple of 3 bytes

Decoding reverses the process. Output is ~33% larger than input.

Base64URL replaces `+` with `-` and `/` with `_` for safe use in URLs and HTTP headers (used in JWT).

Encoding is not encryption — anyone can decode it.

## Sources

- https://www.rfc-editor.org/rfc/rfc4648

## Related

- [[JWT]]
- [[Authorization header]]
- [[headers]]

## Process

- What problem does encoding binary as ASCII solve when transmitting over HTTP or SMTP?
- Where does Base64URL differ from standard Base64, and why does that matter for JWTs?
- Base64 is not encryption — what does that mean for data stored in JWT payloads?
