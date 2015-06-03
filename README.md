### Lyqyd's Project Builder - One-Stop Project Setup GUI for ComputerCraft Turtles

# 1 - Setup

## 1.1 - Manual Installation

To manually install this program, copy the `setup` file to a ComputerCraft turtle.  You will also need to copy the [Location API](https://github.com/lyqyd/location) and place it somewhere on the computer that the `setup` script can find, usually in the root of the drive.  Next, create a projects folder (either at `/etc/projects`, wherever you placed the `setup` file, or at `/projects`, in order of priority) to place the project files in.  Finally, add any project files you want the turtle to be able to construct.

## 1.2 - Automatic Installation

The recommended method of installation is to use [Packman](https://github.com/lyqyd/cc-packman) to install the `lyqyd/setup` package, as this will install the setup script and its dependencies.  Project files can then be added by installing them via packman, from the `projects` repository.

# 2 - Usage

Run `setup` with no arguments to start the program.

## 2.1 - Project List Screen

The Project List screen displays a list of all projects available on the turtle.  You can click on a project to select it on an advanced turtle, or use the arrow and enter keys to select a project on a normal turtle.  After selecting a project, pressing P or clicking on the title text on the top row of the screen will return to the Project List.

## 2.2 - Slots Screen

Press the S key or click on the `SLOTS` heading to view the Slots screen.

The slot contents requirements are displayed on this page. This program will only check that the required *quantities* of items are present.  It will not attempt to verify that the required *types* of items are present.  Requirements which are unmet are highlighted in black text on a white background on a normal turtle, or in red text on an advanced turtle.  Once requirements have been met, the text will change to white text on a black background for normal turtles, or to green text on an advanced turtle.

## 2.3 - Locations Screen

Press the L key or click on the `LOCATIONS` heading to view the Locations screen.

The location requirements are not present in all projects, so this screen will not always be available.  This screen is used primarily for projects that would be inconvenient to configure the turtle where the building begins, such as GPS clusters at the top of the world.  To edit the numeric parameters, use the Tab key to cycle between fields.  Use Enter to finish editing the numeric fields.  You can click on the X, Y, and Z fields in an advanced turtle directly to edit them.  Use 'h' to cycle through options for the Current Position's Facing parameter and 'H' to cycle through the Build Position's Facing parameter.  If the project provides preset location options, you can use the O key (or click on `(Options)`) to view them.  On a normal turtle, press the number key corresponding with the desired option, and you can also click on the desired option on an advanced turtle.

## 2.4 - Fuel And Miscellaneous Requirements Screen

Press the F key or click on the `FUEL/ETC` heading to view this screen.

This screen has two different sections.  On the left, the current fuel status can be seen.  The amount of fuel available is displayed, highlighted according to whether or not it is sufficient.  The fuel requirement for the project is also displayed.  You can use the R key or click the `(Refuel)` button to cause the turtle cycle through all of its slots not required for the current project and attempt to refuel using any items found there.  The turtle will refuel greedily during this procedure, attempting to consume every fuel item in these slots, even if it has enough fuel already.

On the right, any additional unmet requirements for the project are displayed.  Some projects must be built with a Crafty turtle, for instance, and that requirement will be displayed here.

## 2.5 - Other Usage

When all requirements are met, the `GO` button will appear at the bottom of the screen.  Click on this button or press the G key to begin the project.

At any time, press the Q key to exit the program.

# 3 - Project Files

Project files are a mixture between Lua scripts and APIs. They are loaded by executing them, and they must leave certain things in their environment table (like an API does) when they are finished.  The first thing is a human-readable name for the project file, stored as `name`.  The next is the `requirements` table.  Finally, each project file needs to provide an `instructions` function.

## 3.1 - Requirements Table

The requirements table can have up to four subsections; `slots`, `locations`, `fuel`, and `other`.  These are the prerequisites that must be met prior to the project being attempted.

### 3.1.1 - Slots Table

The slots table is a numerically-indexed table in the `slots` key of the `requirements` table.  Each of the numerically indexed tables represents a slot in the turtle and should contain five keys; `name,` `minCount`, `maxCount`, `exactCount`, and `readyFunction`.

The `name` key should contain a string used to display what the contents of the slot should be to the user.  For instance, you might use "Logs" for a slot that should contain wooden logs.  Names should be kept to a reasonable length, as they will need to be displayed in a two-column format if there are more than eight slots used.

The `minCount` and `maxCount` keys are used to specify a range of acceptable numbers of items to place in the slot.  These numbers are used to display the slot quantity requirement as well.  If the two numbers are not equal, they will be displayed as a range.

The `exactCount` key should contain a boolean, specifying whether the number of items in the slot must be an exact match to the `minCount` and `maxCount` requirements.  If this is false, quantities in excess of the value of `maxCount` will be considered as fulfilling the slot requirements, whereas if it is true, the excess items will need to be removed for the requirements to be satisfied.

The `readyFunction` key should be either nil or a function, and is used to customize the readiness behavior of the slot.  If a function is present here, it will be called to determine whether or not the slot is considered ready, and the `minCount` and `maxCount` fields will only be used for display purposes.

### 3.1.2 - Locations Table

The locations table is numerically-indexed, but also requires a `name` key to describe to users what the location being selected will be used for.  Each of the numeric indices should have a table containing a `name` key containing a string to display for the option as well as a `location` key containing a function that will return a location object.  The numerically-indexed location options are not required, and can be omitted--the mere presence of the `locations` table in the `requirements` table is enough to trigger the requirement of location information prior to the project being built.

### 3.1.3 - Fuel Value

The value of the `fuel` key can be either a number or a function that calculates and returns a number, which represents the minimum fuel level the turtle will require to complete the project.

### 3.1.4 - Other Requirements

If an `other` key is present in the `requirements` table, it must contain a function that either returns true (if all miscellaneous requirements are met), or returns false and a set of string arguments to be displayed to the user describing the unmet requirements.  These strings should be fairly short, as they must be displayed in a two-column layout and will be truncated to fit.

## 3.2 - Instructions Function

The instructions function is the meat and bones of a project file, as all of the work must be done inside it. The simplest way to set up a project file is to copy an existing program inside the instructions function and add a requirements table detailing the prerequisite conditions.  Each project's instruction function will be run with no arguments passed to it, but with an environment that includes the project's requirements table (at `requirements`), the current shell, and, if the project had location requirements, the initial location and build location values injected at `requirements.locations.start` and `requirements.locations.setup`, respectively.  The environment will also have access to the APIs found in _G.  This function is run when the user clicks the `GO` button to start the program.