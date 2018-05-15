Autoit-Gui-Skeleton (AGS)
=========================

> Proposes to provide an architecture and an organization to efficiently build an desktop application Windows with AutoIt.

- [Autoit-Gui-Skeleton (AGS)](#autoit-gui-skeleton--ags-)
  * [Architecture](#architecture)
    + [Directory `assets`](#directory--assets-)
    + [Directory `deployment`](#directory--deployment-)
    + [Directory `vendor`](#directory--vendor-)
    + [Directory `views`](#directory--views-)
    + [AGS overview](#ags-overview)
  * [Code Organization](#code-organization)
    + [Main entry program](#main-entry-program)
    + [Centralize the declaration of global variables](#centralize-the-declaration-of-global-variables)
    + [Write the business/logic code in dedicated files](#write-the-business-logic-code-in-dedicated-files)
    + [Main program to manage the GUI](#main-program-to-manage-the-gui)
    + [Declare code for all views in dedicated files](#declare-code-for-all-views-in-dedicated-files)
    + [Main events manager](#main-events-manager)
    + [Switch view](#switch-view)
  * [About](#about)
    + [Release history](#release-history)
    + [Contributing](#contributing)
    + [License](#license)


<br/>

## Architecture

In order to organize the code of an AutoIT application with a graphical interface, we propose to use the following model.

```
project root folder
|
|   myApplication.au3          # Main entry program
|   myApplication_GLOBAL.au3   # All global variables declaration
|   myApplication_GUI.au3      # Main program to handle GUI
|   myApplication_LOGIC.au3    # Business code only
|   README.md                  # Cause We always need it
|  
+---assets                     # All applications assets (images, files...)
|   +---css
|   +---html
|   +---images
|   \---javascript
|
+---deployment                
|   \---releases               # Contains releases setup (zip and Windows setup files)
|   deployment.bat             # Windows batch bandmaster to pilot the creation of the Windows setup
|   deploymeny.iss             # ISS to generate Windows setup
|
+---vendor                     # All third-party code use in this project
|   \--- FolderVendor
|              
\---views                      # Views declaration
    View_About.au3
    View_Footer.au3
    View_Welcome.au3
```


### Directory `assets`

This directory contains the elements used in the application like images, text files, pdf, html, css, javascript. Indeed, note that it is possible to integrate in a AutoIt application, a static html file, into a web browser embedded in the HMI with the `_IECreateEmbedded ()` method provided by the `IE.au3` library.


### Directory `deployment`

This directory contains a Windows batch that controls the creation of a Windows installer with the [InnoSetup](http://www.jrsoftware.org/isinfo.php) solution. To run the batch, it is necessary that the InnoSetup compiler and 7zip be installed on the pc. If this is not the case, I advise you to use the Windows package manager [Chocolatey](https://chocolatey.org/) to install them simply:

```
C: \> choco install 7zip
C: \> choco install innosetup
```


### Directory `vendor`

This directory is the place where to conventionally store the code developed by third parties in a project. In this project (https://github.com/v20100v/autoit-gui-skeleton), we have for example put the GUICtrlOnHover v2.0 library created by G.Sandler a.k.a MrCreatoR in this directory.


### Directory `views`

This directory contains view managers. All the code of all the views are defined each time in a specific file and stored in that directory.


### AGS overview

![](docs/autoit-gui-skeleton_overview.png)


<br/>

## Code Organization

> Explain the organization of the code of a project respecting the AGS conventions. We describe below its main elements.

### Main entry program

The `myApplication.au3` program serves as a single point of entry for our application. This is where the application starts. In the latter we start by including all the other dependencies that it needs: libraries of AutoIt, third-party libraries, the declaration of global variables, the code of the application GUI and business. It calls a single method: `_main_GUI ()`. It is the main GUI manager that is used to build the interface and manage user interactions.

```AutoIt
;; myApplication.au3 ;;

Opt('MustDeclareVars', 1)

; Include all built-in AutoIt library requires
#include <IE.au3>
#include <GUIConstantsEx.au3>
#include <WinAPIDlg.au3>

; Include all third-party code in a project store in vendor directory
#include 'vendor/GUICtrlOnHover/GUICtrlOnHover.au3'

; Include myApplication scripts
#include 'myApplication_GLOBAL.au3'
#include 'myApplication_GUI.au3'
#include 'myApplication_LOGIC.au3'

; Start main graphic user interface defined in myApplication_GUI.au3
_main_GUI()
```


### Centralize the declaration of global variables

This script is used to define all the constants and variables of the application in the overall scope of the program, with the exception of graphic elements, which are defined in a specific view file. Moreover by convention, all the variables declared in `myApplication_GLOBALS.au3` must be written in capital letters and separated by underscores.

The `Global` statement is used to explicitly indicate which access to the scope is desired for a variable. If you declare a variable with the same name as a parameter, using Local in a user function, an error will occur. Global can be used to assign global variables within a function, but if a local variable (or parameter) has the same name as a global variable, the local variable will be the only one used. It is recommended that local and global variables have distinct names.

The statement `Global Const` is used to declare a constant. Once created a global constant, you can not change the value of a constant. In addition, you can not replace an existing variable with a constant.

```
;; myApplication_GLOBAL.au3 ;;

; Application main constants
Global Const $APP_NAME = "myApplication"
Global Const $APP_VERSION = "1.0.0"
Global Const $APP_WEBSITE = "https://myApplication-website.org"
Global Const $APP_EMAIL_CONTACT = "myApplication@website.org"
Global Const $APP_ID = "acme.myApplication"
Global Const $APP_LIFE_PERIOD = "2016-"&@YEAR
Global Const $APP_COPYRIGHT = "© "&$APP_LIFE_PERIOD&", A.C.M.E."

; Application GUI constants
Global Const $APP_WIDTH = 800
Global Const $APP_HEIGHT = 600
Global Const $APP_GUI_TITLE_COLOR = 0x85C4ED
Global Const $APP_GUI_LINK_COLOR = 0x5487FB

; Application global variable
; Example in order to persist an opened file on action "File > Open"
Global $OPEN_FILE = False
Global $OPEN_FILE_PATH = -1
Global $OPEN_FILE_NAME = -1
```


### Write the business/logic code in dedicated files

In this model, we add a `myApplication_LOGIC.au3` file that contains all the methods of the business code. These logical methods are then called by others usually triggered by user interactions, whose links are defined in the GUI. Simple principle of separation between logic and view.

For example, we manage a dialog box as follows:

```AutoIt
;; myApplication_LOGIC.au3 ;;

;===========================================================================================================
; Show a dialog box to user, in order that he chooses a file in the Windows Explorer
;
; @params : void
; @return : $array_result[0] = file path
;           $array_result[1] = file name
;===========================================================================================================
Func _dialogbox_open()
   Local $info_fichier = _WinAPI_GetOpenFileName("Open file", "*.*", @WorkingDir, "", _
                          "", 2, BitOR($OFN_ALLOWMULTISELECT, $OFN_EXPLORER), _
              $OFN_EX_NOPLACESBAR)
   Local $array_result[2]

   If @error Then
      $array_result[0] = -1
      $array_result[1] = -1
   Else
      ; Chemin relatif du nouveau fichier
      $array_result[0] = $info_fichier[1]&"\"&$info_fichier[2] ; $PATHFILE_OF_OPEN_FILE_IN_APP
      $array_result[1] = $info_fichier[2] ; $NAME_OF_OPEN_FILE_IN_APP
   EndIf

   Return $array_result
EndFunc
```


### Main program to manage the GUI

The `myApplication_GUI.au3` file contains the` _main_GUI () `method which is called by the main input program. This method is designed to create the graphical user interface (GUI) and manage all user interactions and events. So it calls all the other methods that initialize the elements of the GUI.

```AutoIt
;; myApplication_GUI.au3 ;;

#include-once

; Includes all views definition
#include './views/View_Footer.au3'
#include './views/View_Welcome.au3'
#include './views/View_About.au3'

;==============================================================================================================
; Main graphic user interface
;
; @param void
; @return void
;==============================================================================================================
Func _main_GUI()
   Global $main_GUI = GUICreate($APP_NAME, $APP_WIDTH, $APP_HEIGHT, -1, -1)

   _GUI_Init_Menu()

   _GUI_Init_Footer()       ; By default all elements of this view are visible
   _GUI_Init_View_Welcome() ; By default all elements of this view are hidden
   _GUI_Init_View_About()   ; By default all elements of this view are hidden

   ; Set configuration application : icon, background color
   _GUI_Configuration()

   ; Show Welcome view on startup
   _GUI_ShowHide_View_Welcome($GUI_SHOW)
   GUISetState(@SW_SHOW)

   ; Handle all user interactions and events
   _GUI_HandleEvents()

   GUIDelete()
   Exit
EndFunc

(...)
```

Some comments:

 - All uppercase variables (`$ APP_NAME`,` $ APP_WIDTH`, `$ APP_HEIGHT`) are declared in the global scope of the application. Their definition is done in the file `myApplication_GLOBAL.au3`;
 - `_GUI_Init_Menu ()` is used to create a menu control in the main GUI;
 - `_GUI_Init_Footer ()` is used to create footer elements in the main GUI. Its definition is made in a separate special file. All footer elements are visible in all views by default, so we do not need to manage its visibility.
 - `_GUI_Init_View_Welcome ()` is used to create GUI elements for a "Welcome" view name. All items declared in this method are hidden by default. To display the "Welcome" view, that is, to make it visible, simply call the method with this parameter `_GUI_ShowHide_View_Welcome ($ GUI_SHOW)`. And to hide them, just call `_GUI_ShowHide_View_Welcome ($ GUI_HIDE)`;
 - `_GUI_HandleEvents ()` handles all user interactions and events by parsing the return message with the `GUIGetMsg ()` method. The event return with the GUIGetMsg method is the control ID of the control that sends the message. This method calls another specific handler event per view, for example `_GUI_HandleEvents_View_Welcome ($ msg)`;


### Declare code for all views in dedicated files

For example, for managing the creation of the graphic elements of the "Welcome" view, we use the `_GUI_Init_View_Welcome ()` method.

```AutoIt
;; ./view/View_Welcome.au3 ;;

Func _GUI_Init_View_Welcome()
   ; Create GUI elements here for "Welcome view" in global scope
   Global $label_title_View_Welcome = GUICtrlCreateLabel("Welcome", 20, 30, 400)
EndFunc
```

For the management of the display of the elements of the "Welcome" view, we use the `_GUI_ShowHide_View_Welcome ($ action)` method

```AutoIt
;; ./view/View_Welcome.au3 ;;

Func _GUI_ShowHide_View_Welcome($action)
   Switch $action
      Case $GUI_SHOW
         ; Define here all elements to show when user come into this view
         _GUI_Hide_all_view() ; Hide all elements defined in all method _GUI_ShowHide_View_xxx
         GUICtrlSetState($label_title_View_Welcome, $GUI_SHOW)
         GUICtrlSetState($label_welcome, $GUI_SHOW)

      Case $GUI_HIDE
         ; Define here all elements to hide when user leave this view
         GUICtrlSetState($label_title_View_Welcome, $GUI_HIDE)
         GUICtrlSetState($label_welcome, $GUI_HIDE)
    EndSwitch
EndFunc
```

For event handling in the "Welcome" view, use the `_GUI_HandleEvents_View_Welcome ($ msg)` method. This method is called in the `_GUI_HandleEvents ()` main handler method.

```AutoIt
;; ./view/View_Welcome.au3 ;;

Func _GUI_HandleEvents_View_Welcome($msg)
   Switch $msg

      ; Trigger for click on $image_banner
      Case $label_welcome
         ConsoleWrite('Click on "$label_welcome"' & @CRLF)

      ; Add another trigger in view 'Welcome' here
   EndSwitch
EndFunc
```


### Main events manager

The main user and application event handler is named `_GUI_HandleEvents()`. It is the latter who will call all the other event managers specific to each view. They are named by convention `_GUI_HandleEvents_View_Xxx($msg)`.

```AutoIt
;; myApplication_GUI.au3 ;;

Func _GUI_HandleEvents()
   Local $msg
   While 1
    ; event return with GUIGetMsg method, i.e. the control ID of the control sending the message
    $msg = GUIGetMsg()
      Switch $msg
         ; Trigger on close dialog box
         Case $GUI_EVENT_CLOSE
            ExitLoop

         ; Trigger on click on item menu 'File > Exit'
         Case $menuitem_Exit
            ExitLoop
      EndSwitch

      _GUI_HandleEvents_View_Welcome($msg)
      _GUI_HandleEvents_View_About($msg)
      _GUI_HandleEvents_Menu_File($msg)
      _GUI_HandleEvents_Menu_About($msg)
   WEnd
EndFunc
```

### Switch view

To switch from a start view to another arrival view, you first have to hide all the graphic elements, then in a second time to display only those of the arrival view. So, how to hide all the graphic elements? Just with a `_GUI_Hide_all_view()` method that will call the view manager of each view. These are conventionally named `_GUI_ShowHide_View_Xxx`.

```AutoIt
;; myApplication_GUI.au3 ;;

Func _GUI_Hide_all_view()
   _GUI_ShowHide_View_Welcome($GUI_HIDE)
   _GUI_ShowHide_View_About($GUI_HIDE)
EndFunc
```


<br/>

## About


### Release history

 - AGS v1.0.0 - 2018.05.15


### Contributing

Comments, pull-request & stars are always welcome !


### License

Copyright (c) 2018 by [v20100v](https://github.com/v20100v). Released under the [Apache license](https://github.com/v20100v/autoit-gui-skeleton/blob/master/LICENSE.md).

![r2d2](docs/r2d2.gif)
