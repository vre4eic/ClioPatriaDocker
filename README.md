# ClioPatriaDocker

This builds a ClioPatria docker image.

You can use the image directly to start a ClioPatria server in interactive mode or as a daemon (web service).
To easy development, you can also use run it by mounting your local project directory on your host. 
by mounting it as a volume in the docker container as /opt/project/. 



## Example usage
All examples start in /opt/project as the working directory in the container.

### Running ClioPatria in container (non-persistent RDF-store):
This starts a vanilla ClioPatria daemon, exposing the default port. 
Note that all data in the RDF-store will be lost when starting a new container:
```bash
docker run -p 3020:3020 -d jrvosse/cliopatria
```
To run ./run.pl (interactively):
```bash
docker run -p 3020:3020 -it jrvosse/cliopatria run
```
To run the daemon with a cpack (here swish) installed:
```bash
docker run -p 3020:3020 -d jrvosse/cliopatria cpack install swish
```
To run the daemon with RDF files preloaded:
```bash
docker run -p 3020:3020 -d jrvosse/cliopatria rdf_load file1.rdf file2.ttl 
```
If you do not want to run ClioPatria but bash, so you can look around and configure and run interactively:
```bash
docker run -p 3020:3020 -it jrvosse/cliopatria bash
```

### Creating new standard ClioPatria project in host directory (persistent code and RDF-store):
This uses the docker image to create a new ClioPatria project on your host file system, under your USER name.
Note that the data in the RDF-store will be stored on your host file system and will thus survive a killed container:
```bash 
mkdir project
cd project
docker run -ti \
	-v $PWD:/opt/project \
	-v /etc/group:/etc/group:ro \
	-v /etc/passwd:/etc/passwd:ro \
	-u $( id -u $USER ):$( id -g $USER ) \
	jrvosse/cliopatria
```

You can also run the above command in an existing ClioPatria project dir.  
Just remove your daemon.pl and run.pl scripts, during the first run these will be replaced with new versions that 
match the paths of swipl and ClioPatria to those in the docker image, but it should leave the rest of your project code alone.

### Use as base image for other Docker images
You can also use this as a base image to easily build docker images for your own ClioPatria project.
See, for example, my amalgame docker image at https://hub.docker.com/r/jrvosse/amalgame/~/dockerfile/

```docker
FROM jrvosse/cliopatria

WORKDIR $CLIOPATRIA_DIR
RUN ./configure --with-localhost --with-rdfpath
RUN ./run.pl cpack install amalgame

WORKDIR $PROJECT_DIR
```

## Docker repo
This repo is the source repo for the docker image jrvosse/cliopatria:
```bash
docker pull jrvosse/cliopatria
```

See also https://hub.docker.com/r/jrvosse/cliopatria/ .

## Details
The image is based on the standard swipl image, to which the git and wget packages are added.
ClioPatria is installed from git in /opt/ClioPatria using an explicit git commit reference to avoid docker caching issues.

The default entrypoint is to run /opt/ClioPatria/entrypoint.sh daemon
in /opt/project as the working directory.
