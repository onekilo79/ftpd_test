# ftpd_test

A simple Docker image with a predefined test user, based on a simple extension of [Andrew Stilliard's Pure FTPd image](https://github.com/stilliard/docker-pure-ftpd)

This also uses ideas from [Chris Pitt's pure ftp image](https://github.com/thirstybear/ftpd_test)

# Purpose
This is a local developers image to use a ftp server for development and testing in isolation.

It is not a intended to run this image as infrastructure. 

# Changes to the original image

1. Sets the PUBLICHOST to localhost
1. Creates a default user **test** with credentials **test** on the FTP server

## Publichost explained

The image from stillard would start up a server with a public host of ftp.foo.com by default. This means it thinks its own public address is ftp.foo.com.  

When your run stillards image you have to remember to override the value PUBLICHOST every time you create the contaienr else it will never work locally as localhost.


E.G. from Stillard's instructions:

```
docker run -d --name ftpd_server -p 21:21 -p 30000-30009:30000-30009 -e "PUBLICHOST=localhost" stilliard/pure-ftpd:hardened
```

Where **-e "PUBLICHOST=localhost"** has to be changed each and every time to run as localhsot


This version the server will run up expecting itself to be localhost and repsect commands to/from localhost so you can use this locally.

# Usage


## Pull

Pull the image from Docker Hub using

```
	docker pull onekilo79/ftpd_test
```

## Run


### Setup a local directory for the ftp server to use

Running the following will create a file under a directory locally. I have chosen to use the directory **/tmp/ftp/** 

When the ftp server is created it will use **/tmp/ftp** as the start of its own file system.  

The user **test** will have a home folder that external to the ftp server will be **/tmp/ftp/ftpusers/test/**

Creating a file under **/tmp/ftp/ftpusers/test/** will allow you to see it when you log into the ftp server at **test**

```
	mkdir -p /tmp/ftp/ftpusers/test/
	touch /tmp/ftp/ftpusers/test/this_working_oh_hai.txt
```

### Create and start the docker container against the locally created directory


```
	docker run -d --name ftpd_server -p 21:21 -p 30000-30009:30000-30009 -v /tmp/ftp:/hostmount onekilo79/ftpd_test
```

### Run command explained

* Run docker container in the background 

```
	docker run -d 
```

* Naming the container ftpd_server. This allows us to use the **/tmp/ftp** folder as the root of the ftp file system. 

```
	--name ftpd_server 
```


* opening port 21 from the outside world and mapping it to port 21 inside the container.

```
	-p 21:21 
```

* opening ports 30000-30009 from the outside world and mapping it to ports 30000-30009 inside the container.

```
	-p 30000-30009:30000-30009 
```

* mounting the dirctory /tmp/ftp externally to /hostmount directory inside the container

```
	-v /tmp/ftp:/hostmount
```

* using image onekilo79/ftpd_test

```
	onekilo79/ftpd_test
```


# Test the ftp connection

You can log into the server via:

```
	ftp ftp://test:test@localhost
```

## Test the file created above is seen


```
	ls
```
Your date will be different but you should see the file **this_working_oh_hai.txt**

```
-rw-r--r--    1 1000       ftpgroup            0 Oct 22 17:46 this_working_oh_hai.txt
```