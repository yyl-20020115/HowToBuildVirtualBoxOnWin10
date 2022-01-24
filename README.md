This project is about how to build VirtualBox 6.1 on Windows platform.

Environment setup and tools to install:
Win10 (ENGLISH VERSION PREFERRED)

TortoiseSVN (https://tortoisesvn.net/downloads.html)

Python2.7 (https://www.python.org/downloads/release/python-2718/)

Strawberry Perl 5.32 (https://strawberryperl.com/download/5.32.1.1/strawberry-perl-5.32.1.1-64bit.msi)

yasm (http://www.tortall.net/projects/yasm/releases/yasm-1.3.0-win64.exe)

nasm (https://www.nasm.us/pub/nasm/releasebuilds/2.15.05/win64/nasm-2.15.05-installer-x64.exe)

Windows Platform SDK V7.1A (https://www.microsoft.com/en-us/download/details.aspx?id=8279)

Windows DDK V7.1 (https://www.microsoft.com/en-us/download/details.aspx?displaylang=en&id=11800)

Windows 10 WDK 19041 (wdksetup-19041)

Visual Studio 2019 (VC142) (with git support) (find it online by yourself)

Windows 10 SDK 10.0.19041.0 (This can be found in VS2019 installation)

Qt5.9.0.0 (https://mirrors.tuna.tsinghua.edu.cn/qt/archive/qt/5.9/5.9.0/qt-opensource-windows-x86-5.9.0.exe)

SDL-1.2 (git clone https://github.com/libsdl-org/SDL-1.2.git)

Tools installation:

    install Win10-EN

    install VS2019-EN (With Win10SDK 10.0.19041.0)

    install WDK-19041

    install python2.7

    install Strawberry Perl

    install WINPSDK7.1

    install DDK71

    install nasm

    install yasm

      download and rename and copy to C:\Windows\System32

    install Qt5.9.0.0

      to C:\Qt

    try C:/WinDDK/7600.16385.1/bin/selfsign/inf2cat.exe 

        if .NET Framework3.5 is not installed, Windows will install it automatically


Source preparation:	
        
    Copy kBuild/sdks/WINPSDK71INCS.kmk to kBuild/sdks/WINPSDK71-INCS.kmk
    Copy /y "C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\shared\sal.h" 
        to C:\WinDDK\7600.16385.1\inc\ddk 
           C:\WinDDK\7600.16385.1\inc\api
             and overwrite if having one already (remember to backup first)

    Copy /y C:\WinDDK\7600.16385.1\lib\win7\amd64\newdev.lib "C:\Program Files (x86)\Windows Kits\10\Lib\10.0.19041.0\km\x64\"
    Copy /y C:\WinDDK\7600.16385.1\lib\win7\i386\newdev.lib  "C:\Program Files (x86)\Windows Kits\10\Lib\10.0.19041.0\km\x86\"
    Copy /y C:\WinDDK\7600.16385.1\lib\win7\amd64\ddraw.lib  "C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1A\Lib\x64\"
    Copy /y C:\WinDDK\7600.16385.1\lib\win7\i386\ddraw.lib  "C:\Program Files (x86)\Windows Kits\10\Lib\10.0.19041.0\km\x86\"
    Copy /y C:\WinDDK\7600.16385.1\lib\win7\amd64\dxguid.lib  "C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1A\Lib\x64\"
    Copy /y C:\WinDDK\7600.16385.1\lib\win7\i386\dxguid.lib  "C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1A\Lib\"
    
    Create folder c:\bin
    Copy c:\Strawberry\c\bin\nm.exe c:\bin\

    Create folder: c:\working\vbox
    checkout vbox project with TortoiseSVN from:
        https://www.virtualbox.org/svn/vbox/trunk
    start "x64 Nativie Tools Command Prompt for VS2019"
        git clone https://github.com/libsdl-org/SDL-1.2.git
    start "x64 Check Build Environment" from DDK
        cd c:\working\vbox
        makecert.exe -r -pe -ss my -eku 1.3.6.1.5.5.7.3.3 -n "CN=MyTestCertificate" mytestcert.cer
        certmgr.exe -add mytestcert.cer -s -r localMachine root
        certmgr.exe -add mytestcert.cer -s -r localMachine root
        certmgr.exe -add mytestcert.cer -s -r localMachine trustedpublisher
        Bcdedit.exe -set TESTSIGNING ON
        shutdown -r -t 0 (reboot)

BUILDING SDL-1.2:
    in include folder rename SDL_config_win32.h to SDL_config.h
    in VisualC folder starts SDL_VS2010.sln with VS2019
    choose Release/x64 configuration and build both projects: SDL and SDLmain
    in VisualC/SDL/x64/release pick out 
        SDL.lib
        SDL.dll
    in VisualC/SDLmain/x64/release pick out 
        SDLmain.lib
    these 3 files will be put into libsdl/lib/

MODIFICATIONS:

    LocalConfig.kmk
        create this file under too folder and fill with content:
            VBOX_PATH_SIGN_TOOLS=C:/WinDDK/7600.16385.1/bin/amd64
            VBOX_INF2CAT=C:/WinDDK/7600.16385.1/bin/selfsign/inf2cat.exe
            VBOX_SIGNING_MODE=test
            VBOX_WITHOUT_HARDENING=1
            VBOX_WITH_WEBSERVICES=0

    Config.kmk
    347: VBOX_WITHOUT_CONTROL_FLOW_GUARD = 1
         ^Uncomment this line to disable control flow guard (this will result in link.exe's /EXTRACT: parameter to fail)

    C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\km\wdm.h
        find EtwSetInformation and disable it with #if 0/#endif
        ^ this is not good but has to be done
    C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1A\Include\OAIdl.Idl
    258: //_VARIANT_BOOL bool;         /* (obsolete)           */
    273: //_VARIANT_BOOL *pbool;       /* (obsolete)           */
    C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1A\Include\OAIdl.h
    445: //_VARIANT_BOOL bool;         /* (obsolete)           */
    460: //_VARIANT_BOOL *pbool;       /* (obsolete)           */
    C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1A\Include\PropIdl.h
    319: //_VARIANT_BOOL bool;

    include/iprt/nt/fix.h
        create this file manually and write the following lines:
        #pragma once

        #ifndef WDK_NTDDI_VERSION
        #define WDK_NTDDI_VERSION
        #endif

        #ifndef _IRQL_requires_same_
        #define _IRQL_requires_same_
        #endif

        #ifndef _IRQL_requires_max_
        #define _IRQL_requires_max_(x)
        #endif

        #ifndef _IRQL_requires_min_
        #define _IRQL_requires_min_(x)
        #endif

        #ifndef _IRQL_requires_
        #define _IRQL_requires_(x)
        #endif

        #ifndef _IRQL_raises_
        #define _IRQL_raises_(x)
        #endif

        #ifndef _IRQL_saves_
        #define _IRQL_saves_
        #endif

        #ifndef _IRQL_restores_
        #define _IRQL_restores_
        #endif

        #ifndef _IRQL_uses_cancel_
        #define _IRQL_uses_cancel_
        #endif

        #ifndef _IRQL_always_function_min_
        #define _IRQL_always_function_min_(x)
        #endif

        #ifndef _IRQL_restores_global_
        #define _IRQL_restores_global_(x,y)
        #endif

        #ifndef _IRQL_saves_global_
        #define _IRQL_saves_global_(x,y)
        #endif

        #ifndef _IRQL_requires_
        #define _IRQL_requires_(x)
        #endif

        #ifndef _Kernel_float_restored_
        #define _Kernel_float_restored_
        #endif

        #ifndef _Kernel_clear_do_init_
        #define _Kernel_clear_do_init_(x)
        #endif

        #ifndef _Frees_ptr_opt_
        #define _Frees_ptr_opt_
        #endif

        #ifndef _In_reads_bytes_opt_
        #define _In_reads_bytes_opt_(x)
        #endif

    include/iprt/nt/nt.h
        put 
            #include<iprt/nt/fix.h>
        under 
            #pragma once            

    src\VBox\HostDrivers\VBoxUSB\win\lib\VBoxUsbLib-win.cpp
    src\VBox\Additions\3D\win\VBoxWddmUmHlp\VBoxWddmUmHlp.h
    src\VBox\Frontends\VBoxBugReport\VBoxBugReportWin.cpp
        put 
            #include<iprt/nt/fix.h>
        before
            everything

    src\VBox\Frontends\VirtualBox\src\globals\UICommon.cpp	
    1009: disable the following line:
        #if 0
          darkPalette.setColor(QPalette::PlaceholderText, disabledColor);
        #endif

    qwindowsvistastyle.dll
        this file is never used, find a dll and rename to this and put into 
        C:\Qt\Qt5.9.0\5.9\msvc2015_64\plugins\styles\
    src\VBox\HostDrivers\Support\win\SUPDrv-win.cpp
        2678: disable following lines
        #if 0
            /* Ditto for the XFG variants: */
            if (   pCfg->Size >= RT_UOFFSET_AFTER(IMAGE_LOAD_CONFIG_DIRECTORY, GuardXFGCheckFunctionPointer)
                && pCfg->GuardXFGCheckFunctionPointer != NULL)
                supdrvNtAddExclRegion(&ExcludeRegions, (uintptr_t)pCfg->GuardXFGCheckFunctionPointer - (uintptr_t)pImage->pvImage, sizeof(void *));
            if (   pCfg->Size >= RT_UOFFSET_AFTER(IMAGE_LOAD_CONFIG_DIRECTORY, GuardXFGDispatchFunctionPointer)
                && pCfg->GuardXFGDispatchFunctionPointer != NULL)
                supdrvNtAddExclRegion(&ExcludeRegions, (uintptr_t)pCfg->GuardXFGDispatchFunctionPointer - (uintptr_t)pImage->pvImage, sizeof(void *));
        #endif

    b.cmd
        create a cmd file named b.cmd in root folder with content:
        cscript configure.vbs --with-libsdl=libSDL 

    libsdl
        mkdir libsdl under root
        libsdl/include is compied from SDL-1.2/include/
        libsdl/lib is filled with compiled 
            SDL.dll
            SDL.lib
            SDLmain.lib

BEFORE CONTINUE, MAKE SURE THAT YOU COMPLETED THE PREVIOUS STEPS!

BUILD VIRTUALBOX:

    start "x64 Nativie Tools Command Prompt for VS2019"
    chcp 437
    (change codepage to en-us,this is required if your Windows is not English version)
    cd c:\working\vbox
    mkdir out\win.amd64\release\
    under vbox folder:
        b.cmd (cscript configure.vbs --with-libsdl=libSDL)
        env.bat
        kmk 
        (debug: kmk KBUILD_TYPE=debug)
    this may take half an hour


RUN VIRTUALBOX:

    in out\win.amd64\release\bin

    start

        comregister.cmd

        loadall.cmd

        virtualbox.exe


