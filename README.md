# DWIN LCD CR-10s Pro source

This project aims to recreate missing project file and configuration options needed to rebuild the DWIN firmware for the CR-10s Pro printer.

<img align="top" width=175 src="images/dwin_lcd.jpg" />

Currently the DWIN LCD firmware project is based on [1.60.8 CR-10SPRO Screen firmware.rar](tools/1.60.8%20CR-10SPRO%20Screen%20firmware.rar) (part of latest 1.60.9 motherboard firmware), but those can be easiliy updated to the latest version.

__ATTENTION: ALFA VERSION! MIGHT NOT WORK WELL! USE ON YOUR OWN RISK!__

Currently, I have tested it on my own printer (version 1.60.3, see Pro_V1.60.3 branch) and it seems working well!

## Difference in master branch

- Currently, no modifications to the stock 1.60.8 Screen firmware

## Required steps to recreate the missing files

- Extract the official screen firmware to a temp directory
- ~Copy CONFIG.txt from [firmware/CR10sPro/DWIN_SET/CONFIG.txt](firmware/CR10sPro_1.60.8/DWIN_SET/CONFIG.txt) at the same directory.
    This file was also manually reacreated and might contain wrong parameters. More information about those configuration options you can read form "2.2 DGUS Parameter Configuration", tools/EN_DWIN DGUS Display Development Guide_V4.3.pdf~
    
    ~Another help might come from the demo projects provided by DWIN: [tools/Demo Projects T5LASIC_DGUSII_Part1.zip](tools/Demo%20Projects%20T5LASIC_DGUSII_Part1.zip) and [tools/Demo Projects T5LASIC_DGUSII_Part2.zip](tools/Demo%20Projects%20T5LASIC_DGUSII_Part2.zip)~
    
    Edited: This file seems to be replaced by T5UID1.CFG ("T5UID1.CFG is hardware configuration file for T5UID1 platform. T5UID1 is binary, it can be edit by UltraEdit.") - see [tools/T5UID1 Application Guide.pdf](tools/T5UID1%20Application%20Guide.pdf)
    
- Extract [tools/DGUS_V7383.rar](tools/DGUS_V7383.rar) (you need Windows machine) and run the DGUS Tool V7.383.exe
- Switch the language to English (menu on the top, right "Settings" in Chinese and then from the select menu pick English). Also from the Settings menu -> Set resolution to 480X272.
- Create a New project, select screen resoluton "480X272" (if it is not pre-selected), next pick a directory where you want to store the project. Save.
- Now, copy the entire DWIN_SET from the official screen firmware to the directory for the project you've just created (i.e. overwrite the "empty" DWIN_SET directory of the project)
- Also copy all of the DWIN_SET/*.ICO files to the ICON directory (Hint: this step is not verified and might not be mandatory). Now the temp DWIN_SET is no longer needed. You can remove it.
- From the DGUS GUI, from the Images view (left panel), click Add button (+) and select all of the files inside DWIN_SET. This will import all of the images to the new project
- Click on the Import button (File -> Import) and again select the DWIN_SET directory - that will import all of the Touch and Display variables (13*.bin and 14*.bin). Now all of the images have touch and/or display layers above.
- From Display menu -> Global Check - you should see the complete table and get no error messages.
- Now going from image (page) to image (page) you can see both touch and display fields. To select only Touch or Display variables, you can select that from the select menu on the right (top-right) panel, OR by using Control view (left panel). You can modify those fields by using the same properties panel on the right.
- Save, Generate, Export. File -> Close.
    You can export settings, project and generate the missing TFT directory. 
- Next time you open the "DGUS Tool V7.383.exe" you can directly select the project file: DWprj.hmi

## Missing or unknown artefacts

The following artefacts are of unknown origin to me (but are present in the original firmware):
- T5OS_V20_NOACK.BIN - "Common T5 OS platform core program" - might be an update sent by DWIN
- T5UID1.CFG - "5 Hardware Configuration File", see the pdf file. Not sure how this file was created in the first place.
- T5UID1_V22.BIN - "The GUI core program" - might be an update sent by DWIN
- 0_DWIN_ASC.HZK  (font file, normally should be added via DGUS tool). Currently just copy/paste from/to DWIN_SET

## Open questions

- Make sure the DWprj.hmi is correctly recreated (i.e. not missing something from the INIT section)
- Visually inspect that each page/image that is correctly linked to correct page/image (like Back button goes to correct page/image). That is especially importatant for the the SP (Stack pointer, default setting is 0xFFFF (set by Config. file)) and VP (Variable pointer, 0x0000-0x6FFF. Write 0x0000 for the variables that do not need address assigning. The command will be disabled when the high byte is 0xFF). See "4.4 VP & SP" section of the tools/EN_DGUS V5.10 User Guide.pdf
- Currently some(many) of the Return Key Codes, does not specify "Page switching" (i.e. page to switch is -1, but the page switching is NOT disabled). This might be correct behaviour if this sends codes via serial interface to the motherboard and then the MB to switch page "manually". That has to be verified. An example for this behaviour is page 61 (use Display manu -> Preview from current page to test). However pages like 34 works fine.

*Edit*: As suspected, looking at the Marlin source, "Return Key Code" is available through serial interface. For example RTS_HandleData() and the check "Addrbuf[i] == NzBdSet" sets "Checkkey = ManualSetTemp", which in return in "case ManualSetTemp:" can switch the page number like this: "RTS_SndData(ExchangePageBase + 16, ExchangepageAddr); //exchange to 16 page, the fans off"

Thus, with relative certainty it seems that the pages are imported correctly!

## Contribution

Any contribution to the project is welcome. From verifying the project and configuration files, filling-in the gaps or fixing bugs to testing in on spare DWIN LCD screen (DMT48270C043_06WT (T5UID1)). Once this project is confirmed to properly work on real hardware we can start using it to add extra features or at least fixing bugs.
