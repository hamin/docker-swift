# escape=`

FROM mcr.microsoft.com/windows/servercore:10.0.20348.1487 AS windows

LABEL maintainer="Swift Infrastructure <swift-infrastructure@forums.swift.org>"
LABEL description="Docker Container for the Swift programming language"

ARG GIT=https://github.com/git-for-windows/git/releases/download/v2.39.2.windows.1/Git-2.39.2-64-bit.exe
ARG PYTHON=https://www.python.org/ftp/python/3.10.10/python-3.10.10-amd64.exe
ARG SWIFT_SNAPSHOT=https://download.swift.org/swift-5.8-branch/windows10/swift-5.8-DEVELOPMENT-SNAPSHOT-2023-01-09-a/swift-5.8-DEVELOPMENT-SNAPSHOT-2023-01-09-a-windows10.exe

# restore the default Windows shell for correct batch processing
SHELL ["cmd", "/S", "/C"]

# Enable Developer Mode.
RUN reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\AppModelUnlock" /t REG_DWORD /f /v "AllowDevelopmentWithoutDevLicense" /d "1"

# Enable Long Paths
RUN reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem" /t REG_DWORD /f /v "LongPathsEnabled" /d "1"

# Install Git.
# See: git-[version]-[bit].exe /SAVEINF=git.inf and /?
COPY git.inf .
RUN                                                                             `
  curl -SLo git.exe %GIT%                                                       `
  && (start /w git.exe /SP- /VERYSILENT /SUPPRESSMSGBOXES /NOCANCEL /NORESTART /CLOSEAPPLICATIONS /FORCECLOSEAPPLICATIONS /LOADINF=git.inf ) `
  && del /q git.exe git.inf

# Install Python.
# See: https://docs.python.org/3.10/using/windows.html
# FIXME: it appears that `PYTHONHOME` and `PYTHONPATH` are unset
COPY unattend.xml .
RUN                                                                             `
  curl -SLo python.exe %PYTHON%                                                 `
  && (start /w python.exe /quiet )                                              `
  && del /q python.exe unattend.xml

# Install Visual Studio Build Tools
RUN                                                                             `
  curl -SLo vs_buildtools.exe https://aka.ms/vs/17/release/vs_buildtools.exe    `
  && (start /w vs_buildtools.exe --quiet --wait --norestart --nocache           `
        --add Microsoft.VisualStudio.Component.Windows11SDK.22000               `
        --add Microsoft.VisualStudio.Component.VC.CMake.Project                 `
        --add Microsoft.VisualStudio.Component.VC.Tools.x86.x64                 `
        --add Microsoft.VisualStudio.Component.VC.Tools.ARM64                   `
        --add Microsoft.VisualStudio.Component.VC.Tools.ARM                     `
      || IF "%EXITCODE%"=="3010" EXIT 0)                                        `
  && del /q vs_buildtools.exe

# Install Swift
RUN                                                                             `
  curl -SLo installer.exe %SWIFT_SNAPSHOT%                                      `
  && (start /w installer.exe /q)                                                `
  && del /q installer.exe

# FIXME: we should use a non-Administrator user
# USER ContainerUser

ENV PYTHONUTF8=1
# Default to powershell
ENTRYPOINT ["C:\\Program Files (x86)\\Microsoft Visual Studio\\2022\\BuildTools\\Common7\\Tools\\VsDevCmd.bat", "-arch=amd64", "-host_arch=amd64", "&&", "cmd", "-nologo"]
