A **grapheme cluster** is what a _user perceives_ as a single character, even if it's made up of **multiple Unicode code points**.

| Visible Character | Unicode Points        | Bytes in UTF-8          | Explanation                      |
| ----------------- | --------------------- | ----------------------- | -------------------------------- |
| `a`               | `U+0061`              | 1 byte (`0x61`)         | Basic Latin letter               |
| `😊`              | `U+1F60A`             | 4 bytes (`f0 9f 98 8a`) | Emoji                            |
| `á`              | `U+0061 + U+0301`     | 1 byte + 2 bytes        | `a` + **combining accent** = `á` |
| `👨‍👩‍👧‍👦`     | `U+1F468 + ZWJ + ...` | Many codepoints         | Family emoji (joined by ZWJs)    |
> **Grapheme Cluster = "visible unit" (e.g. letter, emoji, flag, family, etc.)**

**1 grapheme cluster ≠ 1 Unicode point ≠ 1 byte**


This is what is needed for correct [[Rope]] implementation and correct cursor movement