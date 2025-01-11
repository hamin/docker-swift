# escape=`

# 10.0.19044.1706 is not yet published, use the 20H2 snapshot as that is
# believed to work better than the newer image.
FROM mcr.microsoft.com/windows/servercore:10.0.19042.1706 AS windows

LABEL maintainer="Swift Infrastructure <swift-infrastructure@forums.swift.org>"
LABEL description="Docker Container for the Swift programming language"

ARG GIT=https://github.com/git-for-windows/git/releases/download/v2.36.1.windows.1/Git-2.36.1-64-bit.exe
ARG PYTHON=https://www.python.org/ftp/python/3.10.4/python-3.10.4-amd64.exe
ARG WIX=https://wixtoolset.org/downloads/v3.14.0.6526/wix314.exe

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
        --add Microsoft.VisualStudio.Component.VC.ATL                           `
        --add Microsoft.VisualStudio.Component.VC.ATL.ARM                       `
        --add Microsoft.VisualStudio.Component.VC.ATL.ARM64                     `
        --add Microsoft.VisualStudio.Component.VC.CMake.Project                 `
        --add Microsoft.VisualStudio.Component.VC.Tools.x86.x64                 `
        --add Microsoft.VisualStudio.Component.VC.Tools.ARM64                   `
        --add Microsoft.VisualStudio.Component.VC.Tools.ARM                     `
      || IF "%EXITCODE%"=="3010" EXIT 0)                                        `
  && del /q vs_buildtools.exe
COPY InstallModules.cmd .
RUN InstallModules.cmd && del /q InstallModules.cmd

RUN                                                                             `
  curl -SLo wix.exe %WIX%                                                       `
  && (start /w wix.exe /install /quiet /norestart )                             `
  && del /q wix.exe

# FIXME: we should use a non-Administrator user
# USER ContainerUser

ENV PYTHONUTF8=1
# Default to powershell
CMD ["powershell.exe", "-nologo", "-ExecutionPolicy", "Bypass"]
