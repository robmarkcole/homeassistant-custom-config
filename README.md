# Home Assistant custom configuration

Small, self-contained Home Assistant configuration examples.

## RHS Wisley closure sensor

This sensor checks the [RHS Garden Wisley page](https://www.rhs.org.uk/gardens/wisley) every 15 minutes and reports whether the page contains a closure notice for the current date.

It uses Home Assistant's built-in [Scrape integration](https://www.home-assistant.io/integrations/scrape/). The RHS notice is included in the server-rendered HTML, so this does not require a custom Python script, browser automation, Playwright, or HACS.

### Configuration

Add the following to `configuration.yaml`:

```yaml
scrape:
  - resource: "https://www.rhs.org.uk/gardens/wisley"
    scan_interval: 900
    sensor:
      - name: "RHS Wisley Closure"
        unique_id: rhs_wisley_closure
        select: "body"
        value_template: >-
          {% set text = value | lower %}
          {% set month = now().strftime('%B') | lower %}
          {% set day = now().day %}
          {% set today_ordinal = (day | ordinal) ~ ' ' ~ month %}
          {% set today_plain = (day | string) ~ ' ' ~ month %}
          {% if 'rhs garden wisley will be closed' in text
                and (today_ordinal in text or today_plain in text) %}
            closed
          {% else %}
            no_closure_reported
          {% endif %}
```

The sensor has three meaningful outcomes:

| State | Meaning |
| --- | --- |
| `closed` | Today's date and the expected Wisley closure wording were found. |
| `no_closure_reported` | The page was fetched, but the expected closure notice for today was not found. |
| `unavailable` | Home Assistant could not fetch or parse the page. |

Selecting `body` is deliberate. Selecting `.alert-banner` directly would make the entity unavailable whenever the banner is absent, making an ordinary no-closure result indistinguishable from a fetch or parsing problem.

The date check accepts both ordinal and plain UK-style dates, such as `25th October` and `25 October`. On 25 October 2026, the live notice used the ordinal form; a plain `strftime('%-d %B')` comparison would therefore have missed it.

### Installation and validation

1. Add the configuration to `configuration.yaml`.
2. In Home Assistant, open **Developer Tools → YAML** and run **Check configuration**.
3. Restart Home Assistant.
4. Open **Developer Tools → States** and find `sensor.rhs_wisley_closure`.
5. Confirm that the state agrees with the notice on the RHS page.

Do not treat `no_closure_reported` as proof that the garden is open. It only means that the expected wording and today's date were not found. For decisions about travel or safety, check the RHS page directly.

### Test fixture

`fixtures/rhs-wisley-2026-10-25.html` is the raw server response captured on 25 October 2026. It contains the high-winds closure notice and can be used as a stable positive test case if the live page changes.

The template depends on `now()`, so a later automated test must freeze Home Assistant's clock at `2026-10-25` or substitute that date in the test harness. Testing the dated fixture against the real current clock will intentionally produce `no_closure_reported`.

See `fixtures/README.md` for provenance, checksum, expected result, and the refresh command.

### Limitations

- The sensor reports a closure only on the date named in the notice; it does not turn on early for future closures.
- Different wording, such as "Wisley is closed", will require the phrase match to be updated.
- Multi-day notices and date ranges are not currently parsed.
- Any substantial redesign of the RHS page may require revisiting the selector or template.

### References

- [RHS Garden Wisley](https://www.rhs.org.uk/gardens/wisley)
- [Home Assistant Scrape integration](https://www.home-assistant.io/integrations/scrape/)
- [Home Assistant template functions](https://www.home-assistant.io/template-functions/)
