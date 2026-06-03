# format_sync.py

Audit and synchronise fonts and formatting in a Word (`.docx`) document against a template, so the target document ends up formatted the same way as the template.

## What it does

Word formatting lives in two layers, and this script handles both:

1. **Styles** — named styles such as `Normal` and `Heading 1` define font family, size, bold, colour, and spacing. Most text inherits from these, so fixing the styles corrects the bulk of any document.
2. **Direct (inline) formatting** — overrides applied to individual runs of text, such as a manually bolded word or a one-off font swap. These sit on top of the styles and can mask them.

The script can report differences without changing anything, copy the template's style fonts into the matching styles in the target, and optionally strip direct overrides so the style formatting shows through.

The original target file is never modified in place — output always goes to a new file specified with `--out`.

## Requirements

- Python 3.9+
- `python-docx`

```
pip install python-docx
```

## Usage

### 1. Audit only (recommended first step)

Prints every style-level difference between template and target, plus a list of runs that carry direct overrides. Changes nothing.

```
python format_sync.py --template template.docx --target target.docx --report
```

### 2. Sync styles

Copies the template's font settings into the matching named styles in the target. This alone fixes most documents, because content inherits from styles.

```
python format_sync.py --template template.docx --target target.docx --out fixed.docx --sync-styles
```

### 3. Sync styles and clear direct overrides

Also removes run-level manual formatting so the style formatting takes effect. Use when manual overrides are fighting the styles.

```
python format_sync.py --template template.docx --target target.docx --out fixed.docx --sync-styles --clear-direct
```

## Options

| Flag | Description |
|------|-------------|
| `--template` | Reference `.docx` (required) |
| `--target` | The `.docx` to bring into line (required) |
| `--out` | Output path; required whenever changes are made |
| `--report` | Print formatting differences and exit, changing nothing |
| `--sync-styles` | Copy template style fonts into matching target styles |
| `--clear-direct` | Strip direct run-level font overrides in the target |

If no action flag is given, the script behaves like `--report`.

## What gets compared and copied

For each named style and each text run, the script inspects: font name, size, bold, italic, underline, and colour. Style spacing differences are surfaced in the report. When syncing, these font attributes are copied from the template style into the target style of the same name.

## Notes and limitations

- **Styles are matched by name.** A style that exists only in the template is reported but not created in the target, since generating a new style cleanly is document-specific. In practice, syncing the shared styles (`Normal`, `Heading 1`, and so on) covers almost everything because content inherits from them.
- **`--clear-direct` is a blunt instrument.** It removes *all* direct font overrides, including intentional ones such as an italicised book title. Run `--report` first, review the list of overrides, and only use `--clear-direct` if those overrides are unwanted. To keep some, fix them manually in Word after the style sync.
- **Coverage.** The script targets the font/style layer. Formatting carried elsewhere — tables, headers and footers, paragraph indentation, and section properties — is not synced by default.
- **Always verify.** Open the output in Word and confirm before relying on it.

## Example

```
# See what differs
python format_sync.py --template brand.docx --target draft.docx --report

# Bring the draft into line with the brand template
python format_sync.py --template brand.docx --target draft.docx --out draft_fixed.docx --sync-styles --clear-direct
```
