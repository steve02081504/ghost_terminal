- [JP](./README_JP.md)  
- [CN](./README_CN.md)  
- [EN](./README_EN.md)  

# ghost_terminal  

![preview image](./preview.png)  
Original trial product, now finding it somewhat useful (  

## Usage  

Use ghost_terminal like a system terminal  
up/down toggle command, right mouse button for quick paste, supports tab completion (if supported by ghost)  
Type in the expressions supported by your ghost and then evaluate them!  
Easy to use for ghost development  

## Command line arguments  

```text
ghost_terminal [options]
options:
  -h, --help                            : shows this help message.
  -c, --command <command>               : runs the specified command and exits.
  -s, --sakura-script <script>          : runs the specified Sakura script and exits.
  -g, --ghost <ghost>                   : links to the specified ghost by name.
  -gh, --ghost-hwnd <hwnd>              : links to the specified ghost by HWND.
  -gp, --ghost-folder-path <path>       : links to the specified ghost by folder path.
  -r, --run-ghost                       : runs the ghost if (it/she/he/them/other pronouns) is not currently running.
  -rwt, --register-to-windows-terminal  : registers to the Windows terminal (requires -g <ghost name> or -gp <ghost folder path>).
        -rwt-name <name>                : registers to the Windows terminal with the specified name (only works with -rwt).
        -rwt-icon <icon>                : registers to the Windows terminal with the specified icon (PNG or ICO path) (only works with -rwt).
```

For example:  

```bat
//...
..\saori\ghost_terminal.exe -g Taromati2 -c reload
@echo on
```

ghost_name can be the name of the Sakura (`\0`) side, or the `GhostName` returned by `ShioriEcho.GetName`, or the ghost name in descript.txt  

## Event list  

ghost_terminal communicates information with ghost via `X-SSTP-PassThru-*` (see [documentation]( http://ssp.shillest.net/ukadoc/manual/spec_shiori3.html ))  

### Virtual terminal sequence  

Any ghost_terminal output will be rendered by a virtual terminal sequence, rather than plain text.  
Reference: [Console Virtual Terminal Sequences](https://learn.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences)  
You can use it to control the display of the terminal, such as text colour, background colour, font, etc.  

### Start/End

- `ShioriEcho.Begin`  
  ghost_terminal event when this ghost starts  
  - `Reference0`  
    Terminal version  
  - Return value  
    - `X-SSTP-PassThru-Tittle` (optional)  
      Set the terminal title  
- `ShioriEcho.End`  
  ghost_terminal event on normal program exit  
  - return value  
    Ignored, **but speech spirits execute normally**  
- `ShioriEcho.GetName`  
  Event when ghost_terminal gets a name  
  - Return value  
    - `X-SSTP-PassThru-GhostName` (optional)  
      Show ghost name  
    - `X-SSTP-PassThru-UserName` (optional)  
      Show username  

### Execute the command  

- `ShioriEcho`  
  Event after the command has been typed  
  - `Reference0`  
    Commands collected by the terminal  
  - Return value  
    Ignored, **but sakura scripts are executed normally**  
- `ShioriEcho.GetResult`  
  Query for value result event  
  - Possible return value 1  
    - `X-SSTP-PassThru-Result`  
      Display the content and go to the next command to fetch  
    - `X-SSTP-PassThru-Type` (optional)  
      Additional information: value type  
  - Possible return value 2  
    - `X-SSTP-PassThru-Special`  
      Show content and go to the next command fetch  
      This return value will be simply escaped:  
      - `\n` will be converted to a newline  
      - `\t` will be converted to tabs  
      - `\\` will be converted to `\`  
  - Possible return value 3  
    - Empty  
      Wait 1 second and re-initiate `ShioriEcho.GetResult`  
  - Other
    - `X-SSTP-PassThru-State`  
      This return value can be superimposed on the above return value  
      If it is `End`, the terminal terminates  

### Other  

- `ShioriEcho.TabPress`  
  command completes the event by tab  
  - `Reference0`  
    Commands collected by the terminal  
  - `Reference1`  
    The first character of the command where the cursor was when the tab was pressed (starting value 0)  
  - `Reference2`  
    The first consecutive times the user pressed tab (starting value 0)  
  - Return value  
    - `X-SSTP-PassThru-Command` (optional)  
      Replace the command with this content  
    - `X-SSTP-PassThru-InsertIndex` (optional)  
      Move the cursor to this position (or leave it unchanged if not provided)  
- `ShioriEcho.CommandUpdate`  
  Event when command is updated  
  - `Reference0`  
    Commands collected by the terminal  
  - Return value  
    - `X-SSTP-PassThru-CommandForDisplay` (optional)  
      Replace the displayed command with this content  
      ghost can use this event to implement masks on password entry, syntax highlighting on normal reads, and other functions  

### Command History  

- `ShioriEcho.CommandHistory.New`  
  Event when an empty command is added at the end of the command history  
  - Return value  
    Ignored, **but sakura scripts are executed normally**
- `ShioriEcho.CommandHistory.Update`  
   Event when command history is updated  
  - `Reference0`  
    The contents of the history command  
  - `Reference1`  
    Index of the history command (in reverse order, starting value 0)  
  - Return value  
    Ignored, **but sakura scripts are executed normally**
- `ShioriEcho.CommandHistory.Get`  
  Event when command history is fetched  
  - `Reference0`  
    Index of the history command (in reverse order, starting value 0)
  - Return value
    - `X-SSTP-PassThru-Command` (optional)  
      Replace the command with this content  
