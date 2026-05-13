# DOSZ and DOSC Files

## DOSZ Files
A DOSZ file is a ZIP file which contains one DOS game that can be loaded in a DOS emulator similar to how a console game ROM from that era is used with console emulators.

## DOSC Files
A DOSC file is also a ZIP file but it contains config files created from a setup program separate from the games installation
as well as binary patches (in IPS/BPS/VCDIFF/[XOR](../../../MakeXORPatch) format) that remove copy protection schemes.  
DOSC files can also include one or more [launch configurations](#dosc-launch-configurations) using the features described below.

An existing DOSC file is always loaded as a file system overlay for a DOSZ file with a matching name.

## DOS.YML
A DOS.YML file describes hardware and software configuration for the DOS emulator to make running a game correctly as seamless as possible.

The format description can be found in the [DOS.YML document](DOS.YML.md).

Such a file can be bundled as part of the DOSZ or DOSC file of a game. If the game inherently does not run on a DOS emulator's
default compatibility mode it should be part of the core DOSZ file. Otherwise, if the game's configurable settings stored in
the DOSC file warrants special emulation settings, it would go in there. If the file exists in both archives, an emulator will
load the one in DOSZ first followed by the one in DOSC, which can then add or overwrite keys.

## Parent/Child DOSZ Files
A DOSZ file can set itself to be a child of another DOSZ file by including an empty file with a .dosz.parent extension.  
For example, if a file named 'game.dosz.parent' is placed in a DOSZ, it will be loaded as an overlay over game.dosz.  
There can only be one .dosz.parent file but a parent can be a child of another parent (as long as there is no circular dependency).

The DOSC for a child DOSZ can override files contained in a parent (DOSZ or DOSC).
But a binary patch can only apply to a file that exists in the DOSZ with the same name as the DOSC.

## Shared Saves
If a DOSZ file contains an empty file with a .savename extension, modifications to the C: drive will
be stored in a save file named after that file instead of the DOSZ file itself.

For example, if parent and child DOSZ files each contain a .savename file with the same name,
save data like configurations, options or game saves will be shared between them.

Another example is for a game series in which a sequel can import save data from a predecessor. In such a case the
DOSZ files of each entry of the series should be installed in a directory not directly in the root of the C: drive.

## DOSC Launch Configurations
A DOSC file can specify more than one launch configuration by including a directory enclosed in [ ] brackets like `[My Config]`.  
A supported emulator will then on startup ask the user which launch configuration (or the default) should be used.

A launch configuration can do anything the root of the DOSC file can (file overrides, file patches and a DOS.YML file). As a minimum
it should contain a DOS.YML file with an auto start setting (either a `run_path` or `run_boot` key).

### Launch Config Categories
If a launch configuration name contains one or more `+` signs, the user can be prompted via a multiple choice menu.  
For example, if there are 3 directories `[Adlib + German]`, `[MIDI + English] and `[MIDI + German]`, the user will be prompted with
two categories where one has options "Adlib" and "MIDI" and the other "English" and "German". The emulator will automatically use
the default launch configuration (as defined in the root of the DOSC file) for the one combination that does not exist as an explicit
directory (in the example that would be "Adlib" and "English").

Additionally, a DOSC can contain a combination of categories and regular launch configurations which could be additional tools for a game.

### Launch Config Display Order
By default the launch configurations will be offered in alphabetical order. When using categories, the default combination will always show
at the top of the multiple choice menu of each category. If required, a custom order can be defined by prefixing an item with `n#` or `nn#`
where n/nn is a number. For example `[3#Setup]` or `[2#Adlib + 3#CGA]`. Such a prefix will then be hidden in the user interface.

## Loading Order
The number of file system overlays can become quite complex when using all the features above.  
Generally the order is parent-dosz → parent-dosc → child1-dosz → child1-dosc → child2-dosz → child2-dosc ... and finally the launch config directory (or directories).

Here's an example with multiple layers of parent DOSZ files and launch configurations in each layer where there is a main game package with
an expansion that depends on it and a second expansion that depends on the first one. On top of that each defines a [MIDI] launch config.
This is the order how all the packages will be overlaid in the file system and how DOS.YML files will be loaded:

- `Game Main.dosz`
- `Game Main.dosc`
- `Game Expansion 1.dosz` (contains `Game Main.dosz.parent`)
- `Game Expansion 1.dosc`
- `Game Expansion 2.dosz` (contains `Game Expansion 1.dosz`)
- `Game Expansion 2.dosc`
- `Game Main.dosc/[MIDI]`
- `Game Expansion 1.dosc/[MIDI]`
- `Game Expansion 2.dosc/[MIDI]`
