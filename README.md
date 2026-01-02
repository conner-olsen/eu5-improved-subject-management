# EU5 Improved Subject Management

A mod for Europa Universalis V that adds and modifies various subject interactions to significantly improve subject management.

[![Steam Workshop](https://img.shields.io/badge/Steam%20Workshop-Subscribe-blue)](https://steamcommunity.com/sharedfiles/filedetails/?id=3619130338)

## Overview

Managing subjects in EU5 can be frustrating due to numerous restrictions and missing features. This mod addresses these pain points by removing arbitrary limitations on land transfers, adding new management capabilities, and improving quality of life for players who want more control over their subject nations.

## Features

### 1. Transfer Land Between Subjects

Allows you to transfer land between any of your subjects, enabling you to create clean, logical borders throughout your realm.

### 2. Multiple Transfer Size Options

Choose the scale of land transfers with four different modes:
- **Location** - Transfer a single location
- **Province** - Transfer an entire province
- **Area** - Transfer all owned locations in an area
- **Region** - Transfer all owned locations in a region

### 3. Removed Location Restrictions on Land Transfers

In vanilla, you will commonly be unable to give a location just one province away to your subjects due to arbitrary restrictions. This mod removes those limitations, allowing transfers as long as both subjects exist.

### 4. Free Colonial Land Transfers

You can freely transfer land between colonial subjects and yourself. You gave them the land to begin with, and in most cases it's not better to own colonial land yourself, so this should have no effect on game balance.

### 5. War No Longer Fully Blocks Land Management

Why should being at war with a one-province minor halfway across the world prevent you from giving your newly colonized land to your subject? This mod implements smarter war restrictions:
- You can transfer land between subjects that are in the same war
- You can always transfer land from a colonial nation

### 6. Move Subject Capital

Sometimes subjects pick terrible capital locations, or you create a subject with a poor initial capital and want to fix it later. This interaction allows you to relocate your subject's capital to any of their owned locations.

### 7. Rename Regular Subjects

Enabled for all subject types except tributaries. Right-click a subject to access the rename option.

### 8. Subject Maps Spread to Overlord

Every year, any area discovered by a subject that borders an area discovered by its overlord will spread to the overlord. This steadily shares all subject maps with overlords without needing to manually share exploration.

## Technical Implementation

### Land Transfer System

- **`transfer_land_to_subject`** - Select a recipient first, then the giver, then the transfer mode and target
- **`transfer_land_from_subject`** - Select the giver first, then the recipient, then the transfer mode and target
- **`ism_perform_land_transfer`** - Scripted effect that handles the actual ownership changes based on selected mode

Transfer modes are implemented as invisible situations (`ism_transfer_mode_location`, `ism_transfer_mode_province`, etc.) that serve as selectable options in the interaction flow.

### War Restrictions

The modified war checks allow transfers when:
```
OR = {
    # Can always transfer from colonial nations
    subject_type = subject_type:colonial_nation
    # Cannot transfer land between nations not in the same war
    custom_tooltip = {
        text = "is_in_same_war"
        OR = {
            at_war = no
            AND = {
                at_war = yes
                scope:recipient = { at_war = yes }
            }
        }
    }
}
```

### Subject Renaming

The rename functionality modifies the country context menu (`ism_context_menu.gui`) to show a rename option for subjects. The visibility check ensures it only appears for:
- Countries with a valid top overlord
- Where the player is the top overlord
- Where the subject is not a tributary

The rename dialog bypasses the default C++ restrictions by using a console command:
```
onclick = "[ExecuteConsoleCommand( Concatenate( 'effect c:', Concatenate( RenameDialog.GetCountry.GetTag, Concatenate( ' = { change_country_name = \"', Concatenate( GetVariableSystem.Get('renaming_new_name'), '\" }' ) ) ) ) )]"
```

### Map Sharing

The `ism_share_maps` on_action runs yearly for overlords and iterates through all areas. For each undiscovered area that:
1. Is NOT discovered by the overlord
2. Borders an area the overlord HAS discovered
3. IS discovered by any of the overlord's subjects

The area is added to a discovery queue and revealed to the overlord at the end of the pulse.

## Planned Features

- Separate unrestricted location-transfer tool mod for MP use
- Always make you the war leader in any subject war (including rebellions and succession wars)
- Add ability to disallow a subject from colonizing
- Prevent your colonial subjects from declaring war on each other
- Prevent your tributaries from declaring war on your vassals/fiefdoms (but still other tributaries)
- Allow calling allies to aid in vassal wars (such as rebellions)
- Prevent colonial nations from shutting off settlements and plantations
- Prevent colonial nations from founding towns over valuable RGOs
- Make overlord always have food access in subjects (without having to request)
- Add ability to disallow a subject from declaring war
- Prevent subjects from immediately converting their culture back after enforcing your culture (likely a 10-year cooldown + disallowed if loyal)
- Allow disabling the Columbian Exchange for your subjects
- Make subjects' land count for formable requirements
- Make subjects ineligible to be in coalitions against you
- Allow managing subject urbanization

## Translations

- **Chinese**: Available in the [EU5 Mod Chinese Collection](https://steamcommunity.com/sharedfiles/filedetails/?id=3599706198). Thanks to *Niu Nai Da Mo Wang*.

## Compatibility

- **Not Ironman Compatible**
- **Fully save game compatible**
- Should work with any mod
- Compatibility patches available upon request

## Installation

### Steam Workshop (Recommended)
Subscribe on the [Steam Workshop](https://steamcommunity.com/sharedfiles/filedetails/?id=3619130338).

### Manual Installation
1. Clone or download this repository
2. Copy it to `Documents\Paradox Interactive\Europa Universalis V\mod`
3. Enable as normal in game

## Project Structure

```
eu5-improved-subject-management/
├── assets/
│   └── images/                 # Before/after screenshots and thumbnails
├── in_game/                    # Active gameplay files
│   ├── common/
│   │   ├── country_interactions/
│   │   │   ├── give_location_to_subject.txt    # Deletes vanilla interaction
│   │   │   ├── give_province_to_subject.txt    # Deletes vanilla interaction
│   │   │   ├── move_subject_capital.txt        # Capital relocation
│   │   │   ├── seize_location_from_subject.txt # Deletes vanilla interaction
│   │   │   ├── transfer_land_from_subject.txt  # Land transfer from subjects
│   │   │   └── transfer_land_to_subject.txt    # Land transfer to subjects
│   │   ├── on_action/
│   │   │   └── ism_share_maps.txt              # Yearly map sharing logic
│   │   ├── scripted_effects/
│   │   │   └── land_transfer_effects.txt       # Transfer execution logic
│   │   ├── scripted_triggers/
│   │   │   └── ism_can_declare_war.txt         # War declaration scaffolding
│   │   ├── situations/
│   │   │   └── ism_options.txt                 # Transfer mode options
│   │   └── subject_types/
│   │       └── ism_manage_war_declarations.txt # Subject type scaffolding
│   └── gui/
│       ├── ism_context_menu.gui                # Modified context menu
│       └── rename_dialog.gui                   # Subject rename dialog
├── main_menu/
│   └── localization/
│       └── english/
│           ├── ism_country_interaction_l_english.yml
│           ├── ism_custom_triggers_l_english.yml
│           └── ism_situations_l_english.yml
└── scripts/
    └── prepare-release.py                      # Release preparation script
```

## Contributing

Contributions are welcome! If you're interested in helping with development:
- Join the [EU5 Community Discord](https://discord.gg/EU5)
- Visit the `#improved-subject-management` channel
- Ping or add **@arealconner** on Discord

## Feature Requests

If you have any requests for features related to improving subject management, leave a comment in the [suggestions discussion](https://steamcommunity.com/workshop/filedetails/discussion/3619130338/685239622575385631/) on Steam.

## Links

- [Steam Workshop Page](https://steamcommunity.com/sharedfiles/filedetails/?id=3619130338)
- [Suggestions Discussion](https://steamcommunity.com/workshop/filedetails/discussion/3619130338/685239622575385631/)
- [EU5 Community Discord](https://discord.gg/EU5)

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.
