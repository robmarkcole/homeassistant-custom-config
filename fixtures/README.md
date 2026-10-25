# RHS Wisley test fixture

`rhs-wisley-2026-10-25.html` is the unmodified HTML response returned by:

```text
https://www.rhs.org.uk/gardens/wisley
```

Capture date: 25 October 2026  
SHA-256: `7dcb410200a65c630d9ed269378a09eff17c4b5fd84f907df6b0a33f1239900f`

The response contains this positive test case inside `.alert-banner`:

```html
<p>Due to high winds, RHS Garden Wisley will be <strong>closed on Sunday 25th October</strong>. RHS Garden Centre Wisley will also be closed. We are very sorry for the inconvenience.</p>
```

With the template clock fixed at `2026-10-25`, the expected sensor state is `closed`. With a different date, the expected state is `no_closure_reported`.

To capture a new dated fixture, change the output filename and run:

```bash
curl -L --compressed --fail --silent --show-error \
  -A 'HomeAssistant test fixture' \
  -o fixtures/rhs-wisley-YYYY-MM-DD.html \
  https://www.rhs.org.uk/gardens/wisley
```

Then calculate and record its checksum:

```bash
shasum -a 256 fixtures/rhs-wisley-YYYY-MM-DD.html
```

The fixture is retained only to test parsing of the historical RHS response. The live RHS page remains the authoritative source for current visitor information.
