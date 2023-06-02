# Android Application Reverse Engineering 
An easy guide to reverse engineering Android apps using APKTool. Learn how to decompile, analyze, and modify APK files to gain insights into their inner workings. Explore the world of Android app security and customization. Unlock the secrets of APK reverse engineering with this in-depth writeup.

## Table Of Content
  * [APKTool Usage (Decompiled APK)](#apktool-usage--decompiled-apk-)
  * [Bypassing Root Detection & SSL Pinning (Frida)](#bypassing-root-detection---ssl-pinning--frida-)
  * [Bypassing Application-Only Trusted User Certificates](#bypassing-application-only-trusted-user-certificates)
  * [Sign the APK (Optional)](#sign-the-apk--optional-)




## APKTool Usage (Decompiled APK)
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

2. Download the latest APKTool JAR file from https://bitbucket.org/iBotPeaches/apktool/downloads/ and rename it to `apktool.jar`
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

## Bypassing Root Detection & SSL Pinning (Frida)
1. Install Frida
```
pip install frida-tools
```
2. Download the Root Detection & SSL Pinning bypass script from https://codeshare.frida.re/@dzonerzy/fridantiroot/

3. Check the app package name 
```
adb shell pm list packages
```

4. Download the frida server from https://github.com/frida/frida/releases (check the version)
```
adb push [frida server file] /data/local/tmp
```

5. Execute the frida server in the adb shell
```
adb shell
su
cd /data/local/tmp
./[frida server file]
```

6. Run the frida script to bypass Root Detection & SSL Pinning
```
frida -l [frida script] -U --no-pause -f [app_package name] 
```
 or
```
frida -l [frida script] -U -f [app_package name]
```

## Bypassing Application-Only Trusted User Certificates
1. Create a network_security_config.xml config file in `<decompiled_folder>/res/xml`
2. Edit the file and add the following script.
```
<network-security-config> 
    <base-config> 
        <trust-anchors> 
            <certificates src="system" /> 
            <certificates src="user" /> 
        </trust-anchors> 
    </base-config> 
</network-security-config>
```
3. Add the line `android:networkSecurityConfig="@xml/network_security_config"` under application tag in `<decompiled_folder>/AndroidManifest.xml`
4. Recompile the APK 
```
apktool b <folder>
```

## Sign the APK (Optional)
1. Install JDK from https://www.oracle.com/java/technologies/downloads/
2. Add binary file `path\Java\jdk-20\bin` to Environment Variables.
3. Open the command prompt in Windows
4. Generate a new key pair and store it in a keystore file 
```
keytool -genkey -v -keystore test.keystore -storepass password -alias android -keypass password -keyalg RSA -keysize 2048 -validity 10000
```
5. Sign the APK file using a keystore and key pair
```
jarsigner.exe -verbose -keystore test.keystore -storepass password -keypass password /path/<file.apk> android
```
