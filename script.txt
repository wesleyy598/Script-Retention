@echo off
setlocal

set "baseFolderPath=C:\BackupBD\BACKUP\WIN-KC0HCHPBHDF"
set "logFolder=C:\Script_Retention\log"

set "foldersToClean="
set "foldersToClean=%foldersToClean% ALTERDATA_CONT\FULL"
set "foldersToClean=%foldersToClean% ALTERDATA_CONT\LOG"
set "foldersToClean=%foldersToClean% ALTERDATA_PACK\FULL"
set "foldersToClean=%foldersToClean% ALTERDATA_PACK\LOG"
set "foldersToClean=%foldersToClean% Gestao_TI\FULL"
set "foldersToClean=%foldersToClean% Gestao_TI\LOG"

set "daysThreshold=15"

for /f "tokens=2 delims==." %%a in ('wmic os get localdatetime /value') do set "datetime=%%a"
set "dateStamp=%datetime:~6,2%%datetime:~4,2%%datetime:~0,4%"
set "timeStamp=%datetime:~8,2%%datetime:~10,2%%datetime:~12,2%"

set "result=nenhum"
set "logContent="

for %%F in (%foldersToClean%) do (
    set "folderPath=%baseFolderPath%\%%F"

    forfiles /p "%folderPath%" /s /m * /d -%daysThreshold% /c "cmd /c (echo Deletando: @path && del @path && set result=sucesso) || (set result=erro)"
    forfiles /p "%folderPath%" /s /m * /d -%daysThreshold% /c "cmd /c set logContent=!logContent!Deletado: @path^r^necho."
)

set "logFileName=%dateStamp%_%timeStamp%_%result%"
set "logFilePath=%logFolder%\%logFileName%.txt"

echo %logContent% > %logFilePath%

rem Exclui arquivos na pasta de logs com mais de 15 dias
forfiles /p "%logFolder%" /s /m *.txt /d -%daysThreshold% /c "cmd /c del @path"

echo Processo de limpeza concluído. Detalhes no arquivo: %logFilePath%
