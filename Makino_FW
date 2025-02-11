MaxMacros	40
Macro	0

# =============================================================
# This script automatically creates low magnification maps of cryo-grids
# on JEOL CryoARM microscopes.
# ----------------------------------------------------------------------------------------------------------
# Written by Bart Marzec [bart@jeoluk.com] / last update: 01 Nov 2021, modified by FM 12 Apr 2022
# ----------------------------------------------------------------------------------------------------------
# The script requires GridSwapper to work correctly.
# =============================================================

ScriptName Batch_Atlas


# ============================
# Parameters
# ============================
exp_time = 1 # [sec]
spot_atlas = 7 # Spot Size for Atlas
removeCLapt = 0 # 0:No defualt for 3300,  1:Yes
CLapt_sizeID = 1

# Parent directory path - where the files will be written
PARENT_DIRECTORY = X:\makinotest

# Position 0 - on the column
# Positions 1-12 -> storage
# Montage_All set to 1 to auto-determine sample IDs and montage all of them
# or set to 0 and fill in the Positions_to_montage matrix with at least 2 numbers.
POSITIONS_TO_MONTAGE = { 477 474 } #input IDs keep space after number. e.g. {### }

# This should be normally in C:\JEOL\GridSwapper
GRIDSWAPPER_PATH = C:\ProgramData\SerialEM\Tool\

# Montage magnification should be the actual low mag value
MONTAGE_MAGNIFICATION = 50


# Please adjust parameters listed in capital letters as required.

#For DE
# 0: Do not use DE
# 1: DE64
# 2: DE64 with old DE Server (<2.1)
# 3: Apollo
useDE = 0

NewArray IDs 0 1
IDs = $POSITIONS_TO_MONTAGE

echo Sample IDs that will be montaged are as follows:
Loop $#IDs index
    echo $IDs[$index]
EndLoop

# ========= Load EMProperties =========
Call EMProperties

SuppressReports 

    # ==== Open Beam valve =======
    SetBeamBlank 1
    ReportColumnOrGunValve 
    If $repVal1 == 0
        SetColumnOrGunValve 1
    EndIf


    # ========= Open/Reset Navigator File =========
    ReportIfNavOpen
    If $repVal1 == 2
        CloseNavigator 
    EndIf
    OpenNavigator


    # ========= Load EMProperties =========
    Call EMProperties


    # ========= Set  mode for detector =========
    If $useDE == 0
        SetK2ReadMode V 1
        SetK2ReadMode F 1
        SetK2ReadMode T 1
        SetK2ReadMode R 1
        SetK2ReadMode M 1

        montBin = 1
    ElseIf $useDE == 1
        SetK2ReadMode V 1
        SetK2ReadMode F 1
        SetK2ReadMode T 1
        SetK2ReadMode R 1
        SetK2ReadMode M 1

        montBin = 2
    ElseIf $useDE == 2    # Old DE server
        SetK2ReadMode V 0
        SetK2ReadMode F 0
        SetK2ReadMode T 0
        SetK2ReadMode R 0
        SetK2ReadMode M 0

        montBin = 1
    ElseIf $useDE == 3
        SetK2ReadMode V 0
        SetK2ReadMode F 0
        SetK2ReadMode T 0
        SetK2ReadMode R 0
        SetK2ReadMode M 0

        montBin = 2
    EndIf
    KeepCameraSetChanges 



Loop $#IDs index
    echo Montaging grid with the ID of $IDs[$index]...
    echo Swapping grids... Please wait.
    SetColumnOrGunValve 0
    #RunInShell python $GRIDSWAPPER_PATH\Transfer_Cartridge.py $IDs[$index] 
    echo $GRIDSWAPPER_PATH\GridSwapper.exe "$IDs[$index] 3 0"
    RunInShell $GRIDSWAPPER_PATH\GridSwapper.exe "$IDs[$index] 3 0"
    Gridswap = $RepVal1 #0 is fine, 1 is failed
    SetColumnOrGunValve 1
    MakeDirectory $PARENT_DIRECTORY\$IDs[$index]
    Delay 1
    # ========= Take Atlas map =========
    # Setup illumination for Atlas
    SetLowDoseMode 0

    # Set CLapt1 for CARM200
    If $scope_type == 0
        CLapt_type = 1
    # Set CLapt1 for CARM300
    ElseIf $scope_type == 1
        CLapt_type = 0
    EndIf
    Delay 1 sec

    CallFunction Funcs::SetAtlasIllumination $removeCLapt
    SetSpotSize $spot_atlas
    MoveStageTo 0 0

    If $Gridswap == 0
    # ========= Setup file path for Atlas map =========
 
        OpenNavigator

        # ---------- To avoid wired error -----------
        Try 
            SetBinning M $montBin
            SetExposure M $exp_time
            SetBinning R $montBin
            SetExposure R $exp_time
            KeepCameraSetChanges 
           OpenNewMontage $piece_X $piece_Y  $PARENT_DIRECTORY\$IDs[$index]\$IDs[$index].mrc
           #SetMontageParams 1 $overlap_x  $overlap_y $frame_x $frame_y 0 1
        Catch 
           Echo ===> Hi :)
        EndTry 

        CloseFile 
        RemoveFile $PARENT_DIRECTORY\$IDs[$index]\$IDs[$index].mrc
        OpenNewMontage $piece_X $piece_Y  $PARENT_DIRECTORY\$IDs[$index]\$IDs[$index].mrc
        SetBinning M $montBin
        SetExposure M $exp_time
        SetBinning R $montBin
        SetExposure R $exp_time
        KeepCameraSetChanges 
        # ----------------------------------------------------

        SetMontageParams 1 $overlap_x  $overlap_y $frame_x $frame_y 0 $montBin
        Montage
        NewMap 
        SaveNavigator $PARENT_DIRECTORY\$IDs[$index]\$IDs[$index]_nav.nav
        CloseNavigator 
        CloseFile

        SaveToOtherFile A JPG NONE $PARENT_DIRECTORY\$IDs[$index]\atlas_snap.jpg

        echo Montage of ID $IDs[$index] has been completed.
    Else
        echo Grids were not swapped. Please investigate.
        Pause TransferCartrige error. Please investigate.
    EndIf
    Delay 5 sec
EndLoop

echo All montages have been acquired.
SetColumnOrGunValve 0
EndMacro
Macro	1
ScriptName PrepAtlas

# ============================
# Parameters
# ============================
exp_time = 1 # [sec]
spot_atlas = 7 # Spot Size for Atlas
removeCLapt = 1 # 0:No defualt for 3300,  1:Yes
CLapt_sizeID = 1

# Set initial BeamShift before taking atlas?
setBSforAtlas = 0 # 0:No, 1:Yes

# Set initial BeamShift after taking atlas?
setBSforSquare = 0 # 0:No, 1:Yes

shift_atlas2square = 1 # 0:No, 1:Yes, 2: Show dialog box

#For DE
# 0: Do not use DE
# 1: DE64
# 2: DE64 with old DE Server (<2.1)
# 3: Apollo
useDE = 0

# ============================
# main
# ============================

CallFunction PrepAtlas::Core

##############################################################

Function Core

    SuppressReports 

    # ==== Open Beam valve =======
    SetBeamBlank 1
    ReportColumnOrGunValve 
    If $repVal1 == 0
        SetColumnOrGunValve 1
    EndIf


    # ========= Open/Reset Navigator File =========
    ReportIfNavOpen
    If $repVal1 == 2
        CloseNavigator 
    EndIf
    OpenNavigator


    # ========= Setup file path for Atlas map =========
    # Setup Session directory
    Echo ===> Please set your session directory.
    UserSetDirectory
    ReportDirectory
    session_dir = $repVal1

    # Setup atlas map file
    atlas_map @= Atlas.mrc
    path_to_atlas = $session_dir\$atlas_map

    CloseFile
    RemoveFile $path_to_atlas


    # ========= Load EMProperties =========
    Call EMProperties


    # ========= Set  mode for detector =========
    If $useDE == 0
        SetK2ReadMode V 1
        SetK2ReadMode F 1
        SetK2ReadMode T 1
        SetK2ReadMode R 1
        SetK2ReadMode M 1

        montBin = 1
    ElseIf $useDE == 1
        SetK2ReadMode V 1
        SetK2ReadMode F 1
        SetK2ReadMode T 1
        SetK2ReadMode R 1
        SetK2ReadMode M 1

        montBin = 2
    ElseIf $useDE == 2    # Old DE server
        SetK2ReadMode V 0
        SetK2ReadMode F 0
        SetK2ReadMode T 0
        SetK2ReadMode R 0
        SetK2ReadMode M 0

        montBin = 1
    ElseIf $useDE == 3
        SetK2ReadMode V 0
        SetK2ReadMode F 0
        SetK2ReadMode T 0
        SetK2ReadMode R 0
        SetK2ReadMode M 0

        montBin = 2
    EndIf
    KeepCameraSetChanges 

    # ========= Take Atlas map =========
    # Setup illumination for Atlas
    SetLowDoseMode 0

    # Set CLapt1 for CARM200
    If $scope_type == 0
        CLapt_type = 1
    # Set CLapt1 for CARM300
    ElseIf $scope_type == 1
        CLapt_type = 0
    EndIf
    Delay 1 sec
    #RunInShell python $path_to_rootSPA\Tool\InsertCLapt.py $CLapt_type $CLapt_sizeID

    CallFunction Funcs::SetAtlasIllumination $removeCLapt
    SetSpotSize $spot_atlas
    MoveStageTo 0 0

    # ---------- To avoid wired error -----------
    Try 
        SetBinning M $montBin
        SetExposure M $exp_time
        SetBinning R $montBin
        SetExposure R $exp_time
        KeepCameraSetChanges 
        OpenNewMontage $piece_X $piece_Y $path_to_atlas
    Catch 
        Echo ===> Hi :)
    EndTry 

    CloseFile
    RemoveFile $path_to_atlas
    OpenNewMontage $piece_X $piece_Y $path_to_atlas
    SetBinning M $montBin
    SetExposure M $exp_time
    SetBinning R $montBin
    SetExposure R $exp_time
    KeepCameraSetChanges 
    # ----------------------------------------------------

    SetMontageParams 1 $overlap_x  $overlap_y $frame_x $frame_y 0 $montBin
    M
    NewMap
    SaveNavigator $session_dir\navigator.nav

    SaveToOtherFile A JPG NONE $session_dir\atlas_snap.jpg


    # ===== Reset =====
    #ReInsertAperture 0
    CloseFile

    # ========= Atlas to Square =========
    If $shift_atlas2square == 0
        apply_atlas2square = 0
    ElseIf $shift_atlas2square == 1
        apply_atlas2square = 1
    ElseIf $shift_atlas2square == 2
        YesNoBox Go to Square mag?
        #0:no, 1:yes
        If $repVal1 == 1 
            apply_atlas2square = 1
        EndIf
    EndIf

    If $apply_atlas2square == 1
        SetMag $default_SquareMag
        CallFunction Funcs::SetSquareIllumination
        Echo Apply shift [$atlas2square_x, $atlas2square_y]
        
        div = 3
        smallShiftX = $atlas2square_x / $div 
        smallShiftY = $atlas2square_y / $div 
        Loop $div iter
            ShiftItemsByMicrons $smallShiftX $smallShiftY
        EndLoop

    Else
        Echo ===> Do not apply shift.
    EndIf

EndFunction
EndMacro
Macro	2
ScriptName TakeSquares

# =================
# Parameters
# =================
exp_time = 1 #[sec]
SquareMag = 150
# x150 for 200mesh
# x200 for 300mesh
# x250 for 400mesh

removeCLapt = 1 # 0:No default for 3300, 1:Yes
CLapt_sizeID = 1
CLapt_sizeIDforLD = 1

# Set initial BeamShift after taking squares?
setBSforLD = 0

#For DE
# 0: Do not use DE
# 1: DE64
# 2: DE64 with old DE Server (<2.1)
# 3: Apollo
useDE = 0

shift_atlas2square = 0 # 0:No, 1:Yes, 2: Show dialog box
adjust_eucentric = 1 # 0:No, 1:Yes, 2: Show dialog box
shift_square2view = 1 # 0:No, 1:Yes, 2: Show dialog box

# Used for adding square when you are at LowDoseMode
shift_view2square = 1 # 0:No, 1:Yes

# ============================
# Main
# ============================

CallFunction TakeSquares::Core

#########################################################

Function Core

    Call EMProperties

    # Initialize
    apply_atlas2square = 0
    apply_square2view = 0
    apply_eucentric = 0

    SetBeamBlank 1

    IsVariableDefined FIRSTNAVDONE
    If $repVal1 == 0
        # set FIRSTNAVDONE
        FIRSTNAVDONE := 0
    EndIf 

    #==== Open Beam valve =======
    ReportColumnOrGunValve 
    If $repVal1 == 0
        SetColumnOrGunValve 1
    EndIf 
    #=========================

    # ========= Set  mode for detector =========
    If $useDE == 0
        SetK2ReadMode V 1
        SetK2ReadMode F 1
        SetK2ReadMode T 1
        SetK2ReadMode R 1
        SquareBin = 1
    ElseIf $useDE == 1
        SetK2ReadMode V 1
        SetK2ReadMode F 1
        SetK2ReadMode T 1
        SetK2ReadMode R 1
        SquareBin = 2
    ElseIf $useDE == 2    # Old DE server
        SetK2ReadMode V 0
        SetK2ReadMode F 0
        SetK2ReadMode T 0
        SetK2ReadMode R 0
        SquareBin = 1
    ElseIf $useDE == 3
        SetK2ReadMode V 0
        SetK2ReadMode F 0
        SetK2ReadMode T 0
        SetK2ReadMode R 0
        SquareBin = 2
    EndIf
    SetBinning R $SquareBin
    KeepCameraSetChanges
    #=========================

    ReportMag mag
    ReportLowDose
    isLowDoseOn = $repVal1

    SetLowDoseMode 0
    SetSlitIn 0
    SetMag $SquareMag
    Delay 5 sec
    CallFunction Funcs::SetSquareIllumination $removeCLapt

    If $FIRSTNAVDONE == 0
        ReportNumNavAcquire 
        numOfNaviAcq = $repVal1
        echo $numOfNaviAcq squares are selected and about to exposure...

        #====  Align FOV from Atlas to Square=======
        If $shift_atlas2square == 0
            apply_atlas2square = 0
        ElseIf $shift_atlas2square == 1
            apply_atlas2square = 1
        ElseIf $shift_atlas2square == 2
            YesNoBox Do Align FOV Atlas to Square? (Nomally -> Yes. / If you already have square maps -> No)
            #0:no, 1:yes
            If $repVal1 == 1 
                apply_atlas2square = 1
            EndIf
        EndIf

        If $apply_atlas2square == 1
            Echo ===> Apply shift (Atlas to Square).
            Echo Apply shift [$atlas2square_x, $atlas2square_y]
            ShiftItemsByMicrons $atlas2square_x $atlas2square_y
        Else
            Echo ===> Do not apply shift (Atlas to Square).
        EndIf
        #=========================

        #====  Align FOV from View to Square =======
        If $shift_view2square == 0
            apply_view2squar = 0
        ElseIf $shift_view2square == 1
            apply_view2squar = 1
        EndIf

        If $apply_view2squar == 1
            If ($mag >= 1500) OR ($isLowDoseOn == 1)
                Echo ===> Apply shift (View to Square).
                view2squar_x = -1 * $square2view_x
                view2squar_y = -1 * $square2view_y
                Echo Apply shift [$view2squar_x, $view2squar_y]
                ShiftItemsByMicrons $view2squar_x $view2squar_y
            Else
                Echo ===> Do not apply shift (View to Square). LowMag is already ON.
            EndIf
        Else
            Echo ===> Do not apply shift (View to Square).
        EndIf
        #=========================


        #==== Adjust eucentric height ====
        If $adjust_eucentric == 0
            apply_eucentric = 0
        ElseIf $adjust_eucentric == 1
            apply_eucentric = 1
        ElseIf $adjust_eucentric == 2
            YesNoBox Adjust global eucentric hight ?
            If $repVal1 == 1
                apply_eucentric = 1
            EndIf
        EndIf

        If $apply_eucentric == 1
            Echo ===> Adjust eucentircity for whole squares
            ReportTiltAngle TT
            Eucentricity 1
            MoveStage 0 0 $eucentric_offset
            UpdateGroupZ 
            TiltTo $TT
        Else
            Echo ===> Skip adjust eucentircity for whole squares
        EndIf
        #=========================

        FIRSTNAVDONE := 1
        MoveToNavItem 
    EndIf 

    SaveNavigator
    CloseFile

    ReportMag mag

    # Current Item info
    ReportNavItem
    item_label = $navLabel

    # Set file path for square maps
    ReportNavFile 1
    path_to_navfile = $repVal3

    squares @= squares.mrc
    path_to_squares = $path_to_navfile$squares
    Echo ===> DEBUG path_to_squares $path_to_squares 

    # Set SquareJPG folder
    sqJPG_dir @= SquaresJPG
    path_to_sqJPG = $path_to_navfile$sqJPG_dir
    RunInShell mkdir $path_to_sqJPG

    totalSq @= TotalSquares.txt
    path_to_totalSq = $path_to_sqJPG\$totalSq

    # Set square number
    Try 
        ReadTextFile sq_count $path_to_totalSq
     Catch
        sq_count = 0
     EndTry

    # Open square file
    Try
        OpenOldFile $path_to_squares
    Catch
        OpenNewFile $path_to_squares
    EndTry

    # Set JPG square image and info name 
    sqJPG_file = sq_$sq_count.jpg
    ##sqJPG_file = $sq_count_SquareMap$item_label.jpg
    path_to_sqJPG_file = $path_to_sqJPG\$sqJPG_file
    sqJPG_info = sqJPG_info_$sq_count.txt
    path_to_sqJPG_info = $path_to_sqJPG\$sqJPG_info

    # Set camera parameters
    SetExposure R $exp_time
    SetDoseFracParams R 0 0 0 0 0

    # Take square image and save it
    R
    Save
    NewMap
    SaveToOtherFile A JPG NONE $path_to_sqJPG_file

    # Reset Camera parameters
    RestoreCameraSet R

    # Save image info 
    ImageProperties A
    dimX = $repVal1
    dimY = $repVal2
    bin = $repVal3
    exposure = $repVal4
    pixsize = $repVal5 #[nm]
    ReportMagIndex mag_idx

    RunInShell echo SquareMap : $item_label >> $path_to_sqJPG_info
    RunInShell echo MagIndex : $mag_idx >> $path_to_sqJPG_info
    RunInShell echo Mag : $mag >> $path_to_sqJPG_info
    RunInShell echo X : $dimX >> $path_to_sqJPG_info
    RunInShell echo Y : $dimY >> $path_to_sqJPG_info
    RunInShell echo bin : $bin >> $path_to_sqJPG_info
    RunInShell echo exposure : $exposure >> $path_to_sqJPG_info
    RunInShell echo pixsize : $pixsize >> $path_to_sqJPG_info

    sq_count = $sq_count + 1
    RunInShell echo $sq_count > $path_to_totalSq

    CloseFile

    #====  Call AlignFOV_AtlasToSquare ====
    ReportNumNavAcquire
    If $repVal1 == 1
        If $shift_square2view == 0
            apply_square2view = 0
        ElseIf $shift_square2view == 1
            apply_square2view = 1
        ElseIf $shift_square2view == 2
            YesNoBox Did you finish taking squares and go to LowDose mode?
            If $repVal1 == 1 
                apply_square2view = 1
            EndIf
        EndIf

        If $apply_square2view == 1

            If $useDE == 1
                SetK2ReadMode V 0
                SetK2ReadMode F 0
                SetK2ReadMode T 0
                SetK2ReadMode R 1

                SetBinning V 2
                SetBinning F 2
                SetBinning T 2
                SetBinning R 2
            EndIf
            KeepCameraSetChanges 

            Echo ===> Apply shift (Square to View).
            ShiftItemsByMicrons $square2view_x $square2view_y
            SetLowDoseMode 1
            GoToLowDoseArea R
            GoToLowDoseArea V

            Call SetFocusTrialPosition

            # Set CLapt1 for CARM200
            If $scope_type == 0
                CLapt_type = 1
            # Set CLapt1 for CARM300
            ElseIf $scope_type == 1
                CLapt_type = 1
            EndIf
            RunInShell python $path_to_rootSPA\Tool\InsertCLapt.py $CLapt_type $CLapt_sizeIDforLD
        Else
            Echo ===> Do not apply shift (Square to View).
        EndIf

        If $setBSforLD == 1
            GoToLowDoseArea V
            SetBeamShift $LowDoseBSX $LowDoseBSY
        EndIf

        FIRSTNAVDONE := 0
    EndIf 

EndFunction
EndMacro
Macro	3
ScriptName FromAtlasToSquare

# ============================
# Main
# ============================

Call EMProperties

# Setting for Atlas and Square
shift_x =  $atlas2square_x 
shift_y = $atlas2square_y

factor = 1
#YesNoBox "Shift nomally? Yes: Shift nomally. (Atlas to Square) / No: Shift opposite way. (Square to Atlas)"
#       #0:no, 1:yes
#       If $repVal1 == 1 
#          factor = 1
#       Else
#          factor = -1
#       Endif 

shift_x = $shift_x * $factor
shift_y = $shift_y * $factor

Echo shift_x = $shift_x 
Echo shift_y = $shift_y

div = 3
smallShiftX = $atlas2square_x / $div 
smallShiftY = $atlas2square_y / $div 
Loop $div iter
    ShiftItemsByMicrons $smallShiftX $smallShiftY
EndLoop
EndMacro
Macro	4
ScriptName FromSquareToLowDose

# ============================
# Main
# ============================

Call EMProperties

# Setting for Square:x150
shift_x = $square2view_x
shift_y = $square2view_y

YesNoBox "Shift nomally? Yes: Shift nomally. (Square to LowDose) / No: Shift opposite way. (LowDose to Square)"
       #0:no, 1:yes
       If $repVal1 == 1 
          factor = 1
       Else
          factor = -1
       Endif 

shift_x = $shift_x * $factor
shift_y = $shift_y * $factor

Echo shift_x = $shift_x 
Echo shift_y = $shift_y

ShiftItemsByMicrons $shift_x $shift_y 
EndMacro
Macro	6
ScriptName SetFocusTrialPosition

    # ============================
    # Main
    # ============================

    SuppressReports 

    ReportNavFile 1
    path_to_navfile = $repVal3
    vector_file = $path_to_navfileVectors_ViewMode.txt
    init_vector_file = $path_to_navfileVector_init_param.txt

    DoesFileExist $path_to_navfileFocusTrialPosition.txt
    If $repVal1 == 1
        ReadTextFile dist_and_angle $path_to_navfileFocusTrialPosition.txt
        dist = $dist_and_angle[1]
        angle = $dist_and_angle[2]

        SetLowDoseMode 1
        GoToLowDoseArea R
        GoToLowDoseArea V

        SetAxisPosition F $dist $angle
        SetAxisPosition T $dist $angle
        Exit
    EndIf
    

    open_agin = 0
    Try
        ReadTextFile vector $vector_file
        IS_RAD1_h   = $vector[1]
        IniAng_h    = $vector[2]
        IS_RAD1_v   = $vector[3]
        IniAng_v    = $vector[4]
        dist = ($IS_RAD1_h + $IS_RAD1_v) / 2
        dist = $dist * SQRT (2) / 2
        angle = ($IniAng_h + $IniAng_v) / 2
        angle = MODULO ($angle  90)

        Echo ===> Read $vector_file
        Echo ===> Dist : $dist [um], Angle : $angle [degree]

    Catch
        Echo ===> Cannot find $vector_file
        Echo ===> Try to open $init_vector_file
        open_agin = 1
    EndTry

    If $open_agin == 1
        Loop 3 idx
            Try 
                ReadTextFile vector $init_vector_file
                IS_RADIUS = $vector[1]
                IS_ANGLE = $vector[2]
                dist = $IS_RADIUS * SQRT (2) / 2
                angle_tmp = $IS_ANGLE + 45
                angle = MODULO ($angle_tmp 90)
                Break
            Catch
                Echo ===> No vector info...
                Echo ===> Need to run "FindInitVectors" or "FindVectors" beforehand.
                Echo ===> Run automatically.
                Call FindInitVectors
            EndTry

            If $idx == 3
                Echo ===> Program failed.
                Exit
            EndIf
        EndLoop
    EndIf

    # SetLowDoseMode 1
    # GoToLowDoseArea R

    # SetLowDoseMode 0
    # Delay 2 sec
    # CurrentSettingsToLDArea F
    # Delay 2 sec
    # SetLowDoseMode 0
    # Delay 2 sec
    # CurrentSettingsToLDArea T
    # Delay 2 sec

    SetLowDoseMode 1
    GoToLowDoseArea R
    GoToLowDoseArea V

    SetAxisPosition F $dist $angle
    SetAxisPosition T $dist $angle

    RunInShell echo $dist $angle > $path_to_navfileFocusTrialPosition.txt
EndMacro
Macro	7
ScriptName Z_byV

    Call EMProperties

    Echo ===> Running Z_byV ...
    #====================================
    # for defocus offset of V in Low Dose, save it
    # ===================================
    GoToLowDoseArea V

    #==================
    # set object lens 
    #==================
    ReportBeamShift 
    bsx_beforeZbyV = $repVal1
    bsy_beforeZbyV = $repVal2

    SaveFocus # this is used for "High defocus mag"
    SetEucentricFocus

    Echo ===> Offset for Z_byV is $offset_for_Z_byV [um]
    Echo ===> Target is $offset_for_Z_byV [um]

    #===========
    # Adjust Z
    #===========
    ReportStageXYZ originalX originalY originalZ

    Loop 10
        ReportStageXYZ
        If ($repVal3 < $safty_z_lower) OR ( $safty_z_upper < $repVal3)
            Echo ===> Fail to adjust Eucentric height.
            Echo ===> Back to original Stage height.
            MoveStageTo $originalX $originalY $originalZ

            skip_message @= "Skipped. Hit the limitation of Z height."
            CallFunction Funcs::AnnotateSkipItem $skip_message
            SkipAcquiringNavItem 
            Exit
        EndIf

        IsVariableDefined targetZ
        If $repVal1 == 0
            targetZ = 0
        EndIf
        Echo ===> targetZ : $targetZ

        G -1 1 # autofocus with view
        ReportAutofocus DEF
        DEF = $DEF - $offset_for_Z_byV - $targetZ
        Z = -1 * $DEF
        relax_z1 = $Z / ABS ($Z) * $backlash_z
        relax_z2 = -1 * $relax_z1
        range = abs $DEF

    #If ($range < 5) # if less than 2 ?m, finish function.
    If (-3 < $DEF) AND ($DEF < 3)
        Break
    EndIf

        MoveStage 0 0 $relax_z1
        MoveStage 0 0 $Z
        MoveStage 0 0 $relax_z2

        Z = ROUND $Z 2
        Echo Z has moved --> $Z micron 
    EndLoop

    #=========================================
    # restore the defocus set in V originally
    # ========================================
    RestoreFocus # this is used for "High defocus mag"
    SetBeamShift $bsx_beforeZbyV $bsy_beforeZbyV
EndMacro
Macro	8
ScriptName AlignToHole

    # ============================
    # Main
    # ============================
    # Initialize 
    SuppressReports 
    Call EMProperties
    Call Parameters

    Echo ====== Start AlignToHole =======

    # Set View mode of LD
    SetLowDoseMode 1
    GoToLowDoseArea V

    # Read hole image
    CallFunction Funcs::ReadHoleImage

    # Start hole alignment
    CallFunction Funcs::AlignToHole $maxholeshift $max_align_iter $align_byIS $template_buffer
EndMacro
Macro	9
ScriptName AutoFocusRoutine

# ============================
# Parameters
# ============================
focus_error = 0.3 #[um]
settle_time = 10 #[sec]
focus_by_Z = 1

# Usually need not to change below
backlash_z = 0.9
max_focusZ_iter = 10
max_focus_iter = 8
z_settle_time = 3

# ============================
# Main
# ============================

Echo ===== Start AutoFocusRoutine =====

SuppressReports

updata_Z_afterFocus = 0

Call EMProperties

# GoTo Focus mode
SetLowDoseMode 1
GoToLowDoseArea F

# Fix target defocus just in case
ReportTargetDefocus
target_defocus = -1 * ABS ($repVal1)
SetTargetDefocus $target_defocus

# Set lowest and highest threashold
range = ABS ($focus_error) * 5
focus_th_low = $repVal1 + $range #[um]
focus_th_high = $repVal1 - $range #[um]

# Set standard focus
Echo ===> Set standard focus
SetEucentricFocus

# Start AutoFocusing
If $focus_by_Z == 1
    CallFunction CustomAutoFocus::AutoFocus_byZ
Else
    CallFunction CustomAutoFocus::AutoFocus_byOL
EndIf

# Wait for settling
Echo ===> Settling $settle_time [sec]
Delay $settle_time sec

Echo ===> Finish AutoFocusing
EndMacro
Macro	10
ScriptName TestShot

# ============================
# Parameters
# ============================
#1: K2, 
#2: K3, 
#3: DE64,
#4, Apollo
detector_type = 2

measure_ice_thickness = 1 # 0: No, 1:Yes

show_optimized_doseFrac = 1 # 0: No, 1: Yes
target_total_dose = 40 # [e/A^2]
target_dose_per_frame = 1 # [e/frame]

# ============================
# Main
# ============================

SuppressReports

Echo ======= Start DoseCheck =======

# Set save folder
ReportNavFile 1
path_to_navfile = $repVal3
doseCheck_dir @= TestShot
path_to_doseCheck = $path_to_navfile$doseCheck_dir
RunInShell mkdir $path_to_doseCheck

img_count_file @= img_count.txt
path_to_img_count @= $path_to_doseCheck\$img_count_file

# DoesFileExist $path_to_img_count
# If $repVal1 == 0
#     img_count = 0
# Else
#     ReadTextFile img_count $path_to_img_count
# EndIf

Try 
    ReadTextFile img_count $path_to_img_count
Catch
    img_count = 0
EndTry

img_count = $img_count + 1

img_file @= $path_to_doseCheck\$img_count.jpg
img_file_mrc @= $path_to_doseCheck\$img_count.mrc
img_info_file @= $path_to_doseCheck\$img_count_info.txt
#fft_img  @= $path_to_doseCheck\$itemIdx_X$stage_x_Y$stage_y_FFT.jpg

# Recommended dose rate
If ($detector_type == 1)
    thresh_dose = 8.0
    # Set camera params
    SetBinning R 1
    #SetDoseFracParams R 1 0 0 0 0
    Echo ===> Binning is set to 1

ElseIf ($detector_type == 2)
    ReportK3CDSmode
    If ($repVal1 == 1)
        thresh_dose = 13.0
    Else
        thresh_dose = 15.0
    EndIf
    # Set camera params
    SetBinning R 1
    #SetDoseFracParams R 1 0 0 0 0
    Echo ===> Binning is set to 1

ElseIf ($detector_type == 3)
    thresh_dose = 3.5
    # Set camera params
    SetBinning R 2
    #SetDoseFracParams R 1 0 0 0 0
    Echo ===> Binning is set to 2

ElseIf ($detector_type == 4)
    thresh_dose = 40
    # Set camera params
    SetBinning R 2
    #SetDoseFracParams R 1 0 0 0 0
    Echo ===> Binning is set to 2

Else
    Echo ===> No detector type of $detector_type.
    Exit
EndIf

KeepCameraSetChanges R

ReportTargetDefocus TD

ReportEnergyFilter 
initailslit_status = $repVal3

#SetDoseFracParams R 1 1 0 0 0

# Take record and measure ice thickness
ice_thickness = 1
GoToLowDoseArea R
If $measure_ice_thickness == 1
    # Slit out
    SetExposure R 0.5
    If $detector_type == 4
        SetExposure R 0.1
    EndIf
    SetDoseFracParams R 0 0 0 0 0
    SetSlitIn 0
    UpdateLowDoseParams R
    R
    ElectronStats A
    electron_count_NoSlit = $repVal5
    RestoreCameraSet R
    RestoreLowDoseParams R

    # SlitIn
    SetSlitIn 1
    UpdateLowDoseParams R
    R
    RestoreLowDoseParams R
    ElectronStats A
    electron_count = $repVal5
    ice_thickness = $electron_count / $electron_count_NoSlit
    If $ice_thickness > 1
        ice_thickness = 1
    EndIf
Else
    R
    ElectronStats A
    electron_count = $repVal5
EndIf

# Save image
SaveToOtherFile A JPG NONE $img_file
SaveToOtherFile A MRC NONE $img_file_mrc
#SaveToOtherFile AF JPG NONE $fft_img
RunInShell echo $img_count > $path_to_img_count

# Imaging info
ImageProperties A
bin = $repVal3
exposure = $repVal4
pixsize = $repVal5 * 10

dose_rate = ( $electron_count / ($pixsize * $pixsize) ) / $ice_thickness
total_dose = $dose_rate * $exposure
solved_exposure = $target_total_dose / $dose_rate
solved_frame_num = NEARINT ($target_total_dose / $target_dose_per_frame)
solved_frame_time = $solved_exposure / $solved_frame_num

SetSlitIn $initailslit_status

ReportAlpha alpha
ReportSpotSize spot_size
ReportPercentC2 brightness
ReportMag mag
ReportStageXYZ stage_x stage_y stage_z

RunInShell echo $img_count > $img_count_file

# Report imaging info
Echo ----------------------------------------------------------------
RunInShell echo ---------------------------------------------------------------- >> $img_info_file

If $detector_type == 1
    Echo Detector type : K2
    RunInShell echo Detector type : K2 >> $img_info_file
ElseIf $detector_type == 2
    Echo Detector type : K3
    RunInShell echo Detector type : K3 >> $img_info_file
ElseIf $detector_type == 3
    Echo Detector type : DE64
    RunInShell echo Detector type : DE64 >> $img_info_file
ElseIf $detector_type == 4
    Echo Detector type : Apollo
    RunInShell echo Detector type : DE64 >> $img_info_file
Else
    Echo Detector type : Other
    RunInShell echo Detector type : Other >> $img_info_file
EndIf


Echo Image number : $img_count
Echo Spot : $spot_size
Echo Angle : $alpha
Echo Binning : $bin
Echo Pixel size (at x$mag) : $pixsize [A/px]
Echo Exposure time : $exposure [sec]
Echo Target Defocus : $TD [um]
Echo Acquisition point (X, Y, Z) : $stage_x $stage_y $stage_z [um]

RunInShell echo Image number : $img_count >> $img_info_file
RunInShell echo Spot : $spot_size >> $img_info_file
RunInShell echo Angle : $alpha >> $img_info_file
RunInShell echo Binning : $bin >> $img_info_file
RunInShell echo Pixel size (at x$mag) : $pixsize [A/px] >> $img_info_file
RunInShell echo Exposure time : $exposure [sec] >> $img_info_file
RunInShell echo Target Defocus : $TD [um] >> $img_info_file
RunInShell echo Acquisition point (X, Y, Z) : $stage_x $stage_y $stage_z [um] >> $img_info_file

# Report dose info
If ($detector_type == 1) OR ($detector_type == 2) OR ($detector_type == 3) OR ($detector_type == 4)
    Echo ----------------------------------------------------------------
    RunInShell echo ---------------------------------------------------------------- >> $img_info_file

    Echo Dose rate on specimen : $dose_rate [e/A^2/s]
    Echo Total dose on specimen : $total_dose [e/A^2]
    Echo Dose rate on detector : $electron_count [e/px/s]

    RunInShell echo Dose rate on specimen : $dose_rate [e/A^2/s] >> $img_info_file
    RunInShell echo Total dose on specimen : $total_dose [e/A^2] >> $img_info_file
    RunInShell echo Dose rate on detector : $electron_count [e/px/s] >> $img_info_file

    If $measure_ice_thickness == 1
        Echo Ice thickness : $ice_thickness (Ratio : SlitOut / SlitIn)
        RunInShell echo Ice thickness : $ice_thickness >> $img_info_file (Ratio : SlitOut / SlitIn)
    EndIf

    If $electron_count > $thresh_dose
        Echo !!! You should lower the dose rate to avoid coinsidence loss.
        Echo !!! Recommended dose rate on your detector is less than $thresh_dose [e/px/s]

        RunInShell echo !!! You should lower the dose rate to avoid coinsidence loss. >> $img_info_file
        RunInShell echo !!! Recommended dose rate on your detector is less than $thresh_dose [e/px/s] >> $img_info_file

        OKBox "!!! You should lower the dose rate to avoid coinsidence loss. Recommended dose rate on your detector is less than $thresh_dose [e/px/s]"
    EndIf

    If $show_optimized_doseFrac == 1
        Echo ----------------------------------------------------------------
        RunInShell echo ---------------------------------------------------------------- >> $img_info_file
        Loop 10
            If ( MODULO ($solved_frame_num 2) == 0 )
                actual_dose_per_frame = $target_total_dose / $solved_frame_num
                Echo Optimized exposure time (Target total dose : $target_total_dose [e/A^2]) : $solved_exposure [sec]
                Echo Optimized frame time (Target dose per frame : $target_dose_per_frame [e/A^2/frame]) : $solved_frame_time [sec]
                Echo Number of frames : $solved_frame_num
                Echo Dose per frame : $actual_dose_per_frame [e/A^2/frame]

                RunInShell echo Optimized exposure time (Target total dose : $target_total_dose [e/A^2]) : $solved_exposure [sec] >> $img_info_file
                RunInShell echo Optimized frame time (Target dose per frame : $target_dose_per_frame [e/A^2/frame]) : $solved_frame_time [sec] >> $img_info_file
                RunInShell echo Number of frames : $solved_frame_num >> $img_info_file
                RunInShell echo Dose per frame : $actual_dose_per_frame [e/A^2/frame] >> $img_info_file

                If ($detector_type == 1) AND ($solved_frame_time < 0.1)
                    Echo !!! You must set longer frame time for K2!
                    Echo !!! Check dose again with reduced beam intensity!

                    RunInShell echo !!! You must set longer frame time for K2! >> $img_info_file
                    RunInShell echo !!! Check dose again with reduced beam intensity! >> $img_info_file

                    OKBox "!!! You must set longer frame time for K2! Check dose again with reduced beam intensity!"
                ElseIf ($detector_type == 3)
                    Echo ===> You need to set the parameters manually. (DE camera)
                Else
                    YesNoBox Set optimized parameters? "Exposure time: $solved_exposure [s]" "Frame time: $solved_frame_time [s]"
                    If $repVal1 == 1
                        SetExposure R $solved_exposure
                        SetFrameTime R $solved_frame_time
                        KeepCameraSetChanges
                    EndIf
                EndIf

                Break
            Else
                solved_frame_time = $solved_frame_time * 0.99 
                solved_frame_num = NEARINT ($solved_exposure / $solved_frame_time)
            EndIf
        EndLoop

    EndIf
    
EndIf

Echo ===> Finish
EndMacro
Macro	12
ScriptName FindVectorsRoutine
    
    # ============================
    # Parameters
    # ============================
    grid_type = 1 # 0: lacey, 1: quantifoil,UltrAufoil
    targetZ = -20 # [um]

    doAll = 0 #0:No, will skip FindInitVector if it has already done. 1:Yes

    # ============================
    # Main
    # ============================
    # Z_byV
    #Echo ===> Z_byV
    #Call Z_byV

    # Hole alignment
    If $grid_type != 0
        Echo ===> Aligning to hole
        Call AlignToHole
    ElseIf $grid_type == 0
        Echo ===> Grid is lacey. Skip hole alignment.
    EndIf

    # Find initial vector param
    initVecFile @= Vector_init_param.txt
    DoesFileExist $path_to_navfile$initVecFile
    If ( $repVal1 == 0 ) OR ( $doAll == 1 )
        Echo ===> Calculating the initial parameters
        Call FindInitVectors
    EndIf
    
    # Refine vector param
    Echo ===> Refining the parameters
    Call FindVectors
    #CallFunction FindVectors::Core $grid_type $exposure_time $manual $NumberShots $IS_RAD2 $IS_RADIUS $IS_ANGLE $BTtoIS

    # Check multi-hole pattern
    If $grid_type != 0
        Echo ===> Check multihole pattern
        Call CheckMultiHole
    ElseIf $grid_type == 0
        Echo ===> Grid is lacey. Skip check multi-hole.
    EndIf

    # Set focus position
    Echo ===> Set Focus and Trial Position
    Call SetFocusTrialPosition
EndMacro
Macro	13
ScriptName FindInitVectors

    # ============================
    # Parameters
    # ============================
    grid_type = 1 # 0: lacey, 1: quantifoil,UltrAufoil

    # --- Parameters for non-lacey grid ---
    square_num = 0

    useK2K3 = 1 #0: No, 1: Yes

    # --- Use python or not ---
    usePython = 1 # 0: No, 1: Yes
    detector_type = 0 # Set Cameraproperty in SerialEMproperties.txt
    thresh = 0.1 # Threshold for peak search
    num_peaks = 9 # Number of peaks for peak search
    bin_factor = 1 # Binning factor to shrink image

    # --- Parameters for lacey grid ---
    point_idx1 = 85
    point_idx2 = 86
    IS_spacing = 2 #[um] Spacing for Beam-Image shift
    max_IS_dist = 20 #[um] Maximum distance for Beam-Image shift

    # ============================
    # Main
    # ============================

    SuppressReports 
    Call EMProperties

    If $usePython == 0
        #CallFunction FindInitVectors::Core1 
    Else
        CallFunction FindInitVectors::Core2 $grid_type $point_idx1 $point_idx2 $IS_spacing $max_IS_dist $detector_type $square_num $thresh $num_peaks $bin_factor
    EndIf

#################################################

    # By SerialEM implemented from 4.0
    #Function Core1

        #CloseFile 
        #Call EMProperties

        # Open image
        #buf = A
        #squareImages @= squares.mrc
        #OpenOldFile $path_to_navfile$squareImages 
        #ReadFile $square_num  $buf 

        # Get MagIndex info
        #infoFile @= $path_to_navfileSquaresJPG\sqJPG_info_$square_num.txt
        #Echo $infoFile 
        #Read2DTextFile arr2d $infoFile 
        #magIndex = $arr2d[2][3]
        #Echo MagIndex: $magIndex

        # Get image bin and pixel size
        #ImageProperties $buf
        #bin = $repVal3
        #pixsize = $repVal5

        # Get hole spacing parameters by pixel at camera
        #AutoCorrPeakVectors $buf $magIndex
        #spacing = $repVal1 # [px]
        ##spacing = $repVal1 * $pixsize / 1000
        #vecX = $repVal2 #[binned px]
        #vecY = $repVal3 #[binned px]
        #angle = ATAN2 ( $vecY $vecX )

        #Echo At Camera ===> Spacing: $spacing (X: $vecX, Y: $vecY) [px], Angle: $angle [degree] 

        ## Convert from camera to specimen
        #factor = 1
        #If $useK2K3 == 1
        #    factor = 2
        #EndIf 
        #Xin = $vecX * $bin * $factor    # [unbinned px]
        #Yin = $vecY * $bin * $factor * -1    # [unbinned px]

        #CameraToSpecimenMatrix $magIndex
        #XperX = $repVal1
        #XperY = $repVal2
        #YperX = $repVal3
        #YperY = $repVal4

        #X = $Xin * $XperX + $Yin * $XperY
        #Y = $Xin * $YperX + $Yin * $YperY

        #spacingOnSpec = SQRT ( $X * $X + $Y * $Y )
        #angleOnSpec = ATAN2 ( $Y $X )

        #Echo On Specimen ===> Spacing: $spacingOnSpec (X: $X, Y: $Y) [um], Angle: $angleOnSpec [degree] 
       # RunInShell echo $spacingOnSpec $angleOnSpec > $path_to_navfileVector_init_param.txt
        #Echo Saved as $path_to_navfileVector_init_param.txt

       # CloseFile 

     #EndFunction

#################################################

    # By Python implemented
    Function Core2 9 0 grid_type point_idx1 point_idx2 IS_spacing max_IS_dist detector_type square_num thresh num_peaks bin_factor

        # Initialize
        SuppressReports 
        path_to_propertyFile = C:\ProgramData\SerialEM\SerialEMproperties.txt

        ReportNavFile 1
        path_to_navfile = $repVal3
        #path_to_rootSPA @= C:\Users\VALUEDGATANCUSTOMER\Desktop\SerialEM_SPA
        #cmd @= python $path_to_rootSPA\Tool\FindHoleLattice.py

        If $grid_type == 1 
            Echo $cmd_findV $detector_type $square_num $thresh $num_peaks $bin_factor $path_to_propertyFile $path_to_navfile
            RunInShell $cmd_findV $detector_type $square_num $thresh $num_peaks $bin_factor $path_to_propertyFile $path_to_navfile
        Else
            ReportOtherItem $point_idx1
            x1 = $repVal2
            y1 = $repVal3

            ReportOtherItem $point_idx2
            x2 = $repVal2
            y2 = $repVal3

            x = $x2 - $x1
            y = $y2 - $y1

            dist = sqrt ( $x * $x + $y * $y )
            radius = $IS_spacing
            angle = ATAN2 ( $y $x )

            RunInShell echo $radius $angle > $path_to_navfileVector_init_param.txt

            half_width = $dist / 2
            If $half_width <= $max_IS_dist
                layer = NEARINT ( $half_width / $radius - 0.5 )
            Else
                layer = NEARINT ( $max_IS_dist / $radius - 0.5 )
            EndIf

            Echo ===========================
            Echo Distance between two stage points : $dist [um]
            Echo Radius for MultiHole : $radius [um]
            Echo Angle for MultiHole: $angle [degree]
            Echo Maximum layer : $layer
            Echo ===========================
            
            RunInShell echo $layer > $path_to_navfileLacey_layer.txt

        EndIf

    EndFunction

#################################################
EndMacro
Macro	14
ScriptName FindVectors

    # ============================
    # Parameters
    # ============================
    grid_type = 1 # 0: lacey, 1: quantifoil,UltrAufoil
    exposure_time = 3 #[sec]
    open_slit = 0 # Open slit anyway when FindVector. 0:No 1:Yes
    LayerFV = 3 # Distance for the hole pattern

    viewMag = 8000 # Magnification of View for hole alignment. Must be >= 8000

    manual = 0 # 0:No, 1:Yes

    # ------- For manual setting -------
    # For multi-shot by IS(ImageShift) 
    IS_RADIUS   = 2.9    # Distance between each holes [um]
    IS_ANGLE = 119    # Angle between X-axis of "Stage" and X-axis of "hole lattice" (or Y) [degree]
    BTtoIS = 0    # shold be 0

    # For multi-shot by IS(ImageShift) in one hole
    NumberShots = 0     # Number of shots in one hole
    IS_RAD2     = 0.35   # Radius of off-center ImageShift displacement [um]

    # ============================
    # Main
    # ============================

    Call EMProperties

    CallFunction FindVectors::Core $grid_type $exposure_time $manual $NumberShots $IS_RAD2 $IS_RADIUS $IS_ANGLE $BTtoIS

#################################################

    Function Core 8 0 grid_type exposure_time manual NumberShots IS_RAD2 IS_RADIUS IS_ANGLE BTtoIS

        SuppressReports 
        ReportNavFile 1
        path_to_navfile = $repVal3

        If $manual == 0
            ReadTextFile vector $path_to_navfileVector_init_param.txt
            IS_RADIUS = $vector[1]
            IS_ANGLE = $vector[2]
        EndIf

        SetImageShift 0 0 0 0

        # Lacey
        If $grid_type == 0
            CallFunction FindVectors::GetQuantifoilVector $grid_type $exposure_time $NumberShots $IS_RAD2 $IS_RADIUS $IS_ANGLE $BTtoIS

        # Quantifoil, UltrAuFoil
        ElseIf $grid_type == 1
            CallFunction FindVectors::GetQuantifoilVector $grid_type $exposure_time $NumberShots $IS_RAD2 $IS_RADIUS $IS_ANGLE $BTtoIS

        Else
            Echo ===> No such type of grid (grid_type: $grid_type)
        EndIf

        RunInShell echo $grid_type > $path_to_navfileGrid_type.txt

    EndFunction

#################################################

    Function GetQuantifoilVector 7 0 grid_type exposure_time NumberShots IS_RAD2 IS_RADIUS IS_ANGLE BTtoIS

        #LayerFV = 2 # Maximum distance for the hole pattern
        iter_num = 2 # For hole alignment iteration
        IS_RADIUS = $IS_RADIUS * $LayerFV

        GoToLowDoseArea V
        SaveFocus

        If $open_slit == 1
            SetSlitIn 0
        EndIf

        ReportMag initMag
        SetMag $viewMag

        UpdateLowDoseParams V

        SetEucentricFocus

        # Set camera params
        SetExposure V $exposure_time

        ReportImageShift
        isux = $RepVal1
        isuy = $RepVal2
        Echo $IS_RADIUS, $IS_ANGLE

        # Tempalte
        # Acquire first image

        If $grid_type != 0
            V
            Copy A P
        EndIf

         # --- For horizontal vector ---
        rad = $IS_RADIUS
        theta = $IS_ANGLE
        Loop $iter_num icount
            GoToLowDoseArea V
            IS_X = $rad * cos ($theta)
            IS_Y = $rad * sin ($theta)
            ImageShiftByMicrons $IS_X $IS_Y 0 $BTtoIS
            Delay 1 sec
            
            If $grid_type != 0
               Echo ===> Taking V
               V
               Echo ===> Wait 5 sec
               Delay 5 sec
               AlignTo P
               Echo ===> Did align?
            EndIf
            
            ReportSpecimenShift IS_Xn IS_Yn

            echo  IS_Xn, IS_Yn = $IS_Xn, $IS_Yn
            theta = ATAN2 ( $IS_Yn $IS_Xn )
            rad  = sqrt ($IS_Xn * $IS_Xn + $IS_Yn * $IS_Yn)

            If $icount == $iter_num
                GoToLowDoseArea R
                Delay 1 sec
                ReportSpecimenShift IS_Xn IS_Yn
                theta_r = ATAN2 ( $IS_Yn $IS_Xn )
                rad_r  = sqrt ($IS_Xn * $IS_Xn + $IS_Yn * $IS_Yn)
            EndIf

            SetImageShift $isux $isuy
            #echo $icount, $rad, $theta
        EndLoop

        # theta = 90 - $theta

        rad = ( $rad + 0.0001 ) / $LayerFV
        rad_r = ( $rad_r + 0.0001 ) / $LayerFV

        IS_RAD1_Vh = $rad
        IniAng_Vh  = $theta

        Echo ---------------------------------------
        Echo in V
        Echo IS_RAD1_Vh = $rad      # Distance between each holes [um]
        Echo IniAng_Vh  = $theta    # Angle between X-axis of "Stage" and X-axis of "hole lattice" (or Y) [degree]
        Echo ---------------------------------------

        IS_RAD1_Rh = $rad_r
        IniAng_Rh  = $theta_r

        Echo ---------------------------------------
        Echo in R
        Echo IS_RAD1_Rh = $rad_r      # Distance between each holes [um]
        Echo IniAng_Rh  = $theta_r    # Angle between X-axis of "Stage" and X-axis of "hole lattice" (or Y) [degree]
        Echo ---------------------------------------

        # --- For vertical vector ---
        rad = $IS_RADIUS
        theta = $IS_ANGLE + 90
        Loop $iter_num icount
            GoToLowDoseArea V
            IS_X = $rad * cos ($theta)
            IS_Y = $rad * sin ($theta)
            ImageShiftByMicrons $IS_X $IS_Y 0 $BTtoIS
            Delay 1 sec
            
            If $grid_type != 0
               V
               AlignTo P
            EndIf

            ReportSpecimenShift IS_Xn IS_Yn

            echo  IS_Xn, IS_Yn = $IS_Xn, $IS_Yn
            theta = ATAN2 ( $IS_Yn $IS_Xn )
            rad  = sqrt ($IS_Xn * $IS_Xn + $IS_Yn * $IS_Yn)

            If $icount == $iter_num
                GoToLowDoseArea R
                Delay 1 sec
                ReportSpecimenShift IS_Xn IS_Yn
                theta_r = ATAN2 ( $IS_Yn $IS_Xn )
                rad_r  = sqrt ($IS_Xn * $IS_Xn + $IS_Yn * $IS_Yn)
            EndIf

            SetImageShift $isux $isuy
            #echo $icount, $rad, $theta
        EndLoop

        rad = ( $rad + 0.0001 ) / $LayerFV
        rad_r = ( $rad_r + 0.0001 ) / $LayerFV

        IS_RAD1_Vv = $rad
        IniAng_Vv  = $theta

        Echo ---------------------------------------
        Echo In V
        Echo IS_RAD1_Vv := $rad      # Distance between each holes [um]
        Echo IniAng_Vv  := $theta    # Angle between X-axis of "Stage" and X-axis of "hole lattice" (or Y) [degree]
        Echo ---------------------------------------

        IS_RAD1_Rv = $rad_r
        IniAng_Rv  = $theta_r

        Echo ---------------------------------------
        Echo In R
        Echo IS_RAD1_Rv := $rad_r      # Distance between each holes [um]
        Echo IniAng_Rv  := $theta_r    # Angle between X-axis of "Stage" and X-axis of "hole lattice" (or Y) [degree]
        Echo ---------------------------------------

        RunInShell echo $IS_RAD1_Vh $IniAng_Vh $IS_RAD1_Vv $IniAng_Vv > $path_to_navfileVectors_ViewMode.txt
        RunInShell echo $IS_RAD1_Rh $IniAng_Rh $IS_RAD1_Rv $IniAng_Rv > $path_to_navfileVectors_RecordMode.txt

        #====== For Multi-shot in a single hole ======
        RunInShell del $path_to_navfileMultiShot_Vectors_ViewMode.txt
        RunInShell del $path_to_navfileMultiShot_Vectors_RecordMode.txt

        SetImageShift 0 0 0 $BTtoIS
        If $NumberShots > 0
            ISV_X1 = 0
            ISV_Y1 = 0
            ISR_X1 = 0
            ISR_Y1 = 0
            
            Loop $NumberShots icount2
                GoToLowDoseArea V
                IS_ANGLE = ( $icount2 - 1 ) * 360 / $NumberShots
                IS_X = $IS_RAD2 * sin ( $IS_ANGLE )
                IS_Y = $IS_RAD2 * cos ( $IS_ANGLE )

                ImageShiftByMicrons $IS_X $IS_Y 1 $BTtoIS
                #V
                ReportSpecimenShift ISV_X2 ISV_Y2
                ISV_X = $ISV_X2 - $ISV_X1
                ISV_Y = $ISV_Y2 - $ISV_Y1
                RunInShell echo $ISV_X $ISV_Y >> $path_to_navfileMultiShot_Vectors_ViewMode.txt

                GoToLowDoseArea R
                ReportSpecimenShift ISR_X2 ISR_Y2
                ISR_X = $ISR_X2 - $ISR_X1
                ISR_Y = $ISR_Y2 - $ISR_Y1
                RunInShell echo $ISR_X $ISR_Y >> $path_to_navfileMultiShot_Vectors_RecordMode.txt

                ISV_X1 = $ISV_X2
                ISV_Y1 = $ISV_Y2
                ISR_X1 = $ISR_X2
                ISR_Y1 = $ISR_Y2

                Echo ---------------------------------------
                Echo IS for Shot$icount2
                Echo In V : $ISV_X [um], $ISV_Y [um]
                Echo In R : $ISR_X [um], $ISR_Y [um]
                Echo ---------------------------------------

                SetImageShift 0 0

            EndLoop
            
        EndIf

        # Reset View parameters
        GoToLowDoseArea V
        RestoreFocus
        RestoreLowDoseParams V
        SetImageShift 0 0 0 $BTtoIS

        # Reset camera params for View
        RestoreCameraSet V
    
    EndFunction

#################################################
EndMacro
Macro	15
ScriptName CheckMultiHole

    SetLowDoseMode 1

    # =================
    # Parameters
    # =================
    # Use SerialEM implemented multiple Record setting?
    use_multiR = 0 # 0:No, 1:Use only multi-shot setting, 2: Use both multiple-shot and multiple-hole setting

    #! If you use SerialEM implemented, below parameters are ignored.
    # 0:View, 1:Record
    LD_mode = 0

    # For multi-hole by IS(ImageShift)
    # 0:even, 1:odd
    PATTERN = 1 
    # For  odd LAYER => 1: 3x3, 2: 5x5, 3: 7x7, ...
    # For even LAYER => 1: 2x2, 2: 4x4, 3: 6x6, ...
    LAYER = 1
    
    # For multi-shot by IS(ImageShift) in one hole
    do_multishot = 0

    # Active BT comp
    BTtoIS = 1 # 0:No, 1:Yes
    use_custom_BTcomp = 0 # 0:No, 1:Yes
    BTtoOL = 0 # 0:No, 1:Yes

    # For tilt
    tilt_angle = 0 #[degree]

    # Other parameters
    check_coma = 0 #0:No, 1:Yes
    apply_defocus_offset = 0 #0:No, 1:Yes
    defocus_offset = 0 #[um]

    # =================
    # Initialize
    # =================

    Call EMProperties
    
    SetLowDoseMode 1

    ReportNavFile 1
    path_to_navfile = $repVal3

    RunInShell mkdir $path_to_navfileCheckMultiHole

    If $use_multiR != 0
        BTtoIS = 0
    EndIf

    If $LD_mode == 0
        GoToLowDoseArea V
        SaveFocus 
        ReadTextFile vector $path_to_navfileVectors_ViewMode.txt
        SetEucentricFocus
        ReportEnergyFilter 
        initailslit_status = $repVal3
        SetSlitIn 0
        UpdateLowDoseParams V

    ElseIf $LD_mode == 1
        GoToLowDoseArea R
        SaveFocus
        ReadTextFile vector $path_to_navfileVectors_RecordMode.txt

    EndIf

    If $do_multishot == 1

        If $LD_mode == 0
            Read2DTextFile multiVector $path_to_navfileMultiShot_Vectors_ViewMode.txt
        ElseIf $LD_mode == 1
            Read2DTextFile multiVector $path_to_navfileMultiShot_Vectors_RecordMode.txt
        EndIf

        NumberShots = $#multiVector
    
    EndIf

    # For multi-hole
    IS_RAD1_h = $vector[1]
    IniAng_h = $vector[2]
    IS_RAD1_v = $vector[3]
    IniAng_v = $vector[4]
    Echo ===> Horivontal : $IS_RAD1_h $IniAng_h 
    Echo ===> Vertical : $IS_RAD1_v $IniAng_v 

    TT = $tilt_angle

    If $use_custom_BTcomp == 1
        BTtoIS = 0
        ReadTextFile vector_BTtoIS $path_to_rootSPA\CalibMat_BTtoIS.txt
        xpx = $vector_BTtoIS[1]
        ypx = $vector_BTtoIS[2]
        xpy = $vector_BTtoIS[3]
        ypy = $vector_BTtoIS[4]
        
        # Setting for BTtoOL
        If ($BTtoOL == 1)
            ReadTextFile vector_BTtoOL $path_to_rootSPA\CalibMat_BTtoOL.txt
            BTx_to_OL = $vector_BTtoOL[1]
            BTy_to_OL = $vector_BTtoOL[2]
        EndIf
    EndIf

    # For temporal parameters
    IsVariableDefined NumberShots
    If $repVal1 == 0
        NumberShots = 0
        IS_RAD2     = 0.6
    EndIf
        

    # =================
    # Main 
    # =================

    ReportBeamTilt btx_base bty_base
    Echo ===> Initial BT ($btx_base, $bty_base)

    cnt = 0
    CallFunction Check_AquireMultiHoles

    ResetDefocus 
    If $LD_mode == 0
        SetSlitIn $initailslit_status
        UpdateLowDoseParams V
    EndIf

    SetBeamTilt $btx_base $bty_base
    ReportBeamTilt btx_last bty_last
    Echo ===> last BT ($btx_last $bty_last)

################################################################

    Function Check_AquireMultiHoles 0 0

        If $use_multiR == 2
            MultipleRecords -9 -9 -9 0 0
        Else
            If ( $PATTERN == 0 ) AND ( $LAYER > 0 ) 
                CallFunction Check_AquireMultiHoles_Even
            Else
                CallFunction Check_AquireMultiHoles_Odd
            EndIf
        EndIf

    EndFunction

################################################################

    Function Check_AquireHole 5 1 isux isuy btux btuy NumberShots

        If $do_multishot == 1

            Loop $NumberShots icount2

                IS_X = $multiVector[$icount2][1]
                IS_Y = $multiVector[$icount2][2]
                
                ### for tilt
                If $TT != 0
                    IS_Y = $IS_Y * cos ( $TT )
                EndIf
                ###

                ImageShiftByMicrons $IS_X $IS_Y 1 $BTtoIS

                cnt = $cnt + 1
                ReportSpecimenShift sp_x sp_y

                Echo ===> No. $cnt
                Echo ===> ImageShiftX:$sp_x, ImageShiftX:$sp_y

                # Set defocus for tilted stage
                CallFunction ChangeFocusForTilt $sp_y
                # Set active BT comp if desired
                CallFunction Custom_BTcomp $btx_base $bty_base

                If $LD_mode == 0

                    V
                    SaveToOtherFile A JPG NONE $path_to_navfileCheckMultiHole\View_$cnt.jpg

                ElseIf $LD_mode == 1

                    R
                    SaveToOtherFile A JPG NONE $path_to_navfileCheckMultiHole\Record_$cnt.jpg

                    If $check_coma == 1
                        FixComaByCTF 1 1
                    EndIf

                EndIf   

                # Reset IS to center of the hole when multishot finished
                If $icount2 == $NumberShots
                    SetImageShift $isux $isuy
                    SetBeamTilt $btux $btuy
                EndIf

            EndLoop

        Else

            cnt = $cnt + 1
            ReportSpecimenShift sp_x sp_y

            Echo ===> No. $cnt
            Echo ===> ImageShiftX:$sp_x, ImageShiftX:$sp_y
            
            # Set defocus for tilted stage
            CallFunction ChangeFocusForTilt $sp_y
            # Set active BT comp if desired
            CallFunction Custom_BTcomp $btx_base $bty_base

            If $LD_mode == 0

                V
                SaveToOtherFile A JPG NONE $path_to_navfileCheckMultiHole\View_$cnt.jpg

            ElseIf $LD_mode == 1

                If $use_multiR == 0
                    R
                ElseIf $use_multiR == 1
                    MultipleRecords -9 -9 -9 0 0
                EndIf
                SaveToOtherFile A JPG NONE $path_to_navfileCheckMultiHole\Record_$cnt.jpg
                #G -1 -1
                #ReportAutofocus measured_def
                #Echo ===> Measured Def : $measured_def

                If $check_coma == 1
                    FixComaByCTF 1 1
                EndIf

            EndIf            

        EndIf

    EndFunction
    
################################################################

    Function Check_AquireMultiHoles_Odd 0 0

        ReportTickTime
        start_ticks = $repVal1

        ReportDefocus origin_defocus 

        ReportImageShift isux1 isuy1
        ReportBeamTilt btux1 btuy1
        ReportBeamShift bsx1 bsy1

        # Aquire center position
        CallFunction Check_AquireHole $isux1 $isuy1 $btux1 $btuy1 $NumberShots
        # reset IS and BT. TODO These might not be needed.
        SetImageShift $isux1 $isuy1 
        SetBeamTilt $btux1 $btuy1
        SetBeamShift $bsx1 $bsy1

        If $LAYER > 0

            nx = 0
            ny = 0
            Vh_x = $IS_RAD1_h * cos ( $IniAng_h )
            Vh_y = $IS_RAD1_h * sin ( $IniAng_h )
            Vv_x = $IS_RAD1_v * cos ( $IniAng_v )
            Vv_y = $IS_RAD1_v * sin ( $IniAng_v )

            ### for tilt
            If $TT != 0
                Vh_y = $Vh_y * cos ( $TT )
                Vv_y = $Vv_y * cos ( $TT )
            EndIf
            ###

            Loop $LAYER idx

                # Move to next layer
                nx = $Vh_x
                ny = $Vh_y
                ImageShiftByMicrons $nx $ny 0 $BTtoIS
                ReportImageShift isux2 isuy2
                ReportBeamTilt btux2 btuy2
                CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots

                # Move around current layer
                side1 = 2 * $idx - 1
                Loop $side1
                    nx = $Vv_x
                    ny = $Vv_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side2 = 2 * $idx
                Loop $side2
                    nx = -1 * $Vh_x
                    ny = -1 * $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side3 = 2 * $idx
                Loop $side3
                    nx = -1 * $Vv_x
                    ny = -1 * $Vv_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side4 = 2 * $idx
                Loop $side4
                    nx = $Vh_x
                    ny = $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

            EndLoop

        EndIf

        SetDefocus $origin_defocus 

        SetImageShift 0 0
        SetBeamTilt $btux1 $btuy1
        SetBeamShift $bsx1 $bsy1

        ReportTickTime
        end_ticks = $repVal1
        elapsed_time = ($end_ticks - $start_ticks) / 60

        Echo ===> Done.
        Echo ===> Took $elapsed_time min

    EndFunction

################################################################

    Function Check_AquireMultiHoles_Even 0 0

        ReportTickTime
        start_ticks = $repVal1

        ReportDefocus origin_defocus 

        ReportImageShift isux1 isuy1
        ReportBeamTilt btux1 btuy1
        ReportBeamShift bsx1 bsy1

        If $LAYER > 0

            nx = 0
            ny = 0
            Vh_x = $IS_RAD1_h * cos ( $IniAng_h )
            Vh_y = $IS_RAD1_h * sin ( $IniAng_h )
            Vv_x = $IS_RAD1_v * cos ( $IniAng_v )
            Vv_y = $IS_RAD1_v * sin ( $IniAng_v )

            ### for tilt
            If $TT != 0
                Vh_y = $Vh_y * cos ( $TT )
                Vv_y = $Vv_y * cos ( $TT )
            EndIf
            ###

            # Move to first layer
            nx = $Vh_x / 2 + $Vv_x / 2
            ny = $Vh_y / 2 + $Vv_y / 2
            ImageShiftByMicrons $nx $ny 0 $BTtoIS
            ReportImageShift isux2 isuy2
            ReportBeamTilt btux2 btuy2
            CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots

            Loop $LAYER idx

                # Move to next layer
                If $idx > 1
                    nx = $Vh_x
                    ny = $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndIf

                # Move around current layer
                side1 = 2 * $idx - 2
                Loop $side1
                    nx = $Vv_x
                    ny = $Vv_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side2 = 2 * $idx - 1
                Loop $side2
                    nx = -1 * $Vh_x
                    ny = -1 * $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side3 = 2 * $idx - 1
                Loop $side3
                    nx = -1 * $Vv_x
                    ny = -1 * $Vv_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side4 = 2 * $idx - 1
                Loop $side4
                    nx = $Vh_x
                    ny = $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

            EndLoop

        EndIf

        SetDefocus $origin_defocus 

        SetImageShift 0 0
        SetBeamTilt $btux1 $btuy1
        SetBeamShift $bsx1 $bsy1

        ReportTickTime
        end_ticks = $repVal1
        elapsed_time = ($end_ticks - $start_ticks) / 60

        Echo ===> Done.
        Echo ===> Took $elapsed_time min

    EndFunction

###################################################

    Function Check_AquireMultiHoles_Hex 0 0

        ReportTickTime
        start_ticks = $repVal1

        ReportDefocus origin_defocus 

        ReportImageShift isux1 isuy1
        ReportBeamTilt btux1 btuy1
        ReportBeamShift bsx1 bsy1

        # Aquire center position
        CallFunction Check_AquireHole $isux1 $isuy1 $btux1 $btuy1 $NumberShots
        # reset IS and BT. TODO These might not be needed.
        SetImageShift $isux1 $isuy1 
        SetBeamTilt $btux1 $btuy1
        SetBeamShift $bsx1 $bsy1

        If $LAYER > 0

            nx = 0
            ny = 0

            V1_x = $IS_RAD1_1 * cos ( $IniAng_1 )
            V1_y = $IS_RAD1_1 * sin ( $IniAng_1 )
            V2_x = $IS_RAD1_2 * cos ( $IniAng_2 )
            V2_y = $IS_RAD1_2 * sin ( $IniAng_2 )
            V3_x = $IS_RAD1_3 * cos ( $IniAng_3 )
            V3_y = $IS_RAD1_3 * sin ( $IniAng_3 )

            ### for tilt
            If $TT != 0
                V1_y = $V1_y * cos ( $TT )
                V2_y = $V2_y * cos ( $TT )
                V3_y = $V3_y * cos ( $TT )
            EndIf
            ###

            Loop $LAYER idx

                # Move to next layer
                nx = $V1_x
                ny = $V1_y
                ImageShiftByMicrons $nx $ny 0 $BTtoIS
                ReportImageShift isux2 isuy2
                ReportBeamTilt btux2 btuy2
                CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots

                # Move around current layer
                side0 = $idx - 1
                Loop $side1
                    nx = $V2_x
                    ny = $V2_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side1 = $idx
                Loop $side1
                    nx = $V3_x
                    ny = $V3_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side2 = $idx
                Loop $side2
                    nx = -1 * $V1_x
                    ny = -1 * $V1_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side3 = $idx
                Loop $side3
                    nx = -1 * $V2_x
                    ny = -1 * $V2_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side4 = $idx
                Loop $side4
                    nx = -1 * $V3_x
                    ny = -1 * $V3_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side5 = $idx
                Loop $side5
                    nx = $V1_x
                    ny = $V1_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction Check_AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

            EndLoop

        EndIf

        SetDefocus $origin_defocus 

        SetImageShift 0 0
        SetBeamTilt $btux1 $btuy1
        SetBeamShift $bsx1 $bsy1

        ReportTickTime
        end_ticks = $repVal1
        elapsed_time = ($end_ticks - $start_ticks) / 60

        Echo ===> Done.
        Echo ===> Took $elapsed_time min

    EndFunction

################################################################


    Function ChangeFocusForTilt 1 0 sp_y

        If $TT != 0

            additional_defocus = -1 * $sp_y * tan ( $TT )

            setting_defocus = $origin_defocus + $additional_defocus

            If $apply_defocus_offset == 1
                SetDefocus $defocus_offset
            EndIf

            SetDefocus $setting_defocus

            Echo ===> Defocus : $setting_defocus [um]

        EndIf

    EndFunction

#######################################################

Function Custom_BTcomp 2 0 btx_base bty_base

    If $use_custom_BTcomp == 1
        # Get current IS
        ReportSpecimenShift ISx ISy
        # Calculate BT needed by IS
        BTx_IS = $ISx * $xpx + $ISy * $ypx 
        BTy_IS = $ISx * $xpy + $ISy * $ypy
        
        If $BTtoOL == 1
            # Get current defocus
            ReportDefocus OLval
            # Calculate BT needed by OL
            BTx_OL = $OLval * $BTx_to_OL
            BTy_OL = $OLval * $BTy_to_OL
        Else
            BTx_OL = 0
            BTy_OL = 0     
        EndIf

        BTx = $BTx_IS + $BTx_OL
        BTy = $BTy_IS + $BTy_OL

        # Apply BT for IS and OL
        SetBeamTilt ($btx_base + $BTx) ($bty_base + $BTy)
    EndIf

EndFunction

#######################################################
EndMacro
Macro	17
ScriptName AlignComaAndStig

    # ============================
    # Parameters
    # ============================
    # Set settling time
    settle_time = 20 #[sec]
    exp_time = 0.5 #[sec]

    do_ZLPrefine = 0 # 0:No, 1:Yes, 2: Popup dialog box
    area = R # Area for ZLP alignment
    do_beam_centering = 0 # 0:No, 1:Yes, 2: Popup dialog box

    # ============================
    # Main
    # ============================
    Call EMProperties

    GoToLowDoseArea R
    SetEucentricFocus
    SetImageShift 0 0

    ReportBeamTilt btx_init bty_init
    Echo ===> Initial Beam-Tilt ( BTx: $btx_init, BTy: $bty_init )

    focus_by_Z = 1
    max_focusZ_iter = 10
    target_defocus = -1.4

    SetExposure R $exp_time 
    SetBinning R 2
    SetDoseFracParams R 0 0 0 0 0
    
    CallFunction FocusByZ

    RestoreCameraSet R

    # === Refine ZLP =========================
    
    apply_refine_ZLP = 0
    If $do_ZLPrefine == 0
        apply_refine_ZLP = 0
    ElseIf $do_ZLPrefine == 1
        apply_refine_ZLP = 1
    ElseIf $do_ZLPrefine == 2
        YesNoBox Refine ZLP by FL?
         If $repVal1 == 1
             apply_refine_ZLP = 1
         EndIf
    EndIf

    If $apply_refine_ZLP == 1
        Echo ===> Apply Refine ZLP
        SetSlitIn 1
        UpdateLowDoseParams R 1
        ReportTickTime start_time_ZLP
        Call Parameters
        Call ZLPAlignByFL
        #CallFunction ZLPAlignByFL #::Core $ZLP_thld_ratio $do_coarse_search $ZLP_by_maxCount $use_fine_step $FLcmd $area
        ReportTickTime end_time_ZLP
        settle_time = $settle_time - ($end_time_ZLP $start_time_ZLP)
        If $settle_time <= 0
            settle_time = 0
        EndIf
        Echo ===> Waiting for settling $settle_time [sec]   
    Else
        Echo ===> Do not apply Refine ZLP
        Echo ===> Waiting for settling $settle_time [sec]
    EndIf

    # === AlignComa and Stig ===================

    Delay $settle_time sec

    SetExposure R 1.0
    SetBinning R 2
    SetDoseFracParams R 0 0 0 0 0

    Try
        FixComaByCTF
        FixAstigmatismByCTF
        FixAstigmatismByCTF
    Catch
        Echo ===> Fail to fit CTF. Skip coma-free alignment.
        Echo ===> Check rotational center manually and set larger BT% at  "Set CTF Coma-free Param" in "Focus/Tune".
    EndTry

    RestoreCameraSet R
    SetEucentricFocus

    ReportBeamTilt btx_aligned bty_aligned
    Echo ===> Beam-Tilt aligned to ( BTx: $btx_aligned, BTy: $bty_aligned )

    # Normalize LowDoseMode
    Echo ===> Normalizing LowDoseMode
    Loop 3
        GoToLowDoseArea V
        Delay 0.5 sec
        GoToLowDoseArea T
        Delay 0.5 sec
        GoToLowDoseArea F
        Delay 0.5 sec
        GoToLowDoseArea R
        Delay 0.5 sec
        GoToLowDoseArea V
        Delay 0.5 sec
    EndLoop

    Echo ===> Finish Coma-free alignment

    # === Beam Centering =================

    apply_beam_centering = 0
    If $do_beam_centering == 0
        apply_beam_centering = 0
    ElseIf $do_beam_centering == 1
        apply_beam_centering = 1
    ElseIf $do_beam_centering == 2
        YesNoBox Do Beam Centering?
        If $repVal1 == 1
            apply_beam_centering = 1
        EndIf
    EndIf

    If $apply_beam_centering == 1
        GoToLowDoseArea T
        T
        ReportMeanCounts A
        mean_count = $repVal1
        If $mean_count > 10
            ReportBeamShift init_beamX init_BeamY
            Echo ---> BeamCentering...
            T
            CenterBeamFromImage
            T
            ReportMeanCounts A
            mean_count = $repVal1

            If $mean_count < 10
                Echo ---> Beam Centering failed... Reset BeamShift.
                SetBeamShift $init_beamX $init_BeamY
            EndIf
        EndIf

        GoToLowDoseArea F
        GoToLowDoseArea R
        GoToLowDoseArea V
    EndIf


##########################################################

    Function FocusByZ

    backlash_z = 0.9

    #==============================
    # AutoFocus by Z
    #==============================

    Call EMProperties

    If $focus_by_Z == 1
        SetEucentricFocus

        Echo ---> Focusing by Z
        ReportStageXYZ init_x init_y init_z
        Loop $max_focusZ_iter iter

            CallFunction Funcs::WaitForRefilling
            Echo ------------------------------------------------
            Echo Autofocus by_Z iter $iter

            #ReportTargetDefocus target_defocus
            G -1 -1 #Measure defocus at R
            ReportAutofocus measured_defocus
            ReportStageXYZ setting_x setting_y setting_z

            Echo 
            Echo Target = $target_defocus [um]
            Echo Measured = $measured_defocus [um]

            diff = $target_defocus - $measured_defocus
            relax_z1 = ( $diff / ABS ($diff) ) * $backlash_z
            relax_z2 = -1 * ( $diff / ABS ($diff) ) * $backlash_z
            Echo Need change ---> $diff [um]
            range = ABS ($diff)
            If  $range <= 0.3
                Echo ===> Focusing by Z succeeded.
                Break
            ElseIf ( ABS ($measured_defocus) <= 0.0001 ) OR ( ABS ($measured_defocus) >= 200.0 )
                Echo ===> Autofocus by Z has failed, restore previous defocus value
                MoveStageTo $init_x $init_y $init_z
                Return
            Else
                setting_z = $setting_z + $diff + $relax_z1
                MoveStageTo $setting_x $setting_y $setting_z
                Echo ---> Move to $setting_z [um]
                Echo ---> Relaxing and Settling
                MoveStage 0 0 $relax_z2
                Delay 3 sec

                If $iter == $max_focusZ_iter
                    Echo ===> Focusing by Z looks failed. Z_byV again.
                    Call Z_byV
                    Return
                EndIf
            EndIf

        EndLoop

    EndIf

    EndFunction 

##########################################################
EndMacro
Macro	18
MacroName Parameters

    Echo ---> Calling Parameters ...

    #====================================================
    # Flashing, Dark Reference and Refill
    #====================================================
    # C-FEG flashing 
    FlashInterval := 12 * 3600 #[hrs x 3600]
    #FlashInterval := 0.05 * 3600 # 3min for test

    # Dark reference for K2/K3
    update_dark = 0 # 0:No, 1:Yes
    DarkRefInterval := 6 * 3600 #[hrs x 3600]

    # Refill
    flash_when_refill := 1
    darkRef_when_refill := 0
    delay_after_refill = 15 # [min]

    #====================================================
    # ZLP centering
    #====================================================
    do_refineZLP = 1 # 0:No, 1:Yes
    ZLPInterval  := 6 * 3600 #[hrs x 3600]
    area = R # Area for ZLP alignment

    ZLP_thld_ratio = 0.7 # 0:No, 1:Yes
    do_coarse_search = 0 # 0:No, 1:Yes
    use_fine_step = 1 # 0:No, 1:Yes
    ZLP_by_maxCount = 0 # 0:No, 1:Yes
    ZLP_when_flash = 0 # 0:No, 1:Yes

    #====================================================
    # Hole Alignment
    #====================================================
    # Template buffer for hole alignment
    template_buffer = T

    # Threshold for realigning to hole by stage
    maxholeshift = 0.15 * 1000 #[um x 1000] 

    # Maximum iteration for hole alignment
    max_align_iter = 10

    # Use IS for hole alignment?
    align_byIS = 1 # 0:No, 1:Yes

    # Use Piezo drive?
    use_Piezo = 0 # 0:No, 1:Yes
    piezo_threshold = 30 #[nm]

    # If yes, stop hole alignment after Focusing and Drift control. Cancels "realignBeforeRecord"
    stop_hole_realign = 0 # 0:No, 1:Yes

    #====================================================
    # Drift control
    #====================================================
    # Do drift control?
    do_drift_control = 1 # 0:No, 1:Yes
    use_focus_drift = 1 # 0:No, 1:Yes
    once_every_group = 1 # 0:No, 1:Yes
    drift_ctrl_when_tilt = 1 # 0:No, 1:Yes (If yes, always drift control after tilt)
    tilt_settling_time = 0 # [sec], additional settling time after stage tilt

    # Drift rate threshold [A/sec], only get used for skip = 0
    drift_crit = 4 #[A/sec]
    drift_shot = F

    # Move opposit direction while waiting drift ?
    resist_drift = 1 # 0:No, 1:Yes

    additional_settling_time = 0 #[sec]

    #====================================================
    # Focus
    #====================================================
    # Set TargetDefocus(TD) range and step.
    TD_low  = -0.8 #[um]
    TD_high = -1.5 #[um]
    TD_step = 0.1 #[um]

    # Autofocus error
    focus_error = 0.2 #[um] #0.2

    # When the stage moves following distance, then autofocus.
    stageX_limit_for_focus = 50 #[um]
    stageY_limit_for_focus = 50 #[um]
    stageZ_limit_for_focus = 5 #[um]

    # Define, how often the microscope should focus: 
    # focusEachHole := 0   means to keep focus target constant and focus always.
    # focusEachHole := 1   means to increment focus target and focus always.
    # focusEachHole := 5   means to increment focus target and focus only every 5th image.
    focusEachHole := 0

    # Irradiation for focus
    irradiation_time = 0 #[sec]

    # Maximum iteration for auto focus
    max_focusZ_iter = 10
    max_focus_iter = 5

    # Skip BeamCentering when auto focus is NOT performed
    skip_BeamCentering = 1 # 0:No, 1:Yes
    beamCentering_afterFocus = 0 # 0:No, 1:Yes

    # Do focusing by Z?
    focus_by_Z = 0 # 0:No, 1:Yes

    # Do focusing by OL?
    focus_by_OL = 1 # 0:No, 1:Yes

    # Update group Z after Focus?
    updata_Z_afterFocus = 1 # 0:No, 1:Yes

    # Do focusing by CTFFIND? (Under development)
    focus_by_ctf = 0 # 0:No, 1:Yes

    # Do Auto Coma Free after focusing? (Under developmental)
    correct_coma = 0 # 0:No, 1:Yes
    correct_stig = 0 # 0:No, 1:Yes

    # Z_byV every square?
    do_Z_byV = 0 # 0:No, 1:Yes

    # Z focusing settling time
    z_settle_time = 3 #[sec]

    # Maximum erro of Z height change from the initial point of the group
    #safetyZ = 40 # [um]

    #====================================================
    # Record
    #====================================================
    # --- Use SerialEM based multiple Record? ---
    # Recommendation is 0 or 1. Use script-based multiple-hole setting and do NOT use script-based multi-shot
    use_multiR = 0 # 0:No, 1:Use only multi-shot setting, 2: Use both multiple-shot and multiple-hole setting

    # --- Use Script based multiple Record? ---
    # For script based multiple-hole 
    PATTERN = 1  # 0:even, 1:odd
    LAYER = 3  # odd LAYER => 1: 3x3, 2: 5x5, 3: 7x7, ...    # even LAYER => 1: 2x2, 2: 4x4, 3: 6x6, ... # 0 => single shot

    realignBeforeRecord = 1 #0: No, 1:Yes

    # For script based multi-shot
    # Recommendation is 0
    do_multishot = 0 # 0:No, 1:Yes

    # If set to 1, IS is always 0 at center of multi-hole pattern
    zero_IS = 0 # 0:No, 1:Yes

    # Measure thickness? (Only availavle when PATTERN = 1)
    measure_thickness = 0 # 0:No, 1:Yes

    # For Acceptance test
    R_delay = 0 # [sec]

    #====================================================
    # Active Beam-Tilt Compensation
    #====================================================
    # Perform BT(BeamTilt) compensation? (Use SerialEM implemented)
    BTtoIS = 1 # 0:No, 1:Yes
    
    # Use your own data? (Scrip implemented). If Yes, SerialEM implemented BT compensation is neglected.
    use_custom_BTcomp = 0 # 0:No, 1:Yes
    BTtoOL = 0 #0:No, 1:Yes

    #====================================================
    # Tilt (Under development)
    #====================================================
    # Set TargetTilt (TT) and frequency 
    do_tilt = 0 # 0:No, 1:Yes
    TT_list = { 0 10 20 30 40 } # [degree]
    TT_freq = { 0 1 1 0 1 }
    changeTT_byFlashing = 1 # 0:No 1:Yes

    focus_before_tilt = 1 # 0:No 1:Yes, Focusing by Z "before tilting".
    use_eucentric_height = 0 # 0:No 1:Yes, Adjust Z to eucentric height "before tilting".

    max_track_shift = 3000 #[nm]
    stop_OLfocusing = 0 # 0:No 1:Yes, Stop focusing by OL "after tilting" anyway (For NIH claiming)

    #====================================================
    # Phase Plate Setting
    #====================================================
    use_PhasePlate = 0 # 0:No, 1:Yes
    use_ConditionSetup = 1 # 0:No, 1:Yes, Use setting from "Phase Plate Condtioning Setup dialog"

    # If use_ConditionSetup is 1, below setting are ignored.
    PP_interval_images = 60 #[images]
    PP_drift_wait_time = 3 #[min]
    PP_charge_up_time = 1 #[min]

    #====================================================
    # Display Setting
    #====================================================  
    stop_display_R = 0 # 0:display, 1:stop display
 
    # Just for "EarlyReturnNextShot".
    # This is for no return for Record Frame exposure for a K2/K3 camera.
    noReturn = -1
 
    # How often do you want to show images?
    # If you want to see the recorded image for every movie (DisplayOnly=1) or 
    # only for every 10th movie (DisplayOnly=10) or even more rarely:
    DisplayOnly = 1000

    #====================================================
    # LogImage Setting
    #====================================================  
    save_V = 1 # 0:No, 1:Yes
    save_T = 1 # 0:No, 1:Yes
    save_F = 1 # 0:No, 1:Yes
    save_R = 1 # 0:No, 1:Yes

    #====================================================
    # Do not have to touch below
    #====================================================
    parameters_type = 1
EndMacro
Macro	20
ScriptName TakeHoleTemplate

    # ============================
    # Parameters
    # ============================

    exposure_time = 2

    # ============================
    # Main
    # ============================
    SuppressReports

    Echo ============= Start TakeHoleTemplate =============
    
    # Set View mode
    SetLowDoseMode 1
    GoToLowDoseArea V

    # Set file path
    ReportNavFile 1
    path_to_navfile = $repVal3
    file_name @= hole_template.mrc
    file_path @= $path_to_navfile$file_name

    # Set camera params
    SetExposure V $exposure_time

    # Take View image
    V
    Copy A T

    # Save file
    SaveToOtherFile A MRC NONE $file_path

    # Reset camera params
    RestoreCameraSet V

    # Result
    Echo ===> Finish TakeHoleTemplate.
    Echo ===> Hole template image was saved as $file_path

    # Set ImageShift 0 0
    SetImageShift 0 0
    Echo ===> ImageShift is set to 0 0
EndMacro
Macro	21
Macroname Initializer

    #====================================================
    # Grid type
    #====================================================
    Try
        ReadTextFile grid_type $path_to_navfileGrid_type.txt
        Echo ===> grid_type is $grid_type
    Catch
        grid_type = 1
        Echo ===> Grid_type.txt does not exist. Use grid_type of 1 (Quantifoil).
    EndTry

    If $grid_type == 0
        Echo ===> Use lacey grid workflow.
    ElseIf $grid_type == 1
        Echo ===> Use Quantifoil/UltrAuFoil workflow.
    Else
        Echo ===> Grid type $grid_type has not yet determined.
    EndIf

    #====================================================
    # Initial setting
    #====================================================
    # parameters_type 1: Parameters, 2: Parameters_Screening
    # For Parameters
    If $parameters_type == 1
        IsVariableDefined Initializer
        If $repVal1 == 0
            # Set initial time
            ReportTickTime
            initial_ticks := $RepVal1

            # Set focus counter
            FOCUSCOUNTER := 0
            focus_problem_counter := 0

            # Set log file
            SaveLogOpenNew LOGFILE.log

            # Initialize DISPLAYCOUNTER (see below)
            DISPLAYCOUNTER := $DisplayOnly

            # Set Initializer
            Initializer := 1

            # Initialing stage
            LASTX := 10000
            LASTY := 10000
            LASTZ := 10000

            # Focus log initialize
            focused = 0

            # Set standard focus
            ReportLowDose
            If $repVal2 == 0 # V
                GoToLowDoseArea T
                GoToLowDoseArea F
            ElseIf $repVal2 == 2 # T
                GoToLowDoseArea F
            EndIf
            GoToLowDoseArea R
            SetEucentricFocus

            # Reset tilt
            TiltTo (-1 * $backlash_tilt)
            TiltTo $backlash_tilt
            TiltTo 0

            #LongOperation FF 0

            # Create Directory
            log_dir := $path_to_navfileLogImage
            RunInShell mkdir $log_dir
            log_dirV :@= $log_dir\View
            log_dirT :@= $log_dir\Trial
            log_dirF :@= $log_dir\Focus
            log_dirR :@= $log_dir\Record
            RunInShell mkdir $log_dirV
            RunInShell mkdir $log_dirT
            RunInShell mkdir $log_dirF
            RunInShell mkdir $log_dirR
            acq_count := -1

            # Read hole image
            If $grid_type != 0
                CallFunction Funcs::ReadHoleImage
            EndIf
        EndIf

    # For Parameters_Screening
    ElseIf $parameters_type == 2
        IsVariableDefined Initializer_Screening
        If $repVal1 == 0
            # Set initial time
            ReportTickTime
            initial_ticks := $RepVal1

            # Set focus counter
            FOCUSCOUNTER := 0
            focus_problem_counter := 0

            # Set log file
            SaveLogOpenNew LOGFILE.log

            # Initialize DISPLAYCOUNTER (see below)
            DISPLAYCOUNTER := $DisplayOnly

            # Set Initializer
            Initializer_Screening := 1
            #Initializer  := 0 #set FM

            # Initialing stage
            LASTX := 10000
            LASTY := 10000
            LASTZ := 10000

            # Focus log initialize
            focused = 0

            # Set standard focus
            GoToLowDoseArea R
            SetEucentricFocus

            # Reset tilt
            TiltTo (-1 * $backlash_tilt)
            TiltTo $backlash_tilt
            TiltTo 0

            #LongOperation FF 0

            # Create Directory
            log_dir := $path_to_navfileLogImage_Screening
            RunInShell mkdir $log_dir
            log_dirV :@= $log_dir\View
            log_dirT :@= $log_dir\Trial
            log_dirF :@= $log_dir\Focus
            log_dirR :@= $log_dir\Record
            RunInShell mkdir $log_dirV
            RunInShell mkdir $log_dirT
            RunInShell mkdir $log_dirF
            RunInShell mkdir $log_dirR
            acq_count := -1

            # Read hole image
            If $grid_type != 0
                CallFunction Funcs::ReadHoleImage
            EndIf
        EndIf

    # For something other...
    Else
        Echo ===> Please set correct parameters_type value.
    EndIf

    # Open Gun Valve
    ReportColumnOrGunValve 
    If $repVal1 == 0
       SetColumnOrGunValve 1
    EndIf

    SaveLog

    #====================================================
    # Display counting
    #====================================================
    # Mainly used for maintainance. Display every few hours only,
    If $parameters_type == 1
        If $Initializer != 0 

            Initializer := $Initializer - 1
            DISPLAYCOUNTER := $DISPLAYCOUNTER - 1
            Echo Displaying in $DISPLAYCOUNTER item images.

            If $DISPLAYCOUNTER < 1
                # show images & reset DISPLAYCONTER
                DisplayReturn := $noReturn
                DISPLAYCOUNTER := $DisplayOnly
            Else
                # NOT show images
                DisplayReturn := 0
            EndIf

        Else
            # Reset Initializer
            Initializer := 1

        EndIf
    ElseIf $parameters_type == 2
        If $Initializer_Screening != 0

            Initializer_Screening := $Initializer_Screening - 1
            DISPLAYCOUNTER := $DISPLAYCOUNTER - 1
            Echo Displaying in $DISPLAYCOUNTER item images.

            If $DISPLAYCOUNTER < 1
                # show images & reset DISPLAYCONTER
                DisplayReturn := $noReturn
                DISPLAYCOUNTER := $DisplayOnly
            Else
                # NOT show images
                DisplayReturn := 0
            EndIf

        Else
            # Reset Initializer_Screening
            Initializer_Screening := 1

        EndIf
    Else
        Echo ===> Please set correct parameters_type value.
    EndIf

    #====================================================
    # Initialize Flashing, LN2 Refill, ZLP Centering, Dark ref update settings
    #====================================================
    IsVariableDefined flashed
    If $repVal1 == 0
        flashed := 0
    EndIf

    IsVariableDefined refilled 
    If $repVal1 == 0
        refilled := 0
    EndIf

    IsVariableDefined ZLP_aligned 
    If $repVal1 == 0
        ZLP_aligned := 0
    EndIf

    IsVariableDefined dark_updated 
    If $repVal1 == 0
        dark_updated := 0
    EndIf

    If $refilled == 1
        # Change TT by refilling
        If $do_tilt == 1
            CallFunction Funcs::CycleTargetTilt
        Else
            TT = 0
        EndIf
    EndIf

    refilled := 0

    #====================================================
    # Z adjust
    #====================================================
    If ($focus_by_Z == 1) OR ($do_tilt == 1)
        GoToLowDoseArea R
        SetEucentricFocus
    EndIf

    # ---- For CorrectWrongZ Function ----
    IsVariableDefined groupZ
    If $repVal1 == 0
        ReportStageXYZ 
        groupZ := $repVal3
    EndIf
    # ---------------------------------------------

    #====================================================
    # Reset piezo
    #====================================================
    # Reset Piezo
    If $use_Piezo == 1
       CallFunction Funcs::RelaxPiezoXY 
       CallFunction Funcs::CustomMovePiezoXY 0 0
    Endif 


    #====================================================
    # Other paremeter initialize
    #====================================================
    # Initial BeamTilt value
    ReportBeamTilt btx_base bty_base
    Echo ===> Initial Beam-Tilt value ( BTx: $btx_base, BTy: $bty_base )

    # Focus
    TD_low     = -1 * ABS ($TD_low)
    TD_high    = -1 * ABS ($TD_high)
    TD_step    = ABS ($TD_step)

    If $TD_low <= $TD_high
        TD_high = $TD_low - 0.5
    EndIf

    focus_th_low = $TD_low + 0.1 #[um]
    focus_th_high = $TD_high - 0.1 #[um]

    focused = 0

    If $focus_by_OL == 1
        focus_by_Z = 0
    EndIf

    # Acquired image initialize
    IsVariableDefined acquired_image_num
    If $repVal1 == 0
        acquired_image_num := 0
    EndIf

    # Set layer for lacey grid.
    If $grid_type == 0
        ReadTextFile lacey_layer $path_to_navfileLacey_layer.txt
        LAYER = $lacey_layer 
        Echo ===> Set lacey grid parameter (Layer: $lacey_layer)
    EndIf
    
    # Record. Multi-hole setting.
    If $LAYER != 0
        ReadTextFile vector $path_to_navfileVectors_RecordMode.txt
        IS_RAD1_h   = $vector[1]
        IniAng_h    = $vector[2]
        IS_RAD1_v   = $vector[3]
        IniAng_v    = $vector[4]
    EndIf

    # Record. Multi-shot setting.
    NumberShots = 0
    If $do_multishot == 1
        Read2DTextFile multiVector $path_to_navfileMultiShot_Vectors_RecordMode.txt
        NumberShots = $#multiVector 
    EndIf

    If $use_multiR != 0
        BTtoIS = 0
    EndIf

    # Measure thickness
    If $PATTERN != 1
        measure_thickness = 0
    EndIf

    # Tilt
    If $do_tilt == 1
        If $#TT_list != $#TT_freq
            Echo !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
            Echo Number of elements of target tilt list and frequency list has mismatch!
            Echo Tilt will not be performed.
            Echo !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
            do_tilt = 0
        # Reset focus setting for tilted data collection
        Else
            focus_by_Z = 1
            focus_by_OL = 0
            
        EndIf

    EndIf

    # StageProperty
    If $use_apprV == 1
        CallFunction Funcs::CustomMoveStage 0 0
    EndIf

    acq_count := $acq_count + 1

    #====================================================
    # Read own calibration data for Active BeamTilt Compensation
    #====================================================
    If $use_custom_BTcomp == 1
        BTtoIS = 0
        ReadTextFile vector_BTtoIS $path_to_rootSPA\Data\CalibMat_BTtoIS.txt
        xpx = $vector_BTtoIS[1]
        ypx = $vector_BTtoIS[2]
        xpy = $vector_BTtoIS[3]
        ypy = $vector_BTtoIS[4]
        
        # Setting for BTtoOL
        If ($BTtoOL == 1)
            ReadTextFile vector_BTtoOL $path_to_rootSPA\Data\CalibMat_BTtoOL.txt
            BTx_to_OL = $vector_BTtoOL[1]
            BTy_to_OL = $vector_BTtoOL[2]
        EndIf
    EndIf

    #====================================================
    # Current info
    #====================================================
    ReportNavItem
    current_item := $repVal1
    item_label = $navLabel
    
    NavIndexItemDrawnOn $current_item
    map_item = $repVal1
    ReportOtherItem $map_item
    map_label = $navLabel

    ReportStageXYZ
    curr_x = $repVal1
    curr_y = $repVal2
    curr_z = $repVal3
    
    #====================================================
    # Flags
    #====================================================
    run_from_datacollection = 1
EndMacro
Macro	22
MacroName SPADataCollection

# Prepare for Cryo-ARM200 in NIH
# Sep. 12, 2019
#
# Modified for Cryo-ARM200 in NIH
# Feb. 10, 2020
#
# Modified for Cryo-ARM200 in NIH
# Aug. 25, 2020
#
# Modified for Cryo-ARM300 in Scripps
# Oct. 20, 2020
#
# Modified for Cryo-ARM200 in NIH and Cryo-ARM300 in Scripps
# Nov. 10, 2020
#
# Modified for Cryo-ARM200 in NIH and Cryo-ARM300 in Scripps
# Jan. 12, 2021
#
# Modified for Cryo-ARM300 in Scripps
# May. 11, 2021

    #======#
    # Main #
    #======#

    Echo =============== Starting SPADataCollection Macro =============== 

    SuppressReports
    SetLowDoseMode 1

    ReportTickTime
    ticks0 = $repVal1

    Call EMProperties

    Call Parameters

    Call Initializer

    CallFunction FlashingMonitor

    #CallFunction DarkRefMonitor

    CallFunction UpdateGroupZ

    CallFunction AlignToHole

    CallFunction Controller::FocusControl

    CallFunction Controller::DriftControl

    CallFunction Controller::TiltControl

    CallFunction RealignToHole

    If $zero_IS == 1
        SetImageShift 0 0
    EndIf

    CallFunction ReportStatus

    CallFunction AquireMultiHoles

    CallFunction ResetStatus $btx_base $bty_base

    CallFunction MeasureThickness

    CallFunction Controller::ZLPControll

    CallFunction PhasePlateMonitor

    CallFunction ReportProgress $ticks0

    # When fininshing data collection, close valve and emission off.
    ReportNextNavAcqItem 
    If $repVal1 == -1
       SetFEGEmissionState 0
       SetColumnOrGunValve 0
    Endif 

#######################################################

    Function FlashingMonitor 0 0

        IsVariableDefined last_flashing
        If $repVal1 == 0
            last_flashing := $initial_ticks
        EndIf
        
        ReportTickTime current_ticks
        flashing_elapsed_ticks = $current_ticks - $last_flashing

        If $flashing_elapsed_ticks >= $FlashInterval
            echo ---> Flashing ...
            LongOperation FF 0 # Flashing
            ReportTickTime
            last_flashing := $repVal1
            flashed := 1
        Else
            flashed := 0
        EndIf

        # For tilted data collection with Cold-FEG
        If $do_tilt == 1
            CallFunction Funcs::CycleTargetTilt
        Else
            TT = 0
        EndIf

    EndFunction

#######################################################

    Function DarkRefMonitor 0 0

        If $update_dark == 1
            IsVariableDefined last_dark
            If $repVal1 == 0
                last_dark := $initial_ticks
            EndIf

            ReportTickTime current_ticks
            darkRef_elapsed_ticks = $current_ticks - $last_dark

            If $darkRef_elapsed_ticks >= $DarkRefInterval
                echo ---> Dark ref updating ...
                LongOperation Da 0 # DarkRef
                ReportTickTime
                last_dark := $repVal1
                dark_updated := 1
            Else
                dark_updated := 0
            EndIf
        Else
            dark_updated := 0
        EndIf

    EndFunction
    
#######################################################

    Function InitialRelax 0 0

        IsVariableDefined old_x
        
        If $repVal1 == 1

            shift_x = $curr_x - $old_x
            shift_y = $curr_y - $old_y
            shift_z = $curr_z - $old_z

            If ABS ( $shift_x ) < 10 #[um]
                signX = 0
            Else
                signX = $shift_x / ABS ( $shift_x )
            Endif 

            If ABS ( $shift_y ) < 10 #[um]
                signY = 0
            Else 
                signY = $shift_y / ABS ( $shift_y )
            Endif 

            Loop 3 idx
                Echo ---> iter $idx
                moveX = -1 * $signX * $backlash_x
                moveY = -1 * $signY * $backlash_y
                Echo ---> Relaxing by moving stage $moveX $moveY ...
                MoveStage $moveX $moveY
            EndLoop

        EndIf

    EndFunction

#######################################################

    Function UpdateGroupZ 0 0

        process @= UpdateGroupZ
        ReportTickTime
        start_ticks = $repVal1

        group_FLAG := 0

        CallFunction Funcs::WaitForRefilling

        IsVariableDefined CURRENTGROUP
        If $repval1 == 0
            CURRENTGROUP := 0
            accum_item_shiftX := 0
            accum_item_shiftY := 0
        EndIf

        ReportGroupStatus
        groupID = $repval2

        If $CURRENTGROUP != $groupID

            # Reset Item coordinate
            Echo ===> Reset Item coordinates
            ShiftItemsByMicrons $accum_item_shiftX $accum_item_shiftY
            accum_item_shiftX := 0
            accum_item_shiftY := 0
            ReportNavItem 
            item_id =  $repVal1
            Echo ===> Move to Item $item_id
            MoveToNavItem $item_id
            CallFunction Funcs::CustomMoveStage 0 0

            # Update Z hight
            CURRENTGROUP := $groupID
            If ( $do_Z_byV == 1 )
                Echo ----> Update Z-height
                Call Z_byV
                ReportStageXYZ
                groupZ := $repVal3
                UpdateGroupZ
                UpdateItemZ
                # Drift control in View
                CallFunction Funcs::WaitForDrift 5 V
            EndIf
            group_FLAG := 1

        EndIf
        
        ReportTickTime
        end_ticks = $repVal1
        CallFunction Funcs::ElapsedTimeMonitor $start_ticks $end_ticks $process

    EndFunction

#######################################################

    Function CorrectWrongZ

       ReportStageXYZ currentX currentY currentZ

       diffZ = $currentZ - $groupZ
       If ABS ($diffZ) > $safetyZ
           # === Normalize LowDoseMode ===
           GoToLowDoseArea V
           GoToLowDoseArea T
           GoToLowDoseArea F
           GoToLowDoseArea R

           # === Set Eucentric Focus ===
           SetEucentricFocus

           # === Move to initial group Z heght and adjust Z again ===
           GoToLowDoseArea V
           MoveStageTo currentX currentY $groupZ
           Call Z_byV
           UpdateGroupZ
           UpdateItemZ           
       EndIf

    EndFunction

#######################################################

    Function AlignToHole 0 0

        GoToLowDoseArea V

        # Skip in case of lacey grid
        If $grid_type == 0
            Echo ===> Skip hole alignment.
            Return
        EndIf

        process @= AlignToHole
        ReportTickTime
        start_ticks = $repVal1

        # Initial settings
        maxholeshift_tmp = $maxholeshift
        align_byIS_tmp = $align_byIS

        # Reset Piezo
        If $use_Piezo == 1
            CallFunction RelaxPiezoXY 
            CallFunction CustomMovePiezoXY 0 0
        EndIf

        If ( $do_tilt == 1 ) AND ( $TT != 0 )
            maxholeshift = 400
            align_byIS = 1
        EndIf

        ReportImageShift IS_X0 IS_Y0
        ReportSpecimenShift sp_x0 sp_y0
        align_count = 0
        align_err_count = 0

        # Start Hole Aliginment
        CallFunction Funcs::WaitForRefilling
        Echo ===> Running AlignToHole ...
        V
        If $save_V == 1
            SaveToOtherFile A JPG NONE $log_dirV\View_beforeAlign_Item$item_label_Square$map_label_$acq_count.jpg
        EndIf

        # Report Shift
        AlignTo $template_buffer
        align_count = $align_count + 1
        ReportAlignShift # Shift on specimen X/Y axis (TiltX/Y) 
        dx = $RepVal5 #[nm]
        dy = $RepVal6 * -1 #[nm]
        Echo Hole align iter $align_count
        Echo Shift ---> X:$dx [nm] Y:$dy [nm]
        Echo -------------------
        holeshift = sqrt ( $dx * $dx + $dy * $dy )

        If $holeshift >= 500
            align_err_count = $align_err_count + 1
        EndIf

        # Initial shift for item realign
        ReportSpecimenShift sp_x1 sp_y1
        diff_x = $sp_x1 - $sp_x0
        diff_y = $sp_y0 - $sp_y1 # ? Y is opposite direction..

        # Iterate Hole Alignment
        Loop $max_align_iter
            CallFunction Funcs::WaitForRefilling
            # Success
            If $holeshift < $maxholeshift

                # Realign items
                ShiftItemsByMicrons $diff_x $diff_y
                # Accumulated item shift used for reset when group is changed.
                accum_item_shiftX := $accum_item_shiftX - $diff_x
                accum_item_shiftY := $accum_item_shiftY - $diff_y

                If $align_byIS == 0
                    SetImageShift 0 0 # IS is anyway 8000 8000 for stage shift
                EndIf

                If $use_Piezo == 1
                    Loop 10
                        V
                        Echo ===> Piezo drive
                        AlignTo $template_buffer
                        ReportSpecimenShift dx dy
                        SetImageShift 0 0 0 0
                        shift_x = $dx
                        shift_y = $dy * -1 
                        Echo ===> Need shift $shift_x $shift_y
                        CallFunction Funcs::MovePiezoXY_byMicron $shift_x $shift_y
                        holeshift = sqrt ( $dx * $dx + $dy * $dy ) * 1000 #[nm]
                        If $holeshift  <= $piezo_threshold
                            Break
                        EndIf
                    EndLoop
                    
                EndIf

                Echo ===> hole align finished
                If $save_V == 1
                    SaveToOtherFile A JPG NONE $log_dirV\View_afterAlign_Item$item_label_Square$map_label_$acq_count.jpg
                EndIf

                maxholeshift = $maxholeshift_tmp
                align_byIS = $align_byIS_tmp

                Break
            # Iterate
            Else
                ResetImageShift 2 0.05 # relax stage
                ReportSpecimenShift sp_x0 sp_y0
                If $use_apprV == 1
                    CallFunction Funcs::CustomMoveStage 0 0
                EndIF
                V
                AlignTo $template_buffer
                align_count = $align_count + 1
                ReportAlignShift
                dx = $RepVal5 # Shift on specimen X axis [nm]
                dy = $RepVal6 * -1 # Shift on specimen Y axis [nm]
                holeshift = sqrt ( $dx * $dx + $dy * $dy )
                Echo Hole align iter $align_count
                Echo Shift ---> X:$dx [nm] Y:$dy [nm]
                Echo -------------------

                If $holeshift >= 500
                    align_err_count = $align_err_count + 1
                EndIf

                # Shift for item realign
                ReportSpecimenShift sp_x1 sp_y1
                diff_x = $diff_x + ( $sp_x1 - $sp_x0 )
                diff_y = $diff_y + ( $sp_y0 - $sp_y1 ) # ? Y is opposite direction..

                # Failed...
                If $align_err_count >= 3
                    SetImageShift 0 0
                    skip_message @= "Skipped. Hole alignment failure."
                    CallFunction Funcs::AnnotateSkipItem $skip_message
                    SkipAcquiringNavItem
                    Exit
                EndIf

            EndIf

        EndLoop

        ReportTickTime
        end_ticks = $repVal1
        CallFunction Funcs::ElapsedTimeMonitor $start_ticks $end_ticks $process
    
    EndFunction

#######################################################

    Function RealignToHole 0 0

        # Skip in case of lacey grid
        If $grid_type == 0
            Echo ===> Skip hole alignment.
            Return
        EndIf

        If ( ($focus_by_Z == 1) AND ($group_FLAG == 1) ) OR ($realignBeforeRecord == 1)
            If $stop_hole_realign == 1
                Echo ===> Stop hole alignment after Focusing or Drift control
            ElseIf $do_tilt == 0
                Echo ===> Performs hole realign.
                GoToLowDoseArea R
                CallFunction AlignToHole
                GoToLowDoseArea T
                GoToLowDoseArea F
                GoToLowDoseArea R
                SetEucentricFocus
            EndIF
        EndIf

    EndFunction

#######################################################

    Function AquireHole 6 0 isux isuy btux btuy NumberShots DisplayReturn

        If $NumberShots > 0

            Loop $NumberShots icount2

                IS_X = $multiVector[$icount2][1]
                IS_Y = $multiVector[$icount2][2]
                
                ### for tilt
                If $TT != 0
                    IS_Y = $IS_Y * cos ( $TT )
                EndIf
                ###

                ImageShiftByMicrons $IS_X $IS_Y 0 $BTtoIS

                CallFunction Funcs::WaitForRefilling                

                ReportSpecimenShift sp_x sp_y

                # Set defocus for tilted stage
                CallFunction ChangeFocusForTilt $sp_y
                # Set active BT comp if desired
                CallFunction Custom_BTcomp $btx_base $bty_base

                If $stop_display_R == 1
                    #EarlyReturnNextShot 0
                    Delay $R_delay sec
                    R
                Else
                    #EarlyReturnNextShot $DisplayReturn
                    Delay $R_delay sec
                    R
                EndIf

                acquired_image_num := $acquired_image_num + 1

                sp_x_round = ROUND $sp_x 2
                sp_y_round = ROUND $sp_y 2
                Echo ===> ImageShiftX:$sp_x_round [um], ImageShiftY:$sp_y_round [um]

                SetImageShift $isux $isuy
                SetBeamTilt $btux $btuy

            EndLoop

        Else

            CallFunction Funcs::WaitForRefilling

            ReportSpecimenShift sp_x sp_y

            # Set defocus for tilted stage
            CallFunction ChangeFocusForTilt $sp_y
            # Set active BT comp if desired
            CallFunction Custom_BTcomp $btx_base $bty_base

            If $stop_display_R == 1
                #EarlyReturnNextShot 0
            Else
                #EarlyReturnNextShot $DisplayReturn
            EndIf

            Delay $R_delay sec
            If $use_multiR == 0
                R
            ElseIf $use_multiR == 1
                MultipleRecords -9 -9 -9 0.1
            EndIf

            acquired_image_num := $acquired_image_num + 1

            sp_x_round = ROUND $sp_x 2
            sp_y_round = ROUND $sp_y 2
            Echo ===> ImageShiftX:$sp_x_round [um], ImageShiftY:$sp_y_round [um]

        EndIf


    EndFunction
    
################################################################

    Function AquireMultiHoles 0 0

        process @= AquireMultiHoles
        ReportTickTime
        start_ticks = $repVal1

        If $use_multiR == 2
            MultipleRecords -9 -9 -9 0.01
        Else
            If ( $PATTERN == 0 ) AND ( $LAYER > 0 ) 
                CallFunction AquireMultiHoles_Even
            Else
                CallFunction AquireMultiHoles_Odd
            EndIf
        EndIf

        Echo -----> Done
        ReportTickTime
        end_ticks = $repVal1
        CallFunction Funcs::ElapsedTimeMonitor $start_ticks $end_ticks $process

    EndFunction

#######################################################

    Function AquireMultiHoles_Odd 0 0
        
        GoToLowDoseArea R

        ReportDefocus origin_defocus 

        ReportImageShift isux1 isuy1
        ReportBeamTilt btux1 btuy1
        ReportBeamShift bsx1 bsy1

        CallFunction AquireHole $isux1 $isuy1 $btux1 $btuy1 $NumberShots $DisplayReturn
        If $save_R == 1
            SaveToOtherFile A JPG NONE $log_dirR\Record_Item$item_label_Square$map_label_$acq_count_01.jpg
        EndIf

        If $measure_thickness == 1
            ElectronStats A
            electron_count_slitIn = $repVal5
        EndIf

        # reset IS and BT. TODO These might not be needed.
        SetImageShift $isux1 $isuy1 
        SetBeamTilt $btux1 $btuy1
        SetBeamShift $bsx1 $bsy1

        If $LAYER > 0
            nx = 0
            ny = 0
            Vh_x = $IS_RAD1_h * cos ( $IniAng_h )
            Vh_y = $IS_RAD1_h * sin ( $IniAng_h )
            Vv_x = $IS_RAD1_v * cos ( $IniAng_v )
            Vv_y = $IS_RAD1_v * sin ( $IniAng_v )

            If $TT != 0
                Vh_y = $Vh_y * cos ( $TT )
                Vv_y = $Vv_y * cos ( $TT )
            EndIf

            Loop $LAYER idx

                nx = $Vh_x
                ny = $Vh_y
                ImageShiftByMicrons $nx $ny 0 $BTtoIS
                #CallFunction AddBS $nx $ny
                ReportImageShift isux2 isuy2
                ReportBeamTilt btux2 btuy2
                CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots $DisplayReturn

                side1 = 2 * $idx - 1
                Loop $side1
                    nx = $Vv_x
                    ny = $Vv_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    #CallFunction AddBS $nx $ny
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots $DisplayReturn
                EndLoop

                side2 = 2 * $idx
                Loop $side2
                    nx = -1 * $Vh_x
                    ny = -1 * $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    #CallFunction AddBS $nx $ny
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots $DisplayReturn
                EndLoop

                side3 = 2 * $idx
                Loop $side3
                    nx = -1 * $Vv_x
                    ny = -1 * $Vv_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    #CallFunction AddBS $nx $ny
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots $DisplayReturn
                EndLoop

                side4 = 2 * $idx
                Loop $side4
                    nx = $Vh_x
                    ny = $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    #CallFunction AddBS $nx $ny
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots $DisplayReturn
                EndLoop

            EndLoop

        EndIf

        SetDefocus $origin_defocus 

        # completely reset IS and BT
        SetImageShift 0 0
        #SetBeamTilt $btux1 $btuy1
        #SetBeamShift $bsx1 $bsy1

    EndFunction

################################################################

    Function AquireMultiHoles_Even 0 0

        GoToLowDoseArea R

        ReportDefocus origin_defocus 

        ReportImageShift isux1 isuy1
        ReportBeamTilt btux1 btuy1
        ReportBeamShift bsx1 bsy1

        If $LAYER > 0

            nx = 0
            ny = 0
            Vh_x = $IS_RAD1_h * cos ( $IniAng_h )
            Vh_y = $IS_RAD1_h * sin ( $IniAng_h )
            Vv_x = $IS_RAD1_v * cos ( $IniAng_v )
            Vv_y = $IS_RAD1_v * sin ( $IniAng_v )

            ### for tilt
            If $TT != 0
                Vh_y = $Vh_y * cos ( $TT )
                Vv_y = $Vv_y * cos ( $TT )
            EndIf
            ###

            # Move to first layer
            nx = $Vh_x / 2 + $Vv_x / 2
            ny = $Vh_y / 2 + $Vv_y / 2
            ImageShiftByMicrons $nx $ny 0 $BTtoIS
            ReportImageShift isux2 isuy2
            ReportBeamTilt btux2 btuy2
            CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
            
            If $save_R == 1
                SaveToOtherFile A JPG NONE $log_dirR\Record_Item$item_label_Square$map_label_$acq_count_01.jpg
            EndIf

            Loop $LAYER idx

                # Move to next layer
                If $idx > 1
                    nx = $Vh_x
                    ny = $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndIf

                # Move around current layer
                side1 = 2 * $idx - 2
                Loop $side1
                    nx = $Vv_x
                    ny = $Vv_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side2 = 2 * $idx - 1
                Loop $side2
                    nx = -1 * $Vh_x
                    ny = -1 * $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side3 = 2 * $idx - 1
                Loop $side3
                    nx = -1 * $Vv_x
                    ny = -1 * $Vv_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side4 = 2 * $idx - 1
                Loop $side4
                    nx = $Vh_x
                    ny = $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

            EndLoop

        EndIf

        SetDefocus $origin_defocus 

        # completely reset IS and BT
        SetImageShift 0 0
        #SetBeamTilt $btux1 $btuy1
        #SetBeamShift $bsx1 $bsy1

    EndFunction

#######################################################

    Function ReportStatus 0 0

        ReportStageXYZ
        old_x := $repVal1
        old_y := $repVal2
        old_z := $repVal3
        Echo ===> X:$old_x Y:$old_y Z:$old_z

        ReportTiltAngle
        Echo ===> Tilt:$repVal1

    EndFunction

#######################################################

    Function ResetStatus 2 0 btx_base bty_base

        # Reset tilt angle
        If $TT != 0
            CallFunction CustomTilt::ResetTiltStage
        EndIf

        # Reset Beam-Tilt
        #SetBeamTilt $btx_base $bty_base
        #ReportBeamTilt btx_last bty_last
        #Echo ===> Reset Beam-Tilt to ( BTx: $btx_last, BTy: $bty_last )

    EndFunction

#######################################################

    Function MeasureThickness

        If $measure_thickness == 1

            Echo ===> Measuring the thickness

            SetExposure R 0.5
            SetDoseFracParams R 0 0 0 0 0

            # R
            # ElectronStats A
            # electron_count_slitIn = $repVal5

            GoToLowDoseArea R
            SetSlitIn 0
            UpdateLowDoseParams R

            R
            ElectronStats A
            electron_count_slitOut = $repVal5

            RestoreCameraSet R
            RestoreLowDoseParams R

            thickness = $electron_count_slitIn / $electron_count_slitOut
            thickness = ROUND $thickness 2

            If $thickness <= 0.7
                message_thickness @= "Too thick"
                ChangeItemColor $current_item 0 # red
            ElseIf 0.7 < $thickness AND  $thickness <= 0.8
                message_thickness @= "Thick"
                ChangeItemColor $current_item 3 # yellow
            ElseIf 0.8 < $thickness AND  $thickness <= 0.9
                message_thickness @= "So so"
                ChangeItemColor $current_item 1 # green
            ElseIf 0.9 < $thickness AND  $thickness <= 0.95
                message_thickness @= "Thin"
                ChangeItemColor $current_item 2 # blue
            Else
                message_thickness @= "Super thin! (or Empty)"
                ChangeItemColor $current_item 2 # blue
            EndIf 

            ChangeItemNote $current_item Thickness : $thickness, $message_thickness

            Echo ========================================================
            Echo ===> Thickness : $thickness (MeanRatio of slit in/out)
            Echo ===> $message_thickness
            Echo ========================================================

            ## Report ###           
            ReportDateTime 
            date = $repVal1
            OpenTextFile 1 T 0 $path_to_navfileReport$date.txt
            If ( $repVal1 == 1 )
                CloseTextFile 1
                OpenTextFile 1 A 0 $path_to_navfileReport$date.txt
                WriteLineToFile 1 $item_label: MeanRatio of slit in/out: $thickness 
            Else  
                OpenTextFile 1 W 0 $path_to_navfileReport$date.txt
                WriteLineToFile 1 $item_label: MeanRatio of slit in/out: $thickness 
            EndIf 

        EndIf

    EndFunction

#######################################################

    Function PhasePlateMonitor

        If $use_PhasePlate == 1

            IsVariableDefined PP_moved
            If $repVal1 == 0
                PP_moved := 0
                last_PP_image_num := $acquired_image_num
            EndIf

            If $use_ConditionSetup == 1
                Echo ===> Setting Phase Plate to a new position
                ConditionPhasePlate 1
                PP_moved := $PP_moved + 1
            ElseIf ($acquired_image_num - $last_PP_image_num) > $PP_interval_images
                Echo ===> Setting Phase Plate to a new position
                PhasePlateToNextPos
                Echo ---> Waiting for Phase Plate drift settling. $PP_drift_wait_time [min]
                Delay $PP_drift_wait_time min
                SetBeamBlank 0
                Echo ---> Waiting for Phase Plate charge up. $PP_charge_up_time [min]
                Delay $PP_charge_up_time min
                SetBeamBlank 1
                PP_moved := $PP_moved + 1
                last_PP_image_num := $acquired_image_num
            EndIf

            Echo ===> Phase Plate moved $PP_moved times

        Else
            PP_moved := 0
        EndIf

    EndFunction

#######################################################

    Function ReportProgress 1 0 ticks0

        ReportNumNavAcquire num_remain_item

        ReportTickTime current_ticks
        elapsed_ticks = $current_ticks - $ticks0
        total_elapsed_ticks = $current_ticks - $initial_ticks

        # convert to [min]
        elapsed_ticks = $elapsed_ticks / 60
        total_elapsed_ticks = $total_elapsed_ticks / 60

        # report progress
        Echo ===> Item $current_item took $elapsed_ticks min.
        Echo ===> Flashing:$flashed Refilling:$refilled ZLP:$ZLP_aligned DarkRef:$dark_updated
        
        If $total_elapsed_ticks <= 60
            Echo --> Elapsed time $total_elapsed_ticks min
        Else
            total_elapsed_ticks = $total_elapsed_ticks / 60
            Echo --> Elapsed time $total_elapsed_ticks hr
        EndIf

    EndFunction

#######################################################

    Function ChangeFocusForTilt 1 0 sp_y

        If $TT != 0

            additional_defocus = -1 * $sp_y * tan ( $TT )

            setting_defocus = $origin_defocus + $additional_defocus

            #If $apply_defocus_offset == 1
            #    SetDefocus $defocus_offset
            #EndIf

            SetDefocus $setting_defocus

            Echo ===> Defocus : $setting_defocus [um]

        EndIf

    EndFunction

#######################################################

    Function Custom_BTcomp 2 0 btx_base bty_base

        If $use_custom_BTcomp == 1
            # Get current IS
            ReportSpecimenShift ISx ISy
            # Calculate BT needed by IS
            BTx_IS = $ISx * $xpx + $ISy * $ypx 
            BTy_IS = $ISx * $xpy + $ISy * $ypy
            
            If $BTtoOL == 1
                # Get current defocus
                ReportDefocus OLval
                # Calculate BT needed by OL
                BTx_OL = $OLval * $BTx_to_OL
                BTy_OL = $OLval * $BTy_to_OL
            Else
                BTx_OL = 0
                BTy_OL = 0     
            EndIf

            BTx = $BTx_IS + $BTx_OL
            BTy = $BTy_IS + $BTy_OL

            # Apply BT for IS and OL
            SetBeamTilt ($btx_base + $BTx) ($bty_base + $BTy)
        EndIf

    EndFunction

#######################################################
EndMacro
Macro	23
MacroName Controller

#############################################################

    Function FocusControl 0 0

        # Check, if the stage moved to a different grid square at a different Z height:
        ReportStageXYZ
        NEWX = $repVal1
        NEWY = $repVal2
        NEWZ = $repVal3

        If ABS ($NEWX - $LASTX) > $stageX_limit_for_focus
            Echo Large X change detected. LASTX = $LASTX,  NEWX = $NEWX.  Requires new focussing.
            FOCUSCOUNTER := 0
            group_FLAG := 1
        EndIf

        If ABS ($NEWY - $LASTY) > $stageY_limit_for_focus
            Echo Large Y change detected. LASTY = $LASTY,  NEWY = $NEWY.  Requires new focussing.
            FOCUSCOUNTER := 0
            group_FLAG := 1
        EndIf

        #If ABS ($NEWZ - $LASTZ) > $stageZ_limit_for_focus
        #    Echo Z change detected. LASTZ = $LASTZ,  NEWZ = $NEWZ.  Requires new focussing.
        #    FOCUSCOUNTER := 0
        #    group_FLAG := 1
        #EndIf  

        #  if you wish, focusing before stage tilt anyway
        If $focus_before_tilt == 1
            Echo ---> Focusing will be performed before stage tilt anyway. 

        #  otherwise skip for the tilt stage (but if it is the first point on a square, do autofocus):
        Else 
            If ($do_tilt == 1) AND ($group_FLAG == 0)
                ReportTiltAngle
                curr_angle = $repVal1
                diff_angle = ABS ( $TT - $curr_angle )

                If $diff_angle > 3
                    Echo ===> Here is not the inital point of group.
                    Echo ===> Target tilt is $TT [degree], while current angle is $curr_angle [degree].
                    Echo ===> Focusing is skipped.
                    Return
                EndIf
            EndIf

        EndIf

        # Check, if it is already time to focus:
        If $FOCUSCOUNTER < 2

            Echo Focussing on this one..., Focus Counter = $FOCUSCOUNTER
        
            LASTX := $NEWX
            LASTY := $NEWY
            LASTZ := $NEWZ

            # Beam Centering by T
            T
            ReportMeanCounts A
            mean_count = $repVal1
            If $save_T == 1
                SaveToOtherFile A JPG NONE $log_dirT\Trial_beforeCentering_Item$item_label_Square$map_label_$acq_count.jpg
            EndIf

            If $mean_count > 10

                ReportBeamShift init_beamX init_BeamY
                Echo ---> BeamCentering...
                CenterBeamFromImage
                T
                ReportMeanCounts A
                mean_count = $repVal1

                If $mean_count < 10
                    skip_message @= "Skipped. Beam was not detected in Trial image."
                    CallFunction Funcs::AnnotateSkipItem $skip_message
                    SetBeamShift $init_beamX $init_BeamY
                    SkipAcquiringNavItem
                    Exit
                EndIf

            EndIf

            If $save_T == 1
                SaveToOtherFile A JPG NONE $log_dirT\Trial_afterCentering_Item$item_label_Square$map_label_$acq_count.jpg
            EndIf

            # If the image is black, increase focus_problem_counter.
            If $mean_count < 10
                Echo Trial image is black. MeanCounts was $repVal1.
                focus_problem_counter := $focus_problem_counter + 1
                echo FOCUS problem counter = $focus_problem_counter
            Else
                focus_problem_counter := 0
            EndIf

            # If focus_problem_counter get to 3, apply standard defocus and reset beamshift.
            If $focus_problem_counter > 3
                Echo Trying to save the beam:
                SetEucentricFocus   # Call standard objective focus
                SetBeamShift $good_beamshift_x $good_beamshift_y
                focus_problem_counter := 0
                FOCUSCOUNTER := 0
            EndIf

            # Set target defocus
            If $focused == 0
                CallFunction Funcs::CycleTargetDefocus
                ReportTargetDefocus original_TD
                
                If ($do_tilt == 1) AND ($TT != 0) AND ($use_eucentric_height == 1)
                    Echo ===> Use eucentric height for stage tilt.
                    Echo ===> Target defocus for Z is set to $eucentric_height [um].
                    SetTargetDefocus $eucentric_height
                EndIF

                FocusChangeLimits -70 70
            EndIf

            # Irradiation
            If $irradiation_time > 0
                Echo ---> Irradiating $irradiation_time sec
                GoToLowDoseArea F
                SetBeamBlank 0
                Delay $irradiation_time sec
                SetBeamBlank 1
            Else
                Echo ---> No irradiation.
            EndIf

            # AutoFocus
            Echo ---> Start AutoFocus
            F
            ReportMeanCounts A
            mean_count = $repVal1
            If $mean_count < 10
                Echo ---> No beam. Skip this area.
                Exit
            EndIf

            If $save_F == 1
                SaveToOtherFile A JPG NONE $log_dirF\Focus_Item$item_label_Square$map_label_$acq_count.jpg
            EndIf

            If ($focus_by_Z == 1) OR ($group_FLAG == 1)
                ReportTiltAngle curr_angle 
                If ABS ( $curr_angle ) < 3 # If the stage is almost flat (No tilt)
                    CallFunction CustomAutoFocus::AutoFocus_byZ
                    # Reset target defocus for tilted data collection after adjusting to eucentric height.
                    SetTargetDefocus $original_TD
                Else
                    If $stop_OLfocusing == 1 # This stops OL focusing after stage tilt anyway.
                        Echo ===> OL focusing is not applied...
                    Else
                        CallFunction CustomAutoFocus::AutoFocus_byOL
                    EndIf
                EndIf
            ElseIf ($focus_by_OL == 1)
                CallFunction CustomAutoFocus::AutoFocus_byOL
            EndIf
            focused = 1

            If $save_F == 1
                SaveToOtherFile A JPG NONE $log_dirF\Focus_Item$item_label_Square$map_label_$acq_count.jpg
            EndIf

            FOCUSCOUNTER := $focusEachHole
            ReportBeamShift
            good_beamshift_x := $repVal1
            good_beamshift_y := $repVal2
            focus_problem_counter := 0

            Echo ---> good beamshift = $good_beamshift_x, $good_beamshift_y

            # Beam Centering by F after Focusing
            If $beamCentering_afterFocus == 1
                F
                ReportMeanCounts A
                mean_count = $repVal1

                If $mean_count > 10
                    Echo ---> BeamCentering...
                    CenterBeamFromImage
                EndIf
            EndIf

            # Auto Coma Free
            If ($correct_coma == 1) AND ($PATTERN == 0)

                GoToLowDoseArea R
                SetExposure R 1.0
                SetDoseFracParams R 0 0 0 0 0

                FixComaByCtf # 0 1 for Image Shift exist  # 0 1 0 for Not full array

                If $correct_stig == 1 
                    FixAstigmatismByCTF 0 1
                EndIf

                RestoreCameraSet R

            EndIf

        # If it is not time to autofocus, then skip:
        Else

            Echo Skipping focussing, Focus Counter = $FOCUSCOUNTER

            If $skip_BeamCentering == 1
                Echo ---> Skip BeamCentering.
            Else
                GoToLowDoseArea T
                T
                ReportMeanCounts A
                mean_count = $repVal1

                If $mean_count > 10
                    Echo ---> BeamCentering...
                    CenterBeamFromImage
                EndIf
            EndIf

            FOCUSCOUNTER := $FOCUSCOUNTER - 1

        EndIf

    EndFunction

#############################################################

    Function DriftControl 0 0

        If $do_drift_control == 1

            If $do_tilt == 1
                ReportTiltAngle
                curr_angle = $repVal1
                diff_angle = ABS ( $TT - $curr_angle )

                If $diff_angle > 3
                    Echo ===> Target tilt is $TT [degree], while current angle is $curr_angle [degree].
                    Echo ===> Drift control is skipped.
                    Return
                EndIf

                If $drift_ctrl_when_tilt == 1
                    CallFunction Funcs::WaitForDrift $drift_crit $drift_shot
                    Echo ===> Additional settling time for the tilted stage. $tilt_settling_time [sec]
                    Delay $tilt_settling_time sec
                    Return
                EndIf
            EndIf


            If $once_every_group == 1
                If $group_FLAG == 1
                    CallFunction Funcs::WaitForDrift $drift_crit $drift_shot
                EndIf
            Else
                CallFunction Funcs::WaitForDrift $drift_crit $drift_shot
            EndIf

            Echo ===> Additional settling time $additional_settling_time [sec]
            Delay $additional_settling_time sec

        EndIf

    EndFunction

#############################################################

    Function TiltControl 0 0

        If $do_tilt == 1

            Echo ---> Tilt operation is ON.

            If $TT != 0
                Echo ---> Target tilt is $TT [degree].
                Echo ---> Performe tilt operation.

                ReportStageXYZ 
                x_saved = $repVal1
                y_saved = $repVal2
                z_saved = $repVal3

                CallFunction CustomTilt::TiltOnHole

                CallFunction CustomTilt::AlignToTiltHole

                CallFunction FocusControl

                CallFunction DriftControl
            EndIf

        EndIf

    EndFunction

#############################################################

    Function ZLPControll 0 0

        If $do_refineZLP == 1

            If $ZLP_when_flash == 1
                Echo ===> ZLPtest2
                If $flashed == 1
                   Call ZLPAlignByFL
                    
                    ZLP_aligned := 1
                Else
                    ZLP_aligned := 0
                EndIf
            Else
                IsVariableDefined last_ZLP
                If $repVal1 == 0
                    last_ZLP := $initial_ticks
                EndIf
                
                ReportTickTime current_ticks
                ZLP_elapsed_ticks = $current_ticks - $last_ZLP
                ZLP_elapsed_tmp = $ZLP_elapsed_ticks / 3600
                Echo ---> Elapsed time for ZLP : $ZLP_elapsed_tmp [hr]

                If $ZLP_elapsed_ticks >= $ZLPInterval
                   Call ZLPAlignByFL
                   #CallFunction ZLPAlignByFL::Core #$ZLP_thld_ratio $do_coarse_search $ZLP_by_maxCount $use_fine_step $FLcmd $area
                    ReportTickTime
                    last_ZLP := $repVal1
                    ZLP_aligned := 1
                Else
                    ZLP_aligned := 0
                EndIf
            EndIf

        Else
             ZLP_aligned := 0
        EndIf

    EndFunction

#############################################################
EndMacro
Macro	24
ScriptName ZLPAlignByFL

    # ============================
    # Parameters
    # ============================

    ZLP_thresh_ratio = 0.7
    ZLP_by_Count = 0
    area = R

    # ============================
    # Main
    # ============================

    Call EMProperties

    ZLP_thld_ratio = $ZLP_thresh_ratio 
    ZLP_by_maxCount = $ZLP_by_Count 

    RunInShell $FLcmdser &
    delay 12
   
    do_coarse = 1
    use_fine_step = 0
    CallFunction ZLPAlignByFL::Core #$ZLP_thld_ratio $do_coarse $ZLP_by_maxCount $use_fine_step $FLcmd $area
    do_coarse = 0
    use_fine_step = 1
    CallFunction ZLPAlignByFL::Core #$ZLP_thld_ratio $do_coarse $ZLP_by_maxCount $use_fine_step $FLcmd $area

###################################################

    Function Core
   #4 2 ZLP_thld_ratio do_coarse ZLP_by_maxCount use_fine_step FLcmd area

        SuppressReports
        GoToLowDoseArea $area

        ReportEnergyFilter 
        If $repVal3 == 0
            Echo ===> Energy Filter is out. 
            Echo ===> Skip ZLP refine.
            Exit
        Endif 

        # Get slit width
        ReportEnergyFilter
        slit_width = $repVal1

        # Set FL width for search range
        FL_width = $slit_width * 3
        echo $do_coarse
        If $do_coarse == 1
            FL_width = $slit_width * 4
        EndIf

        # Set FL step
        FL_step = NEARINT ( $FL_width / 10 ) #TODO check here for ROUND

        If $FL_step  < 1
            FL_step = 1
        ElseIf 4 < $FL_step
            FL_step = 4
        EndIF

        If $use_fine_step == 1
            FL_step = 1
        EndIf
    
        # Set Record mode params for RefineZLP
        SetExposure $area 0.5
        SetDoseFracParams $area 0 0 0 0 0

        # Start RefineZLP
        Echo # ======================================
        Echo ===> Start RefineZLP
        Echo Search range : $FL_width [eV]
        Echo Refinement FL step : $FL_step

        FL_min = NEARINT ( -1 * $FL_width / 2 - $FL_step ) 
        FL_search_range = NEARINT ( $FL_width / $FL_step ) + 1

        # by maxCount
        If ( $ZLP_by_maxCount == 1 )

            Echo Refinement method : Max mean count
            MaxCount = 0
            FL_shift = 0

            RunInShell $FLcmd $FL_min

            Loop $FL_search_range idx
                RunInShell $FLcmd $FL_step
                $area
                ReportMeanCounts A
                FL_tmp = $FL_min + $idx * $FL_step
                Echo Search index $idx (FL_shift, count) = ($FL_tmp, $repVal1)
                If $repVal1 > $MaxCount
                    MaxCount = $repVal1
                    FL_shift = $FL_tmp
                EndIf
            EndLoop

            Echo ===> Apply $FL_shift FL shift
            FL_shift_fromEnd = $FL_shift - $FL_tmp
            RunInShell $FLcmd $FL_shift_fromEnd

        # by pure ZLP center
        Else

            Echo Refinement method : Pure ZLP center
            ZLP_thld = 0

            NewArray arr_count -1 $FL_search_range 
            NewArray arr_FL_shift -1 $FL_search_range 

            # Set FL to lower limit
            RunInShell $FLcmd $FL_min
                           
            # Get array of electron count
            Echo ===> Searching ZLP Center
            Loop $FL_search_range idx
                RunInShell $FLcmd $FL_step

                $area
                ElectronStats A
                arr_count[$idx] = $repVal5
                arr_FL_shift[$idx] = $FL_min + $FL_step * $idx
                Echo Search Index $idx : FL_shift = $arr_FL_shift[$idx], count = $arr_count[$idx]
            EndLoop

            # Get array of averages from each 3 elements of electron count array
            len_arr_avg = $#arr_count - 2
            NewArray arr_avg -1 $len_arr_avg
            Loop $len_arr_avg idx
                avg = ( $arr_count[$idx] + $arr_count[$idx+1] + $arr_count[$idx+2] ) / 3
                arr_avg[$idx] = $avg
                If $avg > $ZLP_thld
                    ZLP_thld = $avg * $ZLP_thld_ratio
                EndIf
            EndLoop
            Echo ZLP threshold count : $ZLP_thld [e/px/s]

            # Get ZLP size (to create following array of ZLP positions)
            ZLP_size = 0
            Loop $len_arr_avg idx
                If $arr_avg[$idx] > $ZLP_thld
                    ZLP_size = $ZLP_size + 1
                EndIf
            EndLoop
            Echo ===> ZLP size : $ZLP_size

            # Get ZLP positions
            NewArray ZLP_positions -1 $ZLP_size
            idx2 = 0
            Loop $len_arr_avg idx1
                If $arr_avg[$idx1] > $ZLP_thld
                    idx2 = $idx2 + 1
                    ZLP_positions[$idx2] = $idx1 + 1 # Convert average array index to original count array index
                    Echo ZLP Index : $ZLP_positions[$idx2]
                EndIf
            EndLoop

            # Find ZLP center
            center_idx = NEARINT ( ($ZLP_size + 1) / 2 )
            ZLP_center = $ZLP_positions[$center_idx]

            isEvenSize = ( MODULO $ZLP_size 2 ) + 1            
            If $isEvenSize == 1
                If ($arr_count[$ZLP_center-1] > $arr_count[$ZLP_center])
                    ZLP_center = $ZLP_center - 1
                EndIf
            EndIf

            Echo ZLP Center : Index = $ZLP_center, count = $arr_count[$ZLP_center]

            # Set FL to ZLP center
            Echo ===> Apply $arr_FL_shift[$ZLP_center] FL shift
            FL_shift_fromEnd = ($ZLP_center - $FL_search_range) * $FL_step # Convert array index to FL_step
            RunInShell $FLcmd $FL_shift_fromEnd

        EndIf

        # Reset Record mode params
        RestoreCameraSet $area

        Echo ===> Finish RefineZLP
        Echo # ======================================

    EndFunction
EndMacro
Macro	25
ScriptName CustomAutoFocus

#######################################################

    Function AutoFocus_byZ

        process @= AutoFocus_byZ
        ReportTickTime
        start_ticks = $repVal1

        Echo ---> Focusing by Z

        SetEucentricFocus

        If $focus_by_Z == 1
            # To always perform drift control after Z-height chenge.
            once_every_group = 0
        EndIf

        ReportStageXYZ init_x init_y init_z

        Loop $max_focusZ_iter iter

            CallFunction Funcs::WaitForRefilling

            ReportTargetDefocus target_defocus
            G -1
            ReportAutofocus measured_defocus
            ReportStageXYZ setting_x setting_y setting_z
            defocus = $target_defocus - $measured_defocus
            relax_z1 = ( $defocus / ABS ($defocus) ) * $backlash_z
            relax_z2 = -1 * ( $defocus / ABS ($defocus) ) * $backlash_z
            range = ABS ($defocus)

            Echo ------------------------------------------------
            Echo Autofocus_byZ iter $iter
            Echo Target = $target_defocus [um]
            Echo Measured = $measured_defocus [um]
            Echo Need change ---> $defocus [um]

            # Success
            #If ($range <= $focus_error) AND ($measured_defocus <= $focus_th_low) AND ($focus_th_high <= $measured_defocus)
            If ($range <= $focus_error)
                
                Echo ===> Focusing by Z succeeded.
                # offset_Z = 0
                If $updata_Z_afterFocus == 1
                    UpdateGroupZ
                    UpdateItemZ
                EndIf
                Break

            # Failed
            ElseIf ( ABS ($measured_defocus) <= 0.0001 ) OR ( ABS ($measured_defocus) >= 200.0 ) OR ($setting_z <= $safty_z_lower) OR ($safty_z_upper <= $setting_z)
                skip_message @= "Skipped. Failuer of Auto Focus by Z."
                CallFunction Funcs::AnnotateSkipItem $skip_message
                MoveStageTo $init_x $init_y $init_z
                FOCUSCOUNTER := 0
                SkipAcquiringNavItem
                Exit

            # Ajdust focus by Z
            Else

                setting_z = $setting_z + $defocus + $relax_z1
                MoveStageTo $setting_x $setting_y $setting_z
                Echo ---> Move to $setting_z [um]
                Echo ---> Relaxing and Settling
                MoveStage 0 0 $relax_z2
                Delay $z_settle_time sec

                If $iter == $max_focusZ_iter
                    skip_message @= "Skipped. Failuer of Auto Focus by Z."
                    CallFunction Funcs::AnnotateSkipItem $skip_message
                    SkipAcquiringNavItem
                    Exit
                EndIf

            EndIf

        EndLoop

        ReportTickTime
        end_ticks = $repVal1
        CallFunction Funcs::ElapsedTimeMonitor $start_ticks $end_ticks $process

    EndFunction

#######################################################

    Function AutoFocus_byOL

        process @= AutoFocus_byOL
        ReportTickTime
        start_ticks = $repVal1

        Echo ---> Focusing by OL 

        ReportTiltAngle curr_angle
        ReportDefocus initial_defocus

        defocus_total = 0

        ReportStageXYZ init_x init_y init_z

        Loop $max_focus_iter iter

            CallFunction Funcs::WaitForRefilling

            ReportTargetDefocus target_defocus
            G -1
            ReportAutofocus measured_defocus
            ReportDefocus setting_defocus
            ReportStageXYZ setting_x setting_y setting_z
            defocus = $target_defocus - $measured_defocus
            # defocus_total = $defocus_total + $defocus
            range = ABS ($defocus)

            Echo ------------------------------------------------
            Echo Autofocus_byOL iter $iter
            Echo Target = $target_defocus [um]
            Echo Measured = $measured_defocus [um]
            Echo Need change ---> $defocus [um]

            # Success
            If ($range <= $focus_error) AND ($measured_defocus <= $focus_th_low) AND ($focus_th_high <= $measured_defocus)

                Echo ===> Focusing by OL succeeded.

                If $updata_Z_afterFocus == 1
                    UpdateGroupZ
                    UpdateItemZ
                EndIf
                Break

            # Failed
            ElseIf ( ABS ($measured_defocus) <= 0.0001 ) OR ( ABS ($measured_defocus) >= 200.0 )
                skip_message @= "Skipped. Failuer of Auto Focus by OL."
                CallFunction Funcs::AnnotateSkipItem $skip_message
                Echo ===> Reset defocus.
                SetDefocus $initial_defocus
                MoveStageTo $init_x $init_y $init_z
                FOCUSCOUNTER := 0
                SkipAcquiringNavItem
                Exit

            # Ajdust focus by Z for large defocus (>= 3 microns), only at stage is flat
            ElseIf ( ( $range >= 2 ) AND ( ABS ( $curr_angle ) <= 3 ) )

                relax_z1 = ( $defocus / ABS ($defocus) ) * $backlash_z
                relax_z2 = -1 * ( $defocus / ABS ($defocus) ) * $backlash_z
                setting_z = $setting_z + $defocus + $relax_z1
                
                MoveStageTo $setting_x $setting_y $setting_z
                Echo ---> Move to $setting_z [um]
                Echo ---> Relaxing and Settling
                MoveStage 0 0 $relax_z2
                Delay $z_settle_time sec

                If $iter == $max_focusZ_iter
                    skip_message @= "Skipped. Failuer of Auto Focus by OL."
                    CallFunction Funcs::AnnotateSkipItem $skip_message
                    SkipAcquiringNavItem
                    Exit
                EndIf

            # Ajdust focus by OL
            Else
                #SetEucentricFocus # Does it help to remove hysterisis?
                setting_defocus = $setting_defocus + $defocus
                SetDefocus $setting_defocus

                If $iter == $max_focus_iter
                    skip_message @= "Skipped. Failuer of Auto Focus by OL."
                    CallFunction Funcs::AnnotateSkipItem $skip_message
                    SkipAcquiringNavItem
                    Exit
                EndIf
            EndIf

        EndLoop

        ReportTickTime
        end_ticks = $repVal1
        CallFunction Funcs::ElapsedTimeMonitor $start_ticks $end_ticks $process

    EndFunction

#######################################################
EndMacro
Macro	26
ScriptName CustomTilt

#######################################################

    Function TiltOnHole 0 0

        tilt_step = { 5 10 20 30 40 50 60 70 }
        len_tilt_step = $#tilt_step

        first_step = 1
        dx = 0
        dy = 0
        dx_total = 0
        dy_total = 0

        Loop $len_tilt_step idx

            CallFunction Funcs::WaitForRefilling

            If $first_step == 1
                tilt_template_buffer = $template_buffer
            Else
                tilt_template_buffer = L
            EndIf

            If $TT > $tilt_step[$idx]
                TiltTo ($tilt_step[$idx] + $backlash_tilt)
                TiltTo $tilt_step[$idx]
                V

                IsVariableDefined mesh_bar_crit
                If $repVal1 == 0
                    ElectronStats T
                    mesh_bar_crit = $repVal5 / 2
                EndIf

                ElectronStats A
                electron_count = $repVal5
                If $electron_count < $mesh_bar_crit 
                    skip_message @= "Skipped. Lost beam while stage tilting."
                    CallFunction Funcs::AnnotateSkipItem $skip_message
                    Echo ===> Seems to hit the mesh bar.
                    CallFunction ResetTiltStage
                    SetImageShift 0 0
                    SkipAcquiringNavItem
                    Exit
                EndIf

                AlignTo $tilt_template_buffer
                ReportAlignShift
                dx = $repVal5
                dy = $repVal6 * -1 #?
                dx_total = $dx_total + $dx
                dy_total = $dy_total + $dy
                track_shift = sqrt ( $dx * $dx + $dy * $dy )
                track_shift_total = sqrt ( $dx_total * $dx_total + $dy_total * $dy_total )

                Echo ---> Current angle : $tilt_step[$idx] [degree]
                Echo ---> Shift for align : $track_shift_total (x:$dx_total, y:$dy_total) [nm]
                Echo --------------------------------
                
                If $track_shift >= $max_track_shift
                    skip_message @= "Skipped. Fail of tracking hole while stage tilt."
                    CallFunction Funcs::AnnotateSkipItem $skip_message 
                    CallFunction CustomTilt::ResetTiltStage
                    SetImageShift 0 0
                    SkipAcquiringNavItem
                    Exit
                EndIf

                V
                Copy A L
                first_step = 0

            Else
                TiltTo ($TT + $backlash_tilt)
                TiltTo $TT
                V

                IsVariableDefined mesh_bar_crit
                If $repVal1 == 0
                    ElectronStats T
                    mesh_bar_crit = $repVal5 / 2
                EndIf

                ElectronStats A
                electron_count = $repVal5
                If $electron_count < $mesh_bar_crit 
                    skip_message @= "Skipped. Lost beam while stage tilting."
                    CallFunction Funcs::AnnotateSkipItem $skip_message
                    Echo ===> Seems to hit the mesh bar.
                    CallFunction ResetTiltStage
                    SetImageShift 0 0
                    SkipAcquiringNavItem
                    Exit
                EndIf

                AlignTo $tilt_template_buffer
                ReportAlignShift
                dx = $repVal5
                dy = $repVal6 * -1 #?
                dx_total = $dx_total + $dx
                dy_total = $dy_total + $dy
                track_shift = sqrt ( $dx * $dx + $dy * $dy )
                track_shift_total = sqrt ( $dx_total * $dx_total + $dy_total * $dy_total )

                Echo ---> Current angle : $TT [degree]
                Echo ---> Shift for align : $track_shift_total (x:$dx_total, y:$dy_total) [nm]
                Echo --------------------------------

                If $track_shift >= $max_track_shift
                    skip_message @= "Skipped. Failure of tracking hole while stage tilt."
                    CallFunction Funcs::AnnotateSkipItem $skip_message 
                    CallFunction CustomTilt::ResetTiltStage
                    SetImageShift 0 0
                    SkipAcquiringNavItem
                    Exit
                EndIf

                Break
            EndIf

        EndLoop

    EndFunction

#######################################################

    Function AlignToTiltHole 0 0

        holeshift = $track_shift_total
        align_count = 0
        align_err_count = 0

        ReportDefocus initDefocus

        Loop $max_align_iter

            #? Something unknown happened (Unexpected large defocus was set). So initialize anyway.
            #SetEucentricFocus 
            SetDefocus $initDefocus

            CallFunction Funcs::WaitForRefilling

            If $holeshift < $maxholeshift

                If $align_byIS == 0
                    SetImageShift 0 0 # IS is anyway 8000 8000 for stage shift
                Endif

                Echo ----> Tilt hole align finished
                Break

            Else

                ResetImageShift 2 0.05 # relax stage
                
                # DEBUG =============
                #ReportDefocus $repVal1
                #Echo ---> OL : $repVal1    
                #SetEucentricFocus #? Something unknown happened (Unexpected large defocus was set). So initialize anyway.
                #====================

                CallFunction Funcs::CustomMoveStage 0 0
                V
                AlignTo L
                ReportAlignShift
                dx = $RepVal5
                dy = $RepVal6 * -1
                holeshift = sqrt ( $dx * $dx + $dy * $dy )

                align_count = $align_count + 1

                Echo Hole align iter $align_count
                Echo Shift ---> X:$dx [nm] Y:$dy [nm]
                Echo -------------------

                If $holeshift >= 500
                   align_err_count = $align_err_count + 1
                EndIf

                If $align_err_count >= 3
                    skip_message @= "Skipped. Failure of tracking hole while stage tilt."
                    CallFunction Funcs::AnnotateSkipItem $skip_message 
                    CallFunction CustomTilt::ResetTiltStage
                    SetImageShift 0 0
                    SkipAcquiringNavItem
                    Exit
                EndIf 

            EndIf

        EndLoop
         
        If $holeshift > $maxholeshift
            skip_message @= "Skipped. Failure of tracking hole while stage tilt."
            CallFunction Funcs::AnnotateSkipItem $skip_message 
            CallFunction CustomTilt::ResetTiltStage
            SetImageShift 0 0
            SkipAcquiringNavItem
            Exit
        EndIf

    EndFunction

#######################################################

    Function ResetTiltStage 0 0

        TiltTo (-1 * $backlash_tilt)
        TiltTo $backlash_tilt
        TiltTo 0
        MoveStageTo $x_saved $y_saved $z_saved
        CallFunction Funcs::CustomMoveStage 0 0

    EndFunction

#######################################################
EndMacro
Macro	27
ScriptName Funcs
## functions which can be called from a macro or a function.

#######################################################

    Function CycleTargetDefocus 0 0

        Echo ===> Running CycleTargetDefocus ...
        Echo >>>> defined Range and Step (um)  => [ $TD_low, $TD_high ], [ $TD_step ].

        delta = -1 * $TD_step

        SuppressReports
        ReportTargetDefocus 

        If ( $repVal1 > $TD_low ) OR ( $repVal1 < $TD_high )
            SetTargetDefocus $TD_low
        Else 
            IncTargetDefocus $delta
        Endif

        ReportTargetDefocus TD
        TD = ROUND $TD 2
        Echo TargetDefocus = $TD um

    EndFunction 

#######################################################

    Function CycleTargetTilt 0 0

        len_TT_list = $#TT_list

        # Set initial TT_freq_tmp 
        IsVariableDefined TT_freq_tmp
        If $repVal1 == 0
            TT_freq_tmp := $TT_freq
        EndIf

        # If flashing was performed, then update Target Tilt
        If ($changeTT_byFlashing == 1) AND ($flashed == 1)
            Loop $len_TT_list idx
                If $TT_freq_tmp[$idx] != 0
                    TT_freq_tmp[$idx] := $TT_freq_tmp[$idx] - 1
                    Break
                EndIf
            EndLoop
        EndIf

        # Before setting Target Tilt, check whether all elements of TT_freq_tmp are zero.
        keep_TT_freq = 0
        Loop $len_TT_list idx
            keep_TT_freq = $keep_TT_freq + $TT_freq_tmp[$idx]
        EndLoop
        # If zero, then reset TT_freq_tmp.
        If $keep_TT_freq == 0
            TT_freq_tmp := $TT_freq
            Echo ===> Target tilt is reset.
        EndIf

        # Set Target Tilt
        Loop $len_TT_list idx
            If $TT_freq_tmp[$idx] != 0
                TT = $TT_list[$idx]
                # Update Target Tilt for flashing-independet setting
                If $changeTT_byFlashing == 0
                    TT_freq_tmp[$idx] := $TT_freq_tmp[$idx] - 1
                EndIf
                Break
            Else
                Continue
            EndIf
        EndLoop

        Echo ===> Target tilt angle : $TT [degree]
        
    EndFunction

#######################################################

    Function WaitForRefilling 0 0

        AreDewarsFilling  # report 1 for stage tank. report 2 for transfer tank.

        If $repVal1 >= 1

            LongOperation RS 0 RT 0

            # Flashing at a time
            If $flash_when_refill == 1
                LongOperation FF 0
                ReportTickTime
                last_flashing := $repVal1
                flashed := 1
            EndIf

            # Dark ref at a time
            If ($darkRef_when_refill == 1) AND ($update_dark == 1)
                LongOperation Da 0
                ReportTickTime
                last_dark := $repVal1
                dark_updated := 1
            EndIf

            refilled := 1

            Loop 60

                Delay 30 sec
                AreDewarsFilling 

                If $repVal1 == 0
                    Break
                EndIf

                #Echo Still refilling

            EndLoop

            Echo ===> Waiting for the additional settling time of $delay_after_refill [min]
            Delay $delay_after_refill min
            Echo ===> LN2 refill completed.

        EndIf

    EndFunction

#######################################################

    Function ElapsedTimeMonitor 2 1 start_ticks end_ticks process

        ticks = $end_ticks - $start_ticks
        IsVariableDefined current_item

        If $repVal1 == 0
           Echo ===> $process took $ticks
        Else
           Echo ===> $process for $current_item took $ticks
        EndIf 

    EndFunction

#######################################################

    Function WaitForDrift 1 1 crit shot

        # Paremeters
        interval = 10
        times = 60
        period = $interval + 1

        # Skip if drift is already lower than the criteria
        ReportFocusDrift
        focus_drift_x = $repVal1 * 10
        focus_drift_y = $repVal2 * 10 # * -1 ??
        focus_drift = sqrt ( $focus_drift_x * $focus_drift_x + $focus_drift_y * $focus_drift_y )
        Echo -------------------------------
        Echo Focus drift rate: $focus_drift [A/s]
        Echo Rate X: $focus_drift_x [A/s], Rate Y: $focus_drift_y [A/s]
        Echo -------------------------------
        If ($focus_drift <= $crit) AND ($use_focus_drift == 1)
            Echo ===> Focus drift is lower than $crit
            Echo ===> Drift Control would be skipped.
        # If not, do drift control
        Else
            Echo ===> Running WaitForDrift $crit  [A/sec] ...

            $shot
            Delay $interval

            Loop $times idx

                CallFunction WaitForRefilling

                $shot
                AlignTo B
                ReportAlignShift
                ClearAlignment
                dx = $repVal5
                dy = $repVal6 * -1
                dist = sqrt ( $dx * $dx + $dy * $dy )

                If $dist >= 50

                    skip_message @= "Skipped. Failure of cross-correlation while drift measurement"
                    CallFunction Funcs::AnnotateSkipItem $skip_message 
                    SkipAcquiringNavItem
                    Exit

                EndIf

                rate = $dist / $period * 10	
                Echo Rate = $rate A/sec
                Echo driftX:$dx [nm], driftY:$dy [nm]
                Echo ----------------

                If $rate < $crit

                    echo Drift is low enough after shot $idx
                    break

                ElseIf  $idx < $times

                    If $resist_drift == 1
                        CallFunction ResistDrift $dx $dy $rate $crit
                        $shot
                    EndIf

                    Delay $interval

                Else

                    skip_message @= "Skipped because of huge drift."
                    CallFunction Funcs::AnnotateSkipItem $skip_message 
                    SkipAcquiringNavItem
                    Exit

                EndIf

            EndLoop

        EndIf

    EndFunction

#######################################################

    Function ResistDrift 4 0 dx dy rate crit
        
        resist_drift_crit = $crit * 1.0

        If $rate >= $resist_drift_crit
            
            anti_x = -1 * $dx    # -5 * $dx
            anti_y = -1 * $dy    # -5 * $dy
            anti_x_m = $anti_x / 1000 # [um]
            anti_y_m = $anti_y / 1000 # [um]

            Echo ---> Resist a drift.
            Echo ---> Move X:$anti_x [nm]  Y:$anti_y [nm]
            Echo ----------------

            Loop 1
                MoveStage $anti_x_m $anti_y_m
            EndLoop

        Else

            Echo ---> Drift is low enough to stop resisting.

        EndIf
        
    EndFunction

######################################################

    Function CustomMoveStage 2 0 x y

        If $use_apprV == 1
            MoveStage ($x + $apprV_X1) ($y + $apprV_Y1)
            MoveStage ($apprV_X2 - $apprV_X1) ($apprV_Y2 - $apprV_Y1)
            MoveStage (-1 * $apprV_X2) (-1 * $apprV_Y2)
        Else
            MoveStage $x $y
        EndIf

    EndFunction

#######################################################

    Function CustomMoveStageTo 2 0 x y

        If $use_apprV == 1
            MoveStageTo ($x + $apprV_X1) ($y + $apprV_Y1)
            MoveStageTo ($x + $apprV_X2) ($y + $apprV_Y2)
        EndIf

        MoveStageTo $x $y

    EndFunction

#######################################################

    Function GoToNextNavItem 0 0

        If $use_apprV == 1

            ReportNextNavAcqItem

            If $repVal1 != -1
                next_x = $repVal2
                next_y = $repVal3
                next_z = $repVal4
                MoveStageTo ($next_x + $apprV_X1) ($next_x + $apprV_Y1) $next_z
                MoveStageTo ($next_x + $apprV_X2) ($next_x + $apprV_Y2) $next_z
            EndIf

        EndIf

    EndFunction

#######################################################

    Function AlignToHole 3 1 maxholeshift max_align_iter align_byIS template_buffer

        # Initial settings
        Echo ===> Image Shift is reset to 0

        align_count = 0
        align_err_count = 0

        # Reset Piezo
        If $use_Piezo == 1
            CallFunction Funcs::RelaxPiezoXY 
            CallFunction Funcs::CustomMovePiezoXY 0 0
            Delay 1 sec
        EndIf

        # Start Hole Aliginment
        CallFunction Funcs::WaitForRefilling
        Echo ===> Running AlignToHole ...
        V

        # Report Shift
        AlignTo $template_buffer
        align_count = $align_count + 1
        ReportAlignShift # Shift on specimen X/Y axis (TiltX/Y) 
        dx = $RepVal5 #[nm]
        dy = $RepVal6 * -1 #[nm]
        Echo Hole align iter $align_count
        Echo Shift ---> X:$dx [nm] Y:$dy [nm]
        Echo -------------------
        holeshift = sqrt ( $dx * $dx + $dy * $dy )

        If $holeshift >= 500
            align_err_count = $align_err_count + 1
        EndIf

        # Iterate Hole Alignment
        Loop $max_align_iter

            # Success
            If $holeshift < $maxholeshift
                If $align_byIS == 0
                    SetImageShift 0 0
                EndIf

                If $use_Piezo == 1
                    Loop 10
                        V
                        Echo ===> Piezo drive
                        AlignTo $template_buffer
                        ReportSpecimenShift dx dy
                        SetImageShift 0 0 0 0
                        shift_x = $dx
                        shift_y = $dy * -1 
                        Echo ===> Need shift $shift_x $shift_y
                        CallFunction Funcs::MovePiezoXY_byMicron $shift_x $shift_y
                        holeshift = sqrt ( $dx * $dx + $dy * $dy ) * 1000 #[nm]
                        If $holeshift  <= 30
                            Break
                        EndIf
                    EndLoop
                    
                EndIf

                Echo ===> hole align finished
                Break

            # Iterate
            Else
                ResetImageShift 2 0.05 # relax stage
                If $use_apprV == 1
                    CallFunction Funcs::CustomMoveStage 0 0
                EndIF
                V
                AlignTo $template_buffer
                align_count = $align_count + 1
                ReportAlignShift
                dx = $RepVal5 # Shift on specimen X axis [nm]
                dy = $RepVal6 * -1 # Shift on specimen Y axis [nm]
                holeshift = sqrt ( $dx * $dx + $dy * $dy )
                Echo Hole align iter $align_count
                Echo Shift ---> X:$dx [nm] Y:$dy [nm]
                Echo -------------------

                If $holeshift >= 500
                    align_err_count = $align_err_count + 1
                EndIf

                # Failed...
                If $align_err_count >= 3
                    Echo ===> Hole align failed....
                    Echo ===> Skip this item.
                    SetImageShift 0 0
                    Exit
                EndIf

            EndIf

        EndLoop
    
    EndFunction

#######################################################

    Function ReadHoleImage 0 0

        Call EMProperties

        ReportNavFile 1
        path_to_navfile = $repVal3
        file_name @= hole_template.mrc
        hole_file1 = $path_to_navfile$file_name

        path_to_HoleImage @= $path_to_rootSPA\Data\HoleImage
        hole_file2 = $path_to_HoleImage\hole_template.mrc

        # ====== Main =======

        Echo ====== Start AlignToHole =======

        # Set View mode of LD
        SetLowDoseMode 1
        GoToLowDoseArea V

        # Read hole image
        open_again = 0

        Echo $template_buffer
        Echo $hole_file2

        Try
            ReadOtherFile 0 $template_buffer $hole_file1
        Catch
            Echo ===> Cannot find $hole_file1
            Echo ===> Try to open $hole_file2
            ## Temporaly
            ReadOtherFile 0 $template_buffer $hole_file2
        EndTry 

        # If $open_again == 1
        #     Try
        #         ReadOtherFile 0 $template_buffer $hole_file2
        #     Catch
        #         Echo ===> Cannot find $hole_file2
        #         Echo ===> Script requires hole template image.
        #         Echo ===> Script terminated.
        #         Exit
        #     EndTry
        # EndIf

    EndFunction

#################################################

    Function MovePiezoXY_byMicron 2 0 dx dy

        dx = $dx * 1000
        dy = $dy * 1000

        ReadTextFile piezo_mat $path_to_rootSPA\PiezoMat.txt
        a = $piezo_mat[1]
        b = $piezo_mat[2]
        c = $piezo_mat[3]
        d = $piezo_mat[4]

        x = ($dx * $d - $dy * $b) / ($a * $d - $b * $c)
        y = ($dy * $a - $dx * $c) / ($a * $d - $b * $c)

        ReportPiezoXY x0 y0
        x = $x0 + $x
        y = $y0 + $y
        CallFunction Funcs::CustomMovePiezoXY $x $y

    EndFunction

#################################################

    Function CustomMovePiezoXY 2 0 X Y

        MovePiezoXY -1.7 -1.2 # Almost Max
        MovePiezoXY $X $Y

    EndFunction

#################################################

    Function RelaxPiezoXY 0 0

        Loop 1
            CallFunction CustomMovePiezoXY 0 0
        EndLoop

    EndFunction

############################################################

Function SetAtlasIllumination 1 0 removeCLapt

    Call EMProperties

    SetMag $mag_atlas
    Delay 5 sec

    # Remove CL apt
    If $removeCLapt == 1
        If $CLapt_type == 0
            RemoveAperture 1
        ElseIf $CLapt_type == 1
            RemoveAperture 0
        EndIf
    EndIf
    #Remove OL apt
    RemoveAperture 2
    #RunInShell python $path_to_rootSPA\Tool\InsertCLapt.py 1 1
    #echo python $path_to_rootSPA\Tool\InsertCLapt.py 1 1
    # Remove Slit
    SetSlitIn 0
    #SetSpotSize $spot_atlas
    SetPercentC2 $brightness_atlas # 100%

    SetEucentricFocus

EndFunction

############################################################

Function SetSquareIllumination 1 0 removeCLapt

    Call EMProperties

    # Remove CL apt
    #If $removeCLapt == 1
     #   If $CLapt_type == 0
     #       RemoveAperture 1
     #   ElseIf $CLapt_type == 1
     #       RemoveAperture 0
    #    EndIf
    #EndIf
    # Remove OL apt
    # RemoveAperture 2
    RunInShell python $path_to_rootSPA\Tool\InsertCLapt.py  1 1
    echo python $path_to_rootSPA\Tool\InsertCLapt.py 1 1
    SetSlitIn 0
    SetSpotSize $spot_square
    SetPercentC2 $brightness_square 

    SetEucentricFocus

EndFunction

############################################################

Function AnnotateSkipItem 0 1 message_skipped

    IsVariableDefined run_from_datacollection
    If $repVal1 == 1
       ReportNavItem 
       nav_idx = $repVal1
       label_skipped @= $navLabel-skipped
       Echo ===> Item_$navLabel : $message_skipped
       ChangeItemLabel $nav_idx $label_skipped
       ChangeItemNote $nav_idx $message_skipped
       ChangeItemColor $nav_idx 3
    Else
        Echo There is no item for acquisition.
    EndIf

EndFunction

############################################################

Function BeamCentering 1 1 useScreen area

    GoToLowDoseArea $area
    $area
    ReportMeanCounts A
    mean_count = $repVal1
    If $mean_count > 10
        ReportBeamShift init_beamX init_beamY
        Echo ---> BeamCentering...
        $area
        CenterBeamFromImage
        $area
        ReportMeanCounts A
        mean_count = $repVal1

        If $mean_count < 10
            Echo ---> Beam Centering failed... Reset BeamShift.
            SetBeamShift $init_beamX $init_beamY
        EndIf
    EndIf

EndFunction

############################################################

Function TakeRecord

    recNum := $recNum + 1
    ReportTickTime
    recTime = $repVal1 - $iniTime
    recTime = ROUND $recTime 0
    fileNameIn @= $recNum_SlitIn_$recTimes.tif
    fileNameOut @= $recNum_SlitOut_$recTimes.tif
    # s for [sec]

    # Log message
    If $slitOut == 1
        Echo Recoding, $recTime [s], $fileNameIn, $fileNameOut
    Else
        Echo Recoding, $recTime [s], $fileNameIn
    EndIf    

    # Open Valve and wait for 5sec
    SetBeamBlank 1
    SetColumnOrGunValve 1
    Delay 5 sec

    # Unblank and take an image
    SetBeamBlank 0
    R
    SaveToOtherFile A TIF LZW $fileNameIn

    # Slit out if needed
    If $slitOut == 1
        SetSlitIn 0
        R
        SaveToOtherFile A TIF LZW $fileNameOut
    
    EndIf

    ReportTickTime
    lastRec := $repVal1

    # Beam blank and close valve
    SetBeamBlank 1
    SetColumnOrGunValve 0
    SetSlitIn 1

    Echo --------------------------

EndFunction

###########################################

    Function WaitForRefilling2 0 0

        AreDewarsFilling  # report 1 for stage tank. report 2 for transfer tank.

        If $repVal1 >= 1
           
            ReportTickTime
            refillTime = $repVal1 - $iniTime
            refillTime = ROUND $refillTime 0
            Echo Refill, $refillTime [s]

            LongOperation RS 0 RT 0

            # Flashing at a time
            If $flash_when_refill == 1
                LongOperation FF 0
                ReportTickTime
                last_flashing := $repVal1
                flashed := 1
            EndIf

            # Dark ref at a time
            If ($darkRef_when_refill == 1) AND ($update_dark == 1)
                LongOperation Da 0
                ReportTickTime
                last_dark := $repVal1
                dark_updated := 1
            EndIf

            refilled := 1
            
            SetColumnOrGunValve 0

            Echo ===> Refilling and pressurizing...
            Loop 60

                Delay 30 sec
                AreDewarsFilling 

                If $repVal1 == 0
                    Break
                EndIf

                #Echo Still refilling

            EndLoop

            Echo ===> Waiting for the additional settling time of $delay_after_refill [min]
            Delay $delay_after_refill min
            Echo ===> LN2 refill completed.
            
            Echo --------------------------

        EndIf

    EndFunction

#######################################################

# ScriptName CycleLD

# iter = 4

# SuppressReports 
# SetLowDoseMode 1

# Echo ------------------------
# Loop $iter idx
#     GoToLowDoseArea V
#     Echo Round $idx , View1
#     Delay 0.5 sec

#     GoToLowDoseArea T
#     Echo Round $idx , Trial
#     Delay 0.5 sec
#     If $idx == $iter
#         T
#         ReportMeanCounts A
#         mean_count = $repVal1
#         If $mean_count > 10
#             ReportBeamShift init_beamX init_BeamY
#             Echo ---> BeamCentering...
#             CenterBeamFromImage
#             T
#             ReportMeanCounts A
#             mean_count = $repVal1

#             If $mean_count < 10
#                 Echo ---> Beam Centering failed... Reset BeamShift.
#                 SetBeamShift $init_beamX $init_BeamY
#             EndIf
#         EndIf
#     EndIf

#     GoToLowDoseArea F
#     Echo Round $idx , Focus
#     Delay 0.5 sec

#     GoToLowDoseArea R
#     Echo Round $idx , Record
#     Delay 0.5 sec
#     Echo -------------------------
# EndLoop
# Echo ===> Finish CycleLD
EndMacro
Macro	28
ScriptName CalibrationBeamVSImage_onScreen

# This script is for calibration beam shift vs image shift
# Enter four values getting from this script in calibration file (C:\\ProgramData\SrialEM\SerialEMcalibrations.txt)
# At first, screen down and check beam centering on screen
#

shiftBeam = 10 #[um]
waittime  = 1


ScreenDown
GoToLowDoseArea R
SetImageShift 0 0

echo Report Beam shift and Image shift on Image shift (0,0)
ReportImageShift
i0_shiftx = $RepVal1
i0_shifty = $RepVal2

ReportBeamShift
b0_shiftx = $RepVal1
b0_shifty = $RepVal2

ImageShiftByMicrons 0 0 # 1 1

echo Beam centering by beam shift knob (Beam Align 1) on screen within $waittime [sec]
count = $waittime
loop $waittime
   echo remaining time $count [sec]
   delay 1
   count = $count - 1
EndLoop
#T

# ========= 1st ============
echo 1st Image shift ($shiftBeam,0) ...
ImageShiftByMicrons $shiftBeam 0 1 0
#T

ReportImageShift
i1_shiftx = $RepVal1
i1_shifty = $RepVal2

echo
echo Beam centering by beam shift knob (Beam Align 1) on screen within $waittime [sec]
count = $waittime
loop $waittime
   echo remaining time $count [sec]
   delay 1
   count = $count - 1
EndLoop


ReportBeamShift
b1_shiftx = $RepVal1
b1_shifty = $RepVal2

xb1 = $b1_shiftx - $b0_shiftx
yb1 = $b1_shifty - $b0_shifty
xi1 = $i1_shiftx
yi1 = $i1_shifty

#reset beam
SetImageShift 0 0
SetBeamShift $b0_shiftx $b0_shifty
delay 1
# =========================

# ========= 2nd ============
echo 2nd Image shift (0, $shiftBeam) ...
ImageShiftByMicrons 0 $shiftBeam 1 0
#T

ReportImageShift
i2_shiftx = $RepVal1
i2_shifty = $RepVal2

echo
echo Beam centering by beam shift knob (Beam Align 1) on screen within $waittime [sec]
count = $waittime
loop $waittime
   echo remaining time $count [sec]
   delay 1
   count = $count - 1
EndLoop

ReportBeamShift
b2_shiftx = $RepVal1
b2_shifty = $RepVal2

xb2 = $b2_shiftx - $b0_shiftx
yb2 = $b2_shifty - $b0_shifty
xi2 = $i2_shiftx
yi2 = $i2_shifty

#Reset beam
SetImageShift 0 0
SetBeamShift $b0_shiftx $b0_shifty
delay 1
# =========================
a = ( $xb1 * $yi2 -  $xb2 * $yi1 ) / ( $xi1 * $yi2 - $xi2 * $yi1 )
b = ( $xb1 * $xi2 -  $xb2 * $xi1 ) / ( $xi2 * $yi1 - $xi1 * $yi2 )
c = ( $yb1 * $yi2 -  $yb2 * $yi1 ) / ( $xi1 * $yi2 - $xi2 * $yi1 )
d = ( $yb1 * $xi2 -  $yb2 * $xi1 ) / ( $xi2 * $yi1 - $xi1 * $yi2 )

ReportMagIndex
magIn = $RepVal1
ReportMag
mag = $RepVal1
ReportAlpha
alpha = $RepVal1 - 1

echo Result of calibration is as follows... Rewrite it on "BeamShiftCalibration" line in C:\\ProgramData\SrialEM\SerialEMcalibrations.txt
#echo $a $b $c $d
echo BeamShiftCalibration $magIn $a $b $c $d $alpha 1  $mag
EndMacro
Macro	33
MacroName IS-Pixel
Trial
AutoFocus
ShiftCalSkipLensNorm
CalibrateImageShift
ReportMagIndex
SetupWaffleMontage 5 wafflemont-$repVal1.mrc
If $repVal1 == 1
  Montage
  Copy B A
Else
 Record
Endif
#Record
FindPixelSize
EndMacro
Macro	34
MacroName Parameters_Screening


    Echo ---> Calling Parameters ...

    #====================================================
    # Flashing, Dark Reference and Refill
    #====================================================
    # C-FEG flashing 
    FlashInterval := 12 * 3600 #[hrs x 3600]
    #FlashInterval := 0.05 * 3600 # 3min for test

    # Dark reference for K2/K3
    update_dark = 0 # 0:No, 1:Yes
    DarkRefInterval := 6 * 3600 #[hrs x 3600]

    # Refill
    flash_when_refill := 1
    darkRef_when_refill := 0
    delay_after_refill = 15 # [min]

    #====================================================
    # ZLP centering
    #====================================================
    do_refineZLP = 0 # 0:No, 1:Yes
    ZLPInterval  := 6 * 3600 #[hrs x 3600]
    area = R # Area for ZLP alignment

    ZLP_thld_ratio = 0.7 # 0:No, 1:Yes
    do_coarse_search = 0 # 0:No, 1:Yes
    use_fine_step = 1 # 0:No, 1:Yes
    ZLP_by_maxCount = 0 # 0:No, 1:Yes
    ZLP_when_flash = 0 # 0:No, 1:Yes

    #====================================================
    # Hole Alignment
    #====================================================
    # Template buffer for hole alignment
    template_buffer = T

    # Threshold for realigning to hole by stage
    maxholeshift = 0.15 * 1000 #[um x 1000] 

    # Maximum iteration for hole alignment
    max_align_iter = 10

    # Use IS for hole alignment?
    align_byIS = 1 # 0:No, 1:Yes

    # Use Piezo drive?
    use_Piezo = 0 # 0:No, 1:Yes
    piezo_threshold = 30 #[nm]

    # If yes, stop hole alignment after Focusing and Drift control. Cancels "realignBeforeRecord"
    stop_hole_realign = 0 # 0:No, 1:Yes

    #====================================================
    # Drift control
    #====================================================
    # Do drift control?
    do_drift_control = 1 # 0:No, 1:Yes
    use_focus_drift = 1 # 0:No, 1:Yes
    once_every_group = 1 # 0:No, 1:Yes
    drift_ctrl_when_tilt = 1 # 0:No, 1:Yes (If yes, always drift control after tilt)
    tilt_settling_time = 0 # [sec], additional settling time after stage tilt

    # Drift rate threshold [A/sec], only get used for skip = 0
    drift_crit = 7 #[A/sec]
    drift_shot = F

    # Move opposit direction while waiting drift ?
    resist_drift = 1 # 0:No, 1:Yes

    additional_settling_time = 0 #[sec]

    #====================================================
    # Focus
    #====================================================
    # Set TargetDefocus(TD) range and step.
    TD_low  = -2 #[um]
    TD_high = -2 #[um]
    TD_step = 0.1 #[um]

    # Autofocus error
    focus_error = 0.2 #[um] #0.2

    # When the stage moves following distance, then autofocus.
    stageX_limit_for_focus = 50 #[um]
    stageY_limit_for_focus = 50 #[um]
    stageZ_limit_for_focus = 5 #[um]

    # Define, how often the microscope should focus: 
    # focusEachHole := 0   means to keep focus target constant and focus always.
    # focusEachHole := 1   means to increment focus target and focus always.
    # focusEachHole := 5   means to increment focus target and focus only every 5th image.
    focusEachHole := 0

    # Irradiation for focus
    irradiation_time = 0 #[sec]

    # Maximum iteration for auto focus
    max_focusZ_iter = 10
    max_focus_iter = 5

    # Skip BeamCentering when auto focus is NOT performed
    skip_BeamCentering = 1 # 0:No, 1:Yes
    beamCentering_afterFocus = 0 # 0:No, 1:Yes

    # Do focusing by Z?
    focus_by_Z = 0 # 0:No, 1:Yes

    # Do focusing by OL?
    focus_by_OL = 1 # 0:No, 1:Yes

    # Update group Z after Focus?
    updata_Z_afterFocus = 1 # 0:No, 1:Yes

    # Do focusing by CTFFIND? (Under development)
    focus_by_ctf = 0 # 0:No, 1:Yes

    # Do Auto Coma Free after focusing? (Under developmental)
    correct_coma = 0 # 0:No, 1:Yes
    correct_stig = 0 # 0:No, 1:Yes

    # Z_byV every square?
    do_Z_byV = 0 # 0:No, 1:Yes

    # Z focusing settling time
    z_settle_time = 3 #[sec]

    # Maximum erro of Z height change from the initial point of the group
    #safetyZ = 40 # [um]

    #====================================================
    # Record
    #====================================================
    # --- Use SerialEM based multiple Record? ---
    # Recommendation is 0 or 1. Use script-based multiple-hole setting and do NOT use script-based multi-shot
    use_multiR = 0 # 0:No, 1:Use only multi-shot setting, 2: Use both multiple-shot and multiple-hole setting

    # --- Use Script based multiple Record? ---
    # For script based multiple-hole 
    PATTERN = 1  # 0:even, 1:odd
    LAYER = 0  # odd LAYER => 1: 3x3, 2: 5x5, 3: 7x7, ...    # even LAYER => 1: 2x2, 2: 4x4, 3: 6x6, ... # 0 => single shot

    realignBeforeRecord = 1 #0: No, 1:Yes

    # For script based multi-shot
    # Recommendation is 0
    do_multishot = 0 # 0:No, 1:Yes

    # If set to 1, IS is always 0 at center of multi-hole pattern
    zero_IS = 0 # 0:No, 1:Yes

    # Measure thickness? (Only availavle when PATTERN = 1)
    measure_thickness = 1 # 0:No, 1:Yes

    # For Acceptance test
    R_delay = 0 # [sec]

    #====================================================
    # Active Beam-Tilt Compensation
    #====================================================
    # Perform BT(BeamTilt) compensation? (Use SerialEM implemented)
    BTtoIS = 1 # 0:No, 1:Yes
    
    # Use your own data? (Scrip implemented). If Yes, SerialEM implemented BT compensation is neglected.
    use_custom_BTcomp = 0 # 0:No, 1:Yes
    BTtoOL = 0 #0:No, 1:Yes

    #====================================================
    # Tilt (Under development)
    #====================================================
    # Set TargetTilt (TT) and frequency 
    do_tilt = 0 # 0:No, 1:Yes
    TT_list = { 0 10 20 30 40 } # [degree]
    TT_freq = { 0 2 1 0 1 }
    changeTT_byFlashing = 1 # 0:No 1:Yes

    focus_before_tilt = 1 # 0:No 1:Yes, Focusing by Z "before tilting".
    use_eucentric_height = 0 # 0:No 1:Yes, Adjust Z to eucentric height "before tilting".

    max_track_shift = 3000 #[nm]
    stop_OLfocusing = 0 # 0:No 1:Yes, Stop focusing by OL "after tilting" anyway (For NIH claiming)

    #====================================================
    # Phase Plate Setting
    #====================================================
    use_PhasePlate = 0 # 0:No, 1:Yes
    use_ConditionSetup = 1 # 0:No, 1:Yes, Use setting from "Phase Plate Condtioning Setup dialog"

    # If use_ConditionSetup is 1, below setting are ignored.
    PP_interval_images = 60 #[images]
    PP_drift_wait_time = 3 #[min]
    PP_charge_up_time = 1 #[min]

    #====================================================
    # Display Setting
    #====================================================  
    stop_display_R = 0 # 0:display, 1:stop display

    # Just for "EarlyReturnNextShot".
    # This is for no return for Record Frame exposure for a K2/K3 camera.
    noReturn = -1
 
    # How often do you want to show images?
    # If you want to see the recorded image for every movie (DisplayOnly=1) or 
    # only for every 10th movie (DisplayOnly=10) or even more rarely:
    DisplayOnly = 1000

    #====================================================
    # LogImage Setting
    #====================================================  
    save_V = 1 # 0:No, 1:Yes
    save_T = 1 # 0:No, 1:Yes
    save_F = 1 # 0:No, 1:Yes
    save_R = 1 # 0:No, 1:Yes

    #====================================================
    # Do not have to touch below
    #====================================================
    parameters_type = 1
EndMacro
Macro	35
MacroName SPADataCollection_Screening

# Prepare for Cryo-ARM200 in NIH
# Sep. 12, 2019
#
# Modified for Cryo-ARM200 in NIH
# Feb. 10, 2020
#
# Modified for Cryo-ARM200 in NIH
# Aug. 25, 2020
#
# Modified for Cryo-ARM300 in Scripps
# Oct. 20, 2020
#
# Modified for Cryo-ARM200 in NIH and Cryo-ARM300 in Scripps
# Nov. 10, 2020
#
# Modified for Cryo-ARM200 in NIH and Cryo-ARM300 in Scripps
# Jan. 12, 2021
#
# Modified for CryoARM200 in Regensburg
# Feb. 10, 2021

    #======#
    # Main #
    #======#

    Echo =============== Starting SPADataCollection_Screening Macro =============== 

    SuppressReports
    SetLowDoseMode 1

    ReportTickTime
    ticks0 = $repVal1

    Call EMProperties

    Call Parameters_Screening

    Call Initializer

    CallFunction FlashingMonitor

    #CallFunction DarkRefMonitor

    CallFunction UpdateGroupZ

    CallFunction AlignToHole

    CallFunction Controller::FocusControl

    CallFunction Controller::DriftControl

    CallFunction Controller::TiltControl

    CallFunction RealignToHole

    If $zero_IS == 1
        SetImageShift 0 0
    EndIf

    CallFunction ReportStatus

    CallFunction AquireMultiHoles

    CallFunction ResetStatus $btx_base $bty_base

    CallFunction MeasureThickness

    CallFunction Controller::ZLPControll

    CallFunction ReportProgress $ticks0

#######################################################

    Function FlashingMonitor 0 0

        IsVariableDefined last_flashing
        If $repVal1 == 0
            last_flashing := $initial_ticks
        EndIf
        
        ReportTickTime current_ticks
        flashing_elapsed_ticks = $current_ticks - $last_flashing

        If $flashing_elapsed_ticks >= $FlashInterval
            echo ---> Flashing ...
            LongOperation FF 0 # Flashing
            ReportTickTime
            last_flashing := $repVal1
            flashed := 1
        Else
            flashed := 0
        EndIf

        # For tilted data collection with Cold-FEG
        If $do_tilt == 1
            CallFunction Funcs::CycleTargetTilt
        Else
            TT = 0
        EndIf

    EndFunction

#######################################################

    Function DarkRefMonitor 0 0

        If $update_dark == 1
            IsVariableDefined last_dark
            If $repVal1 == 0
                last_dark := $initial_ticks
            EndIf

            ReportTickTime current_ticks
            darkRef_elapsed_ticks = $current_ticks - $last_dark

            If $darkRef_elapsed_ticks >= $DarkRefInterval
                echo ---> Flashing ...
                LongOperation Da 0 # DarkRef
                ReportTickTime
                last_dark := $repVal1
                dark_updated := 1
            Else
                dark_updated := 0
            EndIf
        Else
            dark_updated := 0
        EndIf

    EndFunction
    
#######################################################

    Function InitialRelax 0 0

        IsVariableDefined old_x
        
        If $repVal1 == 1

            shift_x = $curr_x - $old_x
            shift_y = $curr_y - $old_y
            shift_z = $curr_z - $old_z

            If ABS ( $shift_x ) < 10 #[um]
                signX = 0
            Else
                signX = $shift_x / ABS ( $shift_x )
            Endif 

            If ABS ( $shift_y ) < 10 #[um]
                signY = 0
            Else 
                signY = $shift_y / ABS ( $shift_y )
            Endif 

            Loop 3 idx
                Echo ---> iter $idx
                moveX = -1 * $signX * $backlash_x
                moveY = -1 * $signY * $backlash_y
                Echo ---> Relaxing by moving stage $moveX $moveY ...
                MoveStage $moveX $moveY
            EndLoop

        EndIf

    EndFunction

#######################################################

    Function UpdateGroupZ 0 0

        process @= UpdateGroupZ
        ReportTickTime
        start_ticks = $repVal1

        group_FLAG := 0

        CallFunction Funcs::WaitForRefilling

        IsVariableDefined CURRENTGROUP
        If $repval1 == 0
            CURRENTGROUP := 0
            accum_item_shiftX := 0
            accum_item_shiftY := 0
        EndIf

        ReportGroupStatus
        groupID = $repval2

        If $CURRENTGROUP != $groupID

            # Reset Item coordinate
            Echo ===> Reset Item coordinates
            ShiftItemsByMicrons $accum_item_shiftX $accum_item_shiftY
            accum_item_shiftX := 0
            accum_item_shiftY := 0
            ReportNavItem 
            item_id =  $repVal1
            Echo ===> Move to Item $item_id
            MoveToNavItem $item_id
            CallFunction Funcs::CustomMoveStage 0 0

            # Update Z hight
            CURRENTGROUP := $groupID
            If ( $do_Z_byV == 1 )
                Echo ----> Update Z-height
                Call Z_byV
                UpdateGroupZ
                UpdateItemZ
                # Drift control in View
                CallFunction Funcs::WaitForDrift 5 V
            EndIf
            group_FLAG := 1

        EndIf
        
        ReportTickTime
        end_ticks = $repVal1
        CallFunction Funcs::ElapsedTimeMonitor $start_ticks $end_ticks $process

    EndFunction

#######################################################

    Function AlignToHole 0 0

        process @= AlignToHole
        ReportTickTime
        start_ticks = $repVal1

        # Initial settings
        maxholeshift_tmp = $maxholeshift
        align_byIS_tmp = $align_byIS

        If ( $do_tilt == 1 ) AND ( $TT != 0 )
            maxholeshift = 400
            align_byIS = 1
        EndIf

        ReportImageShift IS_X0 IS_Y0
        ReportSpecimenShift sp_x0 sp_y0
        align_count = 0
        align_err_count = 0

        # Start Hole Aliginment
        CallFunction Funcs::WaitForRefilling
        Echo ===> Running AlignToHole ...
        V
        If $save_V == 1
            SaveToOtherFile A JPG NONE $log_dirV\View_beforeAlign_Item$item_label_Square$map_label_$acq_count.jpg
        EndIf

        # Report Shift
        AlignTo $template_buffer
        align_count = $align_count + 1
        ReportAlignShift # Shift on specimen X/Y axis (TiltX/Y) 
        dx = $RepVal5 #[nm]
        dy = $RepVal6 * -1 #[nm]
        Echo Hole align iter $align_count
        Echo Shift ---> X:$dx [nm] Y:$dy [nm]
        Echo -------------------
        holeshift = sqrt ( $dx * $dx + $dy * $dy )

        If $holeshift >= 500
            align_err_count = $align_err_count + 1
        EndIf

        # Initial shift for item realign
        ReportSpecimenShift sp_x1 sp_y1
        diff_x = $sp_x1 - $sp_x0
        diff_y = $sp_y0 - $sp_y1 # ? Y is opposite direction..

        # Iterate Hole Alignment
        Loop $max_align_iter
            CallFunction Funcs::WaitForRefilling
            # Success
            If $holeshift < $maxholeshift

                # Realign items
                ShiftItemsByMicrons $diff_x $diff_y
                # Accumulated item shift used for reset when group is changed.
                accum_item_shiftX := $accum_item_shiftX - $diff_x
                accum_item_shiftY := $accum_item_shiftY - $diff_y

                If $align_byIS == 0
                    SetImageShift 0 0 # IS is anyway 8000 8000 for stage shift
                EndIf

                Echo ===> hole align finished
                If $save_V == 1
                    SaveToOtherFile A JPG NONE $log_dirV\View_afterAlign_Item$item_label_Square$map_label_$acq_count.jpg
                EndIf

                maxholeshift = $maxholeshift_tmp
                align_byIS = $align_byIS_tmp

                Break
            # Iterate
            Else
                ResetImageShift 2 0.05 # relax stage
                ReportSpecimenShift sp_x0 sp_y0
                If $use_apprV == 1
                    CallFunction Funcs::CustomMoveStage 0 0
                EndIF
                V
                AlignTo $template_buffer
                align_count = $align_count + 1
                ReportAlignShift
                dx = $RepVal5 # Shift on specimen X axis [nm]
                dy = $RepVal6 * -1 # Shift on specimen Y axis [nm]
                holeshift = sqrt ( $dx * $dx + $dy * $dy )
                Echo Hole align iter $align_count
                Echo Shift ---> X:$dx [nm] Y:$dy [nm]
                Echo -------------------

                If $holeshift >= 500
                    align_err_count = $align_err_count + 1
                EndIf

                # Shift for item realign
                ReportSpecimenShift sp_x1 sp_y1
                diff_x = $diff_x + ( $sp_x1 - $sp_x0 )
                diff_y = $diff_y + ( $sp_y0 - $sp_y1 ) # ? Y is opposite direction..

                # Failed...
                If $align_err_count >= 3
                    skip_message @= "Skipped. Hole alignment failure."
                    CallFunction Funcs::AnnotateSkipItem $skip_message 
                    SetImageShift 0 0
                    SkipAcquiringNavItem
                    Exit
                EndIf

            EndIf

        EndLoop

        ReportTickTime
        end_ticks = $repVal1
        CallFunction Funcs::ElapsedTimeMonitor $start_ticks $end_ticks $process
    
    EndFunction

#######################################################

Function RealignToHole 0 0

If ($focus_by_Z == 1) AND ($group_FLAG == 1)
    If $stop_hole_realign == 1
        Echo ===> Stop hole alignment after Focusing or Drift control
    ElseIf $do_tilt == 0
        Echo ===> Performs hole realign.
        CallFunction AlignToHole
        GoToLowDoseArea T
        GoToLowDoseArea F
        GoToLowDoseArea R
        SetEucentricFocus
    EndIF
EndIf

EndFunction

#######################################################

    Function AquireHole 6 0 isux isuy btux btuy NumberShots DisplayReturn

       GoToLowDoseArea R
       SetSlitIn 1
       UpdateLowDoseParams R

        If $NumberShots > 0

            Loop $NumberShots icount2

                IS_ANGLE = ( $icount2 - 1 ) * 360 / $NumberShots
                IS_X = $IS_RAD2 * sin ( $IS_ANGLE )
                IS_Y = $IS_RAD2 * cos ( $IS_ANGLE )

                ### for tilt
                If $TT != 0
                    IS_Y = $IS_Y * cos ( $TT )
                EndIf
                ###

                ImageShiftByMicrons $IS_X $IS_Y 0 $BTtoIS

                CallFunction Funcs::WaitForRefilling                

                ReportSpecimenShift sp_x sp_y

                # Set defocus for tilted stage
                CallFunction ChangeFocusForTilt $sp_y
                # Set active BT comp if desired
                CallFunction Custom_BTcomp $btx_base $bty_base

                If $stop_display_R == 1
                    #EarlyReturnNextShot 0                 # comment out by FM due to K2syn... 1
                    R
                Else
                    #EarlyReturnNextShot $DisplayReturn   # comment out by FM due to K2syn... 1
                    R
                EndIf

                sp_x_round = ROUND $sp_x 2
                sp_y_round = ROUND $sp_y 2
                Echo ===> ImageShiftX:$sp_x_round [um], ImageShiftY:$sp_y_round [um]

                SetImageShift $isux $isuy
                SetBeamTilt $btux $btuy

            EndLoop

        Else

            CallFunction Funcs::WaitForRefilling

            ReportSpecimenShift sp_x sp_y

            # Set defocus for tilted stage
            CallFunction ChangeFocusForTilt $sp_y
            # Set active BT comp if desired
            CallFunction Custom_BTcomp $btx_base $bty_base

            If $stop_display_R == 1
                #EarlyReturnNextShot 0
                R
            Else
                #EarlyReturnNextShot $DisplayReturn
                R
                # For test
                # Try
                #     FixComaByCTF  1 1
                # Catch
                #     Echo ===> Skip here
                # EndTry
            EndIf

            sp_x_round = ROUND $sp_x 2
            sp_y_round = ROUND $sp_y 2
            Echo ===> ImageShiftX:$sp_x_round [um], ImageShiftY:$sp_y_round [um]

        EndIf


    EndFunction
    
################################################################

    Function AquireMultiHoles 0 0

        process @= AquireMultiHoles
        ReportTickTime
        start_ticks = $repVal1

        If $use_multiR == 1
            MultipleRecords -9 -9 -9 0 1
        Else
            If ( $PATTERN == 0 ) AND ( $LAYER > 0 ) 
                CallFunction AquireMultiHoles_Even
            Else
                CallFunction AquireMultiHoles_Odd
            EndIf
        EndIf

        Echo -----> Done
        ReportTickTime
        end_ticks = $repVal1
        CallFunction Funcs::ElapsedTimeMonitor $start_ticks $end_ticks $process

    EndFunction

#######################################################

    Function AquireMultiHoles_Odd 0 0
        
        GoToLowDoseArea R

        ReportDefocus origin_defocus 

        ReportImageShift isux1 isuy1
        ReportBeamTilt btux1 btuy1
        ReportBeamShift bsx1 bsy1

        CallFunction AquireHole $isux1 $isuy1 $btux1 $btuy1 $NumberShots $DisplayReturn

        If $measure_thickness == 1
            ElectronStats A
            electron_count_slitIn = $repVal5
        EndIf

        If $save_R == 1
           ReduceImage A 4
           SaveToOtherFile A JPG NONE $log_dirR\Record_Item$item_label_Square$map_label_$acq_count_01.jpg
        EndIf

        # reset IS and BT. TODO These might not be needed.
        SetImageShift $isux1 $isuy1 
        SetBeamTilt $btux1 $btuy1
        SetBeamShift $bsx1 $bsy1

        If $LAYER > 0
            nx = 0
            ny = 0
            Vh_x = $IS_RAD1_h * cos ( $IniAng_h )
            Vh_y = $IS_RAD1_h * sin ( $IniAng_h )
            Vv_x = $IS_RAD1_v * cos ( $IniAng_v )
            Vv_y = $IS_RAD1_v * sin ( $IniAng_v )

            If $TT != 0
                Vh_y = $Vh_y * cos ( $TT )
                Vv_y = $Vv_y * cos ( $TT )
            EndIf

            Loop $LAYER idx

                nx = $Vh_x
                ny = $Vh_y
                ImageShiftByMicrons $nx $ny 0 $BTtoIS
                #CallFunction AddBS $nx $ny
                ReportImageShift isux2 isuy2
                ReportBeamTilt btux2 btuy2
                CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots $DisplayReturn

                side1 = 2 * $idx - 1
                Loop $side1
                    nx = $Vv_x
                    ny = $Vv_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    #CallFunction AddBS $nx $ny
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots $DisplayReturn
                EndLoop

                side2 = 2 * $idx
                Loop $side2
                    nx = -1 * $Vh_x
                    ny = -1 * $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    #CallFunction AddBS $nx $ny
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots $DisplayReturn
                EndLoop

                side3 = 2 * $idx
                Loop $side3
                    nx = -1 * $Vv_x
                    ny = -1 * $Vv_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    #CallFunction AddBS $nx $ny
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots $DisplayReturn
                EndLoop

                side4 = 2 * $idx
                Loop $side4
                    nx = $Vh_x
                    ny = $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    #CallFunction AddBS $nx $ny
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots $DisplayReturn
                EndLoop

            EndLoop

        EndIf

        SetDefocus $origin_defocus 

        # completely reset IS and BT
        SetImageShift 0 0
        #SetBeamTilt $btux1 $btuy1
        #SetBeamShift $bsx1 $bsy1

    EndFunction

################################################################

    Function AquireMultiHoles_Even 0 0

        ReportDefocus origin_defocus 

        ReportImageShift isux1 isuy1
        ReportBeamTilt btux1 btuy1
        ReportBeamShift bsx1 bsy1

        If $LAYER > 0

            nx = 0
            ny = 0
            Vh_x = $IS_RAD1_h * cos ( $IniAng_h )
            Vh_y = $IS_RAD1_h * sin ( $IniAng_h )
            Vv_x = $IS_RAD1_v * cos ( $IniAng_v )
            Vv_y = $IS_RAD1_v * sin ( $IniAng_v )

            ### for tilt
            If $TT != 0
                Vh_y = $Vh_y * cos ( $TT )
                Vv_y = $Vv_y * cos ( $TT )
            EndIf
            ###

            # Move to first layer
            nx = $Vh_x / 2 + $Vv_x / 2
            ny = $Vh_y / 2 + $Vv_y / 2
            ImageShiftByMicrons $nx $ny 0 $BTtoIS
            ReportImageShift isux2 isuy2
            ReportBeamTilt btux2 btuy2
            CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
            
            If $save_R == 1
               ReduceImage A 4
                SaveToOtherFile A JPG NONE $log_dirR\Record_Item$item_label_Square$map_label_$acq_count_01.jpg
            EndIf

            Loop $LAYER idx

                # Move to next layer
                If $idx > 1
                    nx = $Vh_x
                    ny = $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndIf

                # Move around current layer
                side1 = 2 * $idx - 2
                Loop $side1
                    nx = $Vv_x
                    ny = $Vv_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side2 = 2 * $idx - 1
                Loop $side2
                    nx = -1 * $Vh_x
                    ny = -1 * $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side3 = 2 * $idx - 1
                Loop $side3
                    nx = -1 * $Vv_x
                    ny = -1 * $Vv_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

                side4 = 2 * $idx - 1
                Loop $side4
                    nx = $Vh_x
                    ny = $Vh_y
                    ImageShiftByMicrons $nx $ny 0 $BTtoIS
                    ReportImageShift isux2 isuy2
                    ReportBeamTilt btux2 btuy2
                    CallFunction AquireHole $isux2 $isuy2 $btux2 $btuy2 $NumberShots
                EndLoop

            EndLoop

        EndIf

        SetDefocus $origin_defocus 

        # completely reset IS and BT
        SetImageShift 0 0
        #SetBeamTilt $btux1 $btuy1
        #SetBeamShift $bsx1 $bsy1

    EndFunction

#######################################################

    Function ReportStatus 0 0

        ReportStageXYZ
        old_x := $repVal1
        old_y := $repVal2
        old_z := $repVal3
        Echo ===> X:$old_x Y:$old_y Z:$old_z

        ReportTiltAngle
        Echo ===> Tilt:$repVal1

    EndFunction

#######################################################

    Function ResetStatus 2 0 btx_base bty_base

        # Reset tilt angle
        If $TT != 0
            CallFunction CustomTilt::ResetTiltStage
        EndIf

        # Reset Beam-Tilt
        #SetBeamTilt $btx_base $bty_base
        #ReportBeamTilt btx_last bty_last
        #Echo ===> Reset Beam-Tilt to ( BTx: $btx_last, BTy: $bty_last )

    EndFunction

#######################################################

    Function MeasureThickness

        If $measure_thickness == 1

            Echo ===> Measuring the thickness

            SetExposure R 1
            SetDoseFracParams R 0 0 0 0 0

            # R
            # ElectronStats A
            # electron_count_slitIn = $repVal5

            GoToLowDoseArea R
            SetSlitIn 0
            UpdateLowDoseParams R

            R
            ElectronStats A
            electron_count_slitOut = $repVal5

            RestoreCameraSet R
            RestoreLowDoseParams R

            thickness = $electron_count_slitIn / $electron_count_slitOut
            thickness = ROUND $thickness 3

            If $thickness <= 0.7
                message_thickness @= "Too thick"
                ChangeItemColor $current_item 0 # red
            ElseIf 0.7 < $thickness AND  $thickness <= 0.8
                message_thickness @= "Thick"
                ChangeItemColor $current_item 3 # yellow
            ElseIf 0.8 < $thickness AND  $thickness <= 0.9
                message_thickness @= "So so"
                ChangeItemColor $current_item 1 # green
            ElseIf 0.9 < $thickness AND  $thickness <= 0.95
                message_thickness @= "Thin"
                ChangeItemColor $current_item 2 # blue
            Else
                message_thickness @= "Super thin! (or Empty)"
                ChangeItemColor $current_item 2 # blue
            EndIf 

            GoToLowDoseArea R
            SetSlitIn 1
            UpdateLowDoseParams R

            
            ChangeItemNote $current_item Thickness : $thickness, SquareMap $map_label, $message_thickness

            Echo ========================================================
            Echo ===> Thickness : $thickness (MeanRatio of slit in/out)
            Echo ===> $message_thickness
            Echo ========================================================

            ## Report ###           
            ReportDateTime 
            date = $repVal1
            OpenTextFile 1 T 0 $path_to_navfileReport$date.txt
            If ( $repVal1 == 1 )
                CloseTextFile 1
                OpenTextFile 1 A 0 $path_to_navfileReport$date.txt
                WriteLineToFile 1 $item_label: MeanRatio of slit in/out: $thickness, SquareMap $map_label
            Else  
                OpenTextFile 1 W 0 $path_to_navfileReport$date.txt
                WriteLineToFile 1 $item_label: MeanRatio of slit in/out: $thickness, SquareMap $map_label 
            EndIf 

        EndIf

    EndFunction

#######################################################

    Function ReportProgress 1 0 ticks0

        ReportNumNavAcquire num_remain_item

        ReportTickTime current_ticks
        elapsed_ticks = $current_ticks - $ticks0
        total_elapsed_ticks = $current_ticks - $initial_ticks

        # convert to [min]
        elapsed_ticks = $elapsed_ticks / 60
        total_elapsed_ticks = $total_elapsed_ticks / 60

        # report progress
        Echo ===> Item $current_item took $elapsed_ticks min.
        Echo ===> Flashing:$flashed Refilling:$refilled ZLP:$ZLP_aligned DarkRef:$dark_updated
        
        If $total_elapsed_ticks <= 60
            Echo --> Elapsed time $total_elapsed_ticks min
        Else
            total_elapsed_ticks = $total_elapsed_ticks / 60
            Echo --> Elapsed time $total_elapsed_ticks hr
        EndIf

    EndFunction

#######################################################

    Function ChangeFocusForTilt 1 0 sp_y

        If $TT != 0

            additional_defocus = -1 * $sp_y * tan ( $TT )

            setting_defocus = $origin_defocus + $additional_defocus

            #If $apply_defocus_offset == 1
            #    SetDefocus $defocus_offset
            #EndIf

            SetDefocus $setting_defocus

            Echo ===> Defocus : $setting_defocus [um]

        EndIf

    EndFunction

#######################################################

Function Custom_BTcomp 2 0 btx_base bty_base

    If $use_custom_BTcomp == 1
        # Get current IS
        ReportSpecimenShift ISx ISy
        # Calculate BT needed by IS
        BTx_IS = $ISx * $xpx + $ISy * $ypx 
        BTy_IS = $ISx * $xpy + $ISy * $ypy
        
        If $BTtoOL == 1
            # Get current defocus
            ReportDefocus OLval
            # Calculate BT needed by OL
            BTx_OL = $OLval * $BTx_to_OL
            BTy_OL = $OLval * $BTy_to_OL
        Else
            BTx_OL = 0
            BTy_OL = 0     
        EndIf

        BTx = $BTx_IS + $BTx_OL
        BTy = $BTy_IS + $BTy_OL

        # Apply BT for IS and OL
        SetBeamTilt ($btx_base + $BTx) ($bty_base + $BTy)
    EndIf

EndFunction
EndMacro
Macro	36
ScriptName Check_ActiveBTComp

IS_distance = 5
exp_time = 0.5
BTtoIS = 1

# ==========================================

SuppressReports 
SetExposure R $exp_time 
SetBinning R 2
SetDoseFracParams R 0 0 0 0 0

Echo ===> Set IS to 0
SetImageShift 0 0
FixComaByCTF 1 1

Echo ===> Applying $IS_distance mircon IS_X
ImageShiftByMicrons $IS_distance 0 0 $BTtoIS
FixComaByCTF 1 1
SetImageShift 0 0 0 $BTtoIS

Echo ===> Applying $IS_distance mircon IS_Y
ImageShiftByMicrons 0 $IS_distance 0 $BTtoIS
FixComaByCTF 1 1
SetImageShift 0 0 0 $BTtoIS

RestoreCameraSet R
EndMacro
Macro	37
       SetFEGEmissionState 0
       #SetColumnOrGunValve 0
EndMacro
Macro	39
MacroName EMProperties

    Echo ---> Calling EMProperties ...
    
    #====================================================
    # Scope type
    #====================================================
    scope_type = 1 # 0 for CryoARM200, CryoARM300II, 1 for CryoARM300

    # ------ Set CLapt type -------
    # Set CLapt1 for CARM200 and CARM300II
    If ($scope_type == 0) OR ($scope_type == 2)
        CLapt_type = 1
    # Set CLapt1 for CARM300
    ElseIf $scope_type == 1
        CLapt_type = 0
    EndIf
    # -------------------------------------

    #====================================================
    # Working Directory (Setting directory)
    #====================================================
    path_to_rootSPA @= C:\ProgramData\SerialEM\
    #====================================================
    # Session path setting
    #====================================================
    Try
        ReportNavFile 1
        path_to_navfile = $repVal3
    Catch
        Echo ===> Navigator file is close
    EndTry

    #====================================================
    # TEM property
    #====================================================
    # Stage
    backlash_x      = 0.05 #[um]
    backlash_y      = 0.05 #[um]
    backlash_z      = 0.9 #[um]
    backlash_tilt   = 5 #[degree]

    # Use approach vector?
    use_apprV   = 1 # 0:No, 1:Yes
    apprV_X1    = -25 #[um]
    apprV_Y1    = -25 #[um]
    apprV_X2    = 5 #[um]
    apprV_Y2    = 5 #[um]

    safty_z_lower = -200 #[um]
    safty_z_upper = 200 #[um]

    eucentric_height = 0 # [um]
    eucentric_offset = -95 # [um]

    #====================================================
    # FL control command
    #====================================================
    FL_by_PyJEM = 1

    # FL control command
    If $FL_by_PyJEM == 1
        FLcmd @= "python $path_to_rootSPA\Tool\shift_FL_client.py"    
        FLcmdser @= "$path_to_rootSPA\Tool\shift_FL_server.bat"   
    Else
        FLcmd @= "C:\Program Files\FLUpDown\FLUpDownApp.exe"
    EndIf

    #====================================================
    # Offset for Atlas to Square
    #====================================================
    atlas2square_x = 0  # [um]    
    atlas2square_y = 0 # [um]    

    #====================================================
    # Offset for Square to View
    #====================================================
    square2view_x = 6 # [um]  
    square2view_y = 22 # [um]  

    #====================================================
    # Setting for Atlas
    #====================================================
    mag_atlas = 50
    brightness_atlas = 100 # [%] for SetPercentC2

    piece_X = 5 # 6
    piece_Y = 7 #8

    overlap_x = 402
    overlap_y = 402
    frame_x = 5216
    frame_y = 3836

    # Initial beam shift for Atlas
    AtlasBSX = 0 #-0.7
    AtlasBSY = 0 #5.06

    #====================================================
    # Settingfor Square
    #====================================================
    # Default square mag
    default_SquareMag = 150

    # For square mag illumination
    spot_square = 4
    brightness_square = 100 # [%] for SetPercentC2

    # Initial beam shift for Square
    SquareBSX = 0 
    SquareBSY = 0

    #====================================================
    # Z height Offset for View and Record
    #====================================================
    # Offset for Z_byV
    offset_for_Z_byV = 0 #[um]

    #====================================================
    # Initial Beam Shift for View of LowDoseMode 
    #====================================================
    LowDoseBSX = 0 #-2.580 
    LowDoseBSY = 0 #-3.190

    #====================================================
    # Python Call setting
    #====================================================
    # For Regensburg CryoARM200
    #path_to_Rb_Python @= C:\Python_Script
    #cmd_findV @= python $path_to_Rb_Python\Tool\FindHoleLattice.py

    # Start by python
    cmd_findV @= python $path_to_rootSPA\Tool\FindHoleLattice.py

    # Start by python
    #cmd_findV @= C:\Anaconda3\python.exe $path_to_rootSPA\Tool\FindHoleLattice.py

    # Start by bat
    #cmd_findV @= $path_to_rootSPA\Tool\FindHoleLattice_py36.bat 
EndMacro
Macro	40
ReportNumNavAcquire
EndMacro
Macro	41
LongOperation FF 0
EndMacro
Macro	42
SetEucentricFocus 
EndMacro
Macro	43
removeaperture 1
EndMacro
Macro	44
movestage 0 0 90
EndMacro
