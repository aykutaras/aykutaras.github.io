Fun way to install jekyll on Windows, you need docker. 

First install docker with Boot2Docker. (http://docs.docker.com/installation/windows/)

Folder sharing part in Boot2Docker readme
https://github.com/boot2docker/boot2docker/blob/master/README.md#folder-sharing

docker run -d -p 4000:4000 --name="jekyll" --volumes-from source grahamc/jekyll server --watch


https://github.com/grahamc/docker-jekyll/blob/master/Dockerfile