Steps to set up a 3 replica set of mongo DB instances on a Mac laptop:

- Clone this repository somewhere

- Start boot2docker

boot2docker start

You'll see something like:

sr11-a7d8787bcf:mongodb mrmarco$ boot2docker start
2014/11/14 16:08:28 Waiting for VM to be started...

2014/11/14 16:08:29 Started.
2014/11/14 16:08:29 To connect the Docker client to the Docker daemon, please set:
2014/11/14 16:08:29     export DOCKER_HOST=tcp://192.168.59.103:2375

- export the DOCKER_HOST variable as directed by the startup message
export DOCKER_HOST=tcp://192.168.59.103:2375

If you get erros like this one:
2014/11/14 16:11:35 Get http:///var/run/docker.sock/v1.13/containers/json: dial unix /var/run/docker.sock: no such file or directory

You'll need to export the DOCKER_HOST variable.

- Navigate to the repository directory

- Build the image
docker build -t <image-name> .

- Run a new container with the image
docker run -P --name mongo_instance_XYZ -d mongodb --replSet rs0
(You can replace the name with whatever scheme you prefer, and whatever replication set ID you prefer)

e.g.:

docker run -P --name mongo_instance_001 -d mongodb --replSet rs0
docker run -P --name mongo_instance_002 -d mongodb --replSet rs0
docker run -P --name mongo_instance_003 -d mongodb --replSet rs0


- Now, log into the first instance using mongo and initiate the replication
mongo <vm's IP>:<instance mapped port>

You can get the VM's IP using:
boot2docker ip 

And the public-facing port using:
docker port mongo_instance_001 27017

Put the two together and log into the mongoDB instance
mongo <vm's IP>:<instance mapped port>

- Initiate Replication:
rs.initiate()

- Update the IP address of the master
If you run:
rs.status()

You'll see that member with _id 0 probably has a name like: 3cca9bca35a8:27017. We need to change that to use its IP address within the private network.

- Find out the IP of the container:
docker inspect mongo_instance_001
and look for the IPAddress entry

- Log back into the container with mongo and run:
cfg = rs.conf()
cfg.members[0].host = "<ip of the container>:27017>"
rs.reconfig(cfg)

- Check that the IP is now as expected:
rs.status()

- Now add the other servers to the replicat set
rs1:PRIMARY> rs.add("172.17.0.59")
{ "ok" : 1 }
rs1:PRIMARY> rs.add("172.17.0.60")
{ "ok" : 1 }

rs.status()

To check replication works, you can log into a secondary server using mongodb
You'll need to:

- export the value of DOCKER_HOST (so docker commands work)
- find the port of the container you want to connect to (docker ps, or docker port)
You'll see something like rs0:SECONDARY
- To run commands, you'll need to tell mongo it's OK by running:
rs.slaveOk()

Now that you are connected to both primary and secondary, insert something into the primary and see if you find it on the secondary.
