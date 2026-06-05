# Contributing to INTENT-7

INTENT-7 (Common Intent Specification) is the soul layer of the protocol family.

## Specification Files

- English: [spec/INTENT-7.md](spec/INTENT-7.md)
- Chinese: [spec/INTENT-7.zh-CN.md](spec/INTENT-7.zh-CN.md)

## How to Propose Changes

1. Read the [organization-level contribution guide](https://github.com/CommonIntents/.github/blob/main/CONTRIBUTING.md)
2. Open an Issue describing the problem or improvement you've identified
3. If proposing a new `x-` extension field, follow the extensibility rules in §12
4. Update both the English and Chinese versions in your PR

## Extension Field Naming Rules

- All custom fields MUST use the `x-` prefix
- Use the `x-orgname-fieldname` format to avoid conflicts
  Example: `x-cellrix-priority`

## Conformance Testing

The reference implementation Cellrix provides a conformance test suite.
If you discover edge cases not covered by existing tests, new test cases are welcome.
