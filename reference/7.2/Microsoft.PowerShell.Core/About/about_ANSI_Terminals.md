---
description: Describes the features of PowerShell that use ANSI escape sequences and the terminal hosts that support them.
Locale: en-US
ms.date: 06/09/2022
schema: 2.0.0
title: about ANSI terminals
---
# about_ANSI_Terminals

## Support for ANSI escape sequences

PowerShell has many features that support the use of ANSI escape sequences to
control the rendering of output in the terminal application that's hosting
PowerShell.

PowerShell 7.2 added a new automatic variable, `$PSStyle`, and changes to the
PowerShell engine to support the output of ANSI-decorated text.

## Terminal support

The ANSI features are designed to be compatible with the xterm-based terminals.
For more information, see [xterm][1] in Wikipedia.

On Windows 10 and higher, the Windows Console Host is xterm compatible. The
[Windows Terminal][2] application is also xterm compatible.

On macOS, the default terminal application is xterm compatible.

For Linux, each distribution ships with a different terminal application.
Consult the documentation for your distribution to find a suitable terminal
application.

## $PSStyle

The variable has the following properties:

- **Reset** - Turns off all decorations
- **Blink** - Turns Blink on
- **BlinkOff** - Turns Blink off
- **Bold** - Turns Bold on
- **BoldOff** - Turns Bold off
- **Hidden** - Turns Hidden on
- **HiddenOff** - Turns Hidden off
- **Reverse** - Turns Reverse on
- **ReverseOff** - Turns Reverse off
- **Italic** - Turns Italic on
- **ItalicOff** - Turns Italic off
- **Underline** - Turns underlining on
- **UnderlineOff** - Turns underlining off
- **OutputRendering** - Control when output rendering is used
- **Background** - Nested object to control background coloring
- **Foreground** - Nested object to control foreground coloring
- **Formatting** - Nested object that controls default formatting for output
  streams
- **Progress** - Nested object that controls the rendering of progress bars
- **FileInfo** - (experimental) Nested object to control the coloring of
  **FileInfo** objects.

The base members return strings of ANSI escape sequences mapped to their names.
The values are settable to allow customization. For example, you could change
bold to underlined. The property names makes it easier for you to create
decorated strings using tab completion:

```powershell
"$($PSStyle.Background.BrightCyan)Power$($PSStyle.Underline)$($PSStyle.Bold)Shell$($PSStyle.Reset)"
```

The following members control how or when ANSI formatting is used:

- `$PSStyle.OutputRendering` is a
  `System.Management.Automation.OutputRendering` enum with the values:

  - `ANSI`: ANSI escape sequences are always passed through as-is.
  - `PlainText`: ANSI escape sequences are always stripped so that it's only
    plain text.
  - `Host`: This is the default behavior. The ANSI escape sequences are removed
    from redirected or piped output. For more information, see
    [Redirecting output][14].

- The `$PSStyle.Background` and `$PSStyle.Foreground` members are strings that
  contain the ANSI escape sequences for the 16 standard console colors.

  - **Black**
  - **BrightBlack**
  - **White**
  - **BrightWhite**
  - **Red**
  - **BrightRed**
  - **Magenta**
  - **BrightMagenta**
  - **Blue**
  - **BrightBlue**
  - **Cyan**
  - **BrightCyan**
  - **Green**
  - **BrightGreen**
  - **Yellow**
  - **BrightYellow**

  The values are settable and can contain any number of ANSI escape sequences.
  There is also a `FromRgb()` method to specify 24-bit color. There are two
  ways to call the `FromRgb()` method.

  ```csharp
  string FromRgb(byte red, byte green, byte blue)
  string FromRgb(int rgb)
  ```

  Either of the following examples set the background color the 24-bit color
  **Beige**.

  ```powershell
  $PSStyle.Background.FromRgb(245, 245, 220)
  $PSStyle.Background.FromRgb(0xf5f5dc)
  ```

- `$PSStyle.Formatting` is a nested object to control default formatting of
  debug, error, verbose, and warning messages. You can also control attributes
  like bolding and underlining. It replaces `$Host.PrivateData` as the way to
  manage colors for formatting rendering. `$Host.PrivateData` continues to
  exist for backwards compatibility but isn't connected to
  `$PSStyle.Formatting`. `$PSStyle.Formatting` has the following members:

  - **FormatAccent**
  - **TableHeader**
  - **ErrorAccent**
  - **Error**
  - **Warning**
  - **Verbose**
  - **Debug**

- `$PSStyle.Progress` allows you to control progress view bar rendering.

  - **Style** - An ANSI string setting the rendering style.
  - **MaxWidth** - Sets the max width of the view. Defaults to `120`.
    The minimum values is 18.
  - **View** - An enum with values, `Minimal` and `Classic`. `Classic` is the
    existing rendering with no changes. `Minimal` is a single line minimal
    rendering. `Minimal` is the default.
  - **UseOSCIndicator** - Defaults to `$false`. Set this to `$true` for
    terminals that support OSC indicators.

  > [!NOTE]
  > If the host doesn't support Virtual Terminal, `$PSStyle.Progress.View` is
  > automatically set to `Classic`.

  The following example sets the rendering style to a minimal progress bar.

  ```powershell
  $PSStyle.Progress.View = 'Minimal'
  ```

- `$PSStyle.FileInfo` is a nested object to control the coloring of
  **FileInfo** objects.

  - **Directory** - Built-in member to specify color for directories
  - **SymbolicLink** - Built-in member to specify color for symbolic links
  - **Executable** - Built-in member to specify color for executables.
  - **Extension** - Use this member to define colors for different file
    extensions. The **Extension** member pre-includes extensions for archive
    and PowerShell files.

  > [!NOTE]
  > `$PSStyle.FileInfo` is only available when the `PSAnsiRenderingFileInfo`
  > experimental feature ia enabled. For more information, see
  > [about_Experimental_Features][3] and [Using experimental features][4].

## Cmdlets that generate ANSI output

- The markdown cmdlets - the [Show-Markdown][5] cmdlet displays the contents of
  a file containing markdown text. The output is rendered using ANSI sequences
  to represent different styles. You can manage the definitions of the styles
  using the [Get-MarkdownOption][6] and [Set-MarkdownOption][7] cmdlets.
- PSReadLine cmdlets - the PSReadLine module uses ANSI sequences to colorize
  PowerShell syntax elements on the command line. The colors can be managed
  using [Get-PSReadLineOption][8] and [Set-PSReadLineOption][9].
- `Get-Error` - the [Get-Error][10] cmdlet returns a detailed view of an
  **Error** object, formatted to make it easier to read.
- `Select-String` - Beginning with PowerShell 7.0, [Select-String][11] uses
  ANSI sequences to highlight the matching patterns in the output.
- `Write-Progress` - ANSI output is managed using `$PSStyle.Progress`, as
  described above. For more information, see [Write-Progress][12]

## Redirecting output in `Host` mode

By default, `$PSStyle.OutputRendering` is a set to **Host**. The ANSI escape
sequences are removed from redirected or piped output.

**OutputRendering** only applies to rendering in the Host, `Out-File`, and
`Out-String`. Output from native executables isn't affected.

PowerShell 7.2.6 changed the behavior of `Out-File` and `Out-String` for the
following scenarios:

- When the input object is pure string, these cmdlets keep the string unchanged
  regardless of the **OutputRendering** setting.
- When the input object needs to have a formatting view applied to it, these
  cmdlets keep or remove escape sequences from the formatting output strings
  based on the **OutputRendering** setting.

This is a breaking change in these cmdlets compared to PowerShell 7.2.

**OutputRendering** doesn't apply to output from the PowerShell host process,
for example when you run `pwsh` from a command line and redirect the output.

In the following example, PowerShell is run on Linux from `bash`. The
`Get-ChildItem` cmdlet produces ANSI-decorated text. Since redirection occurs
in the `bash` process, outside of the PowerShell host, the output isn't
affected by **OutputRendering**.

```bash
$ pwsh -noprofile -command 'Get-Childitem' > out.txt
```

When you inspect the contents of `out.txt` you see the ANSI escape sequences.

By contrast, when redirection occurs within the PowerShell session,
**OutputRendering** affects the redirected output.

```bash
$ pwsh -noprofile -command 'Get-Childitem > out.txt'
```

When you inspect the contents of `out.txt` there are no ANSI escape sequences.

## Disabling ANSI output

Support for ANSI escape sequences can be turned off using the **TERM** or
**NO_COLOR** environment variables.

The following values of `$env:TERM` change the behavior as follows:

- `dumb` - sets `$Host.UI.SupportsVirtualTerminal = $false`
- `xterm-mono` - sets `$PSStyle.OutputRendering = PlainText`
- `xtermm` - sets `$PSStyle.OutputRendering = PlainText`

If `$env:NO_COLOR` exists, then `$PSStyle.OutputRendering` is set to
**PlainText**. For more information about the **NO_COLOR** environment
variable, see [https://no-color.org/][13].

## Using `$PSStyle` from C\#

C# developers can access `PSStyle` as a singleton, as shown in the following
example:

```csharp
string output = $"{PSStyle.Instance.Foreground.Red}{PSStyle.Instance.Bold}Hello{PSStyle.Instance.Reset}";
```

`PSStyle` exists in the System.Management.Automation namespace.

The PowerShell engine includes the following changes:

- The PowerShell formatting system is updated to respect
  `$PSStyle.OutputRendering`.
- The `StringDecorated` type is added to handle ANSI escaped strings.
- The `string IsDecorated` boolean property was added to return **true** when
  the string contains `ESC` or `C1 CSI` character sequences.
- The `Length` property of a string returns the length for the text without the
  ANSI escape sequences.
- The `StringDecorated Substring(int contentLength)` method returns a substring
  starting at index 0 up to the content length that'sn't a part of ANSI escape
  sequences. This is needed for table formatting to truncate strings and
  preserve ANSI escape sequences that don't take up printable character space.
- The `string ToString()` method stays the same and returns the plaintext
  version of the string.
- The `string ToString(bool Ansi)` method returns the raw ANSI embedded string
  if the `Ansi` parameter is **true**. Otherwise, a plaintext version with ANSI
  escape sequences removed is returned.
- The `FormatHyperlink(string text, uri link)` method returns a string
  containing ANSI escape sequences used to decorate hyperlinks. Some terminal
  hosts, like the [Windows Terminal][2], support this markup, which makes the
  rendered text clickable in the terminal.

<!-- link references -->
[1]: https://wikipedia.org/wiki/Xterm
[2]: https://www.microsoft.com/p/windows-terminal/9n0dx20hk701
[3]: about_Experimental_Features.md
[4]: /powershell/scripting/learn/experimental-features
[5]: xref:Microsoft.PowerShell.Utility.Show-Markdown
[6]: xref:Microsoft.PowerShell.Utility.Get-MarkdownOption
[7]: xref:Microsoft.PowerShell.Utility.Set-MarkdownOption
[8]: xref:PSReadLine.Get-PSReadLineOption
[9]: xref:PSReadLine.Set-PSReadLineOption
[10]: xref:Microsoft.PowerShell.Utility.Get-Error
[11]: xref:Microsoft.PowerShell.Utility.Select-String
[12]: xref:Microsoft.PowerShell.Utility.Write-Progress
[13]: https://no-color.org/
[14]: #redirecting-output-in-host-mode
