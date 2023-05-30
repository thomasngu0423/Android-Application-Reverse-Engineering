# APKTool_writeup
A comprehensive guide to reverse engineering Android apps using APKTool. Learn how to decompile, analyze, and modify APK files to gain insights into their inner workings. Explore the world of Android app security and customization. Unlock the secrets of APK reverse engineering with this in-depth writeup.

## Usage
1. Save the code to `apktool.bat`
```
@echo off
setlocal
set BASENAME=apktool_
chcp 65001 2>nul >nul

set java_exe=java.exe

if defined JAVA_HOME (
set "java_exe=%JAVA_HOME%\bin\java.exe"
)

rem Find the highest version .jar available in the same directory as the script
setlocal EnableDelayedExpansion
pushd "%~dp0"
if exist apktool.jar (
    set BASENAME=apktool
    goto skipversioned
)
set max=0
for /f "tokens=1* delims=-_.0" %%A in ('dir /b /a-d %BASENAME%*.jar') do if %%~B gtr !max! set max=%%~nB
:skipversioned
popd
setlocal DisableDelayedExpansion

rem Find out if the commandline is a parameterless .jar or directory, for fast unpack/repack
if "%~1"=="" goto load
if not "%~2"=="" goto load
set ATTR=%~a1
if "%ATTR:~0,1%"=="d" (
    rem Directory, rebuild
    set fastCommand=b
)
if "%ATTR:~0,1%"=="-" if "%~x1"==".apk" (
    rem APK file, unpack
    set fastCommand=d
)

:load
"%java_exe%" -jar -Duser.language=en -Dfile.encoding=UTF8 "%~dp0%BASENAME%%max%.jar" %fastCommand% %*

rem Pause when ran non interactively
for /f "tokens=2" %%# in ("%cmdcmdline%") do if /i "%%#" equ "/c" pause
```

2. Download the latest JDK file from https://bitbucket.org/iBotPeaches/apktool/downloads/ and rename it to `apktool.jar`
3. Open the command prompt in Windows
4. Decompile the APK 
```
apktool d <file.apk>
```
5. Use the editor tool such as `Vscode` or `Android Studio` to modify and view the source code.
6. Recompile the APK 
```
apktool b <folder>
```
7. Sign the APK (Optional)
```
keytool -genkey -v -keystore test.keystore -storepass password -alias android -keypass password -keyalg RSA -keysize 2048 -validity 10000
```
```
jarsigner.exe -verbose -keystore test.keystore -storepass password -keypass password /path/<file.apk> android
```
