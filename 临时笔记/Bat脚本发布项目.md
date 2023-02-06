## 脚本

```bash
@echo off
@title release new version

set privateFile=~\.ssh\ssh_login_priavtekey
set jarFile=./test-api.jar
set jarFileName=test-api.jar

for /f "delims=" %%i in ('call git pull') do set pullResult=%%i

if "%pullResult%" equ "Already up to date." (
    echo code not update, stop release
    goto exit
)

if not %errorlevel% == 0 (
    echo git pull fail
    goto exit
) else echo "git pull success >>>>>>>>> start code package"

call mvn clean install

if not %errorlevel% == 0 (
    echo code package fail
    goto exit
) else echo "code package success >>>>>>>>> start file upload to server"

call scp -i %privateFile% %jarFile% root@124.70.28.3:/jar/%jarFileName%

if not %errorlevel% == 0 (
    echo file upload to server fail
    goto exit
) else echo "file upload to server success >>>>>>>>> start upload success post options"

call ssh -i %privateFile% root@124.70.28.3 "cd /jar;sh jar_restart.sh restart;exit"

if not %errorlevel% == 0 (
    echo upload success post options fail
    goto exit
)

echo sleep 10s post query status
timeout /T 10

call ssh -i %privateFile% root@124.70.28.3 "cd /jar;sh jar_restart.sh status;exit"

echo \n bat script exec success \n
:exit
pause

```
