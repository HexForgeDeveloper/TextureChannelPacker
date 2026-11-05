# Texture Channel Packer tests

The separate `TextureChannelPackerTests` editor module contains automation tests
under `Source/TextureChannelPackerTests/Private/Tests`:

| Folder | Coverage |
| --- | --- |
| Unit | Pattern lists, path boundaries, preset capture and compatibility, exact UI option labels, reverse conversions and unknown-option defaults |
| Integration | All eight input and output source formats, preview precision, resize, invalid inputs, allocation limits, cancellation during read/write, cancellation rollback, Pack/Repack/Copy/Unpack asset operations, unique naming, blocked output directory, save rejection, Unpack rollback, read-only cleanup failures, unchanged input pixels, isolated ini persistence through the production save method and read-only preset rejection |
| UI | Real Slate layout and rendered preview pixels after the empty window has already been displayed, mapping/preset invalidation and preview regeneration |

The rendered regression first lets Slate arrange an empty preview. It then invokes
the same handler as the Preview button, waits for rendering, and checks both the
image geometry and screenshot pixels. Generating a valid texture alone is not a
passing UI result. A rendered evidence image is saved to
`Saved/Automation/TextureChannelPacker/RenderedPreview.png`.

## Run all tests

Install the plugin in a UE 5.5 project and build the project's Editor target.
Run the editor with a real rendering device:

```text
UnrealEditor-Cmd.exe "<Project>.uproject" -unattended -nosound -RenderOffscreen -ExecCmds="Automation RunTests TextureChannelPacker" -TestExit="Automation Test Queue Empty" -ReportExportPath="<AbsoluteReportDirectory>"
```

Build the project's Editor target first when installing from source. Inspect
`index.json` in the exported report: every case must finish with `Success`, including
`TextureChannelPacker.UI.RenderedPreview`.

Use an empty absolute directory for each exported report. Do not add `-NullRHI`:
it cannot verify Slate pixels. `-RenderOffscreen` runs the rendered UI tests
without opening interactive windows.

Asset tests use unique package folders, check saved source bytes and clean up only
their own generated assets. Settings tests use a separate temporary ini and do not
write the project's editor settings.

## Scope

These tests cover the primary processing and UI paths; they do not establish 100%
branch coverage. Save rejection is injected before disk I/O, and rollback deletion
failure uses a read-only test-owned file. Mid-write I/O faults, crash consistency,
every settings combination, interactive Content Browser selection and platform
behavior outside Windows/UE 5.5 still require additional coverage.

Unpack rollback is best-effort: if a generated file cannot be deleted, the operation
fails and reports the surviving output and its path. Previously existing assets are
not part of the rollback set.

Processing validates dimensions up to 16384 and caps each source, channel or output
buffer at 512 MiB before allocating it. This is a per-buffer cap, not a limit on
total process memory; Unreal source decompression and platform texture compilation
can consume additional memory. Cancellation is cooperative between inputs, in
decode blocks and output rows, and before saving. An individual package save is
not interrupted midway.
