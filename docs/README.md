## install

```shell
bundle install
```

## start
server start:
```shell
jekyll serve --host 0.0.0.0  --port 80 --detach
```

## crontab

shell script:

```shell
#!/bin/bash
cd ~/blog
echo "===GIT==="
git pull 
echo "===JEKYLL==="
/usr/local/bin/jekyll build
echo "===TIME==="
time=$(date "+%Y-%m-%d %H:%M:%S")
echo "${time} ok"
echo " "
```

crontab: 
```shell script
*/10 * * * * ~/blog/flush.sh  >> ~/blog/temp.log
```