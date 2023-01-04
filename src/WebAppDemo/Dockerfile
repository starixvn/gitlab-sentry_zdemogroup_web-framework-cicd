# escape=`
FROM hub.nexdev.net/base/rocketor/buildservice AS builder

ARG TRANSFORM_CONFIG
ARG ROCKETOR_API
ARG REDMINE_IDS

SHELL ["cmd", "/S", "/C"]

WORKDIR /build

# Copy files *.sln, *.csproj, packages.config to restore nuget packages
COPY *.sln .
COPY src\WebAppDemo\*.csproj src\WebAppDemo\
COPY src\WebAppDemo\packages.config src\WebAppDemo\

RUN nuget.exe restore WebAppDemo.sln

# Copy source codes
COPY src src\

RUN MSBuild.exe "src\WebAppDemo\WebAppDemo.csproj" `
    /t:Build;WebPublish /m `
    /p:DebugType=None `
    /p:Configuration=Release `
    /p:WebPublishMethod=FileSystem `
    /p:TransformWebConfigEnabled=False `
    /p:LocalizedBuild=true `
    /p:AutoParameterizationWebConfigConnectionStrings=False `
    /p:MarkWebConfigAssistFilesAsExclude=false `
    /p:publishUrl="C:\Compiled" `
    /consoleloggerparameters:ErrorsOnly

WORKDIR C:\Compiled

# Change SP Version
RUN `
    if defined REDMINE_IDS `
    FOR %P IN ( `
        App_Data\SPNames.%TRANSFORM_CONFIG%.xml, `
        App_Data\SPNames_2.%TRANSFORM_CONFIG%.xml `
    ) `
    DO curl --fail -o C:\sp-result.xml --form file=@"%P" --form "redmineIds=%REDMINE_IDS%" -X POST %ROCKETOR_API% `
        && xcopy C:\sp-result.xml "%P" /Y

# Transform config
RUN echo "%TRANSFORM_CONFIG%" `
    && xcopy "Web.%TRANSFORM_CONFIG%.config" "Web.config" /Y `
    && xcopy "App_Data\SPNames.%TRANSFORM_CONFIG%.xml" "App_Data\SPNames.xml" /Y
#    && xcopy "App_Data\SPNames_2.%TRANSFORM_CONFIG%.xml" "App_Data\SPNames_2.xml" /Y

FROM hub.nexdev.net/base/dotnet/framework/aspnet:4.7.2-windowsservercore-ltsc2019 AS base

WORKDIR C:\inetpub\wwwroot

COPY --from=builder C:\Compiled .

RUN dir .\bin\

ENTRYPOINT ["C:\\LogMonitor\\LogMonitor.exe", "C:\\ServiceMonitor.exe", "w3svc"]
