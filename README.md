This repository contains code used in the **Scaling To Infinity with Docker Swarm, Docker Compose and Consul** article series:

* [A Taste of What Is To Come](http://technologyconversations.com/2015/07/02/scaling-to-infinity-with-docker-swarm-docker-compose-and-consul-part-14-a-taste-of-what-is-to-come/)
* [Manually Deploying Services](http://technologyconversations.com/2015/07/02/scaling-to-infinity-with-docker-swarm-docker-compose-and-consul-part-24-manually-deploying-services/)
* [Blue-Green Deployment, Automation and Self-Healing Procedure](http://technologyconversations.com/2015/07/02/scaling-to-infinity-with-docker-swarm-docker-compose-and-consul-part-34-blue-green-deployment-automation-and-self-healing-procedure/)
* [Scaling Individual Services](http://technologyconversations.com/2015/07/02/scaling-to-infinity-with-docker-swarm-docker-compose-and-consul-part-44-scaling-individual-services/)


###### Note for Windows users:

If you run into issues with ansible complaining about executable permissions, try modifying the `Vagrantfile`'s `synced_folder` entry from this:

```ruby
  config.vm.synced_folder ".", "/vagrant"
```

to this:

```ruby
  config.vm.synced_folder ".", "/vagrant", mount_options: [“dmode=700,fmode=600″]
```


# For demonstration purposes

Up the four VMs:
```ruby
vagrant up
```

Login to swarm-master VM
```ruby
vagrant ssh swarm-master
```

Provision the complete environment:
```ruby
ansible-playbook /vagrant/ansible/infra.yml -i /vagrant/ansible/hosts/prod
```

See what’s in our cluster (consul):
- Commandline: `consul members`
- REST: `curl localhost:8500/v1/catalog/nodes | jq .`
- UI: `http://10.100.199.200:8500/ui/`

Docker Swarm connect
```ruby
export DOCKER_HOST=tcp://0.0.0.0:2375
docker info
```

deploy book service backend using Ansible (coming straight from docker hub)
```ruby
ansible-playbook /vagrant/ansible/books-service.yml -i /vagrant/ansible/hosts/prod
```

See where it is deployed to (notice that service backend is deployed to different node than mongo db):
```ruby
docker ps | grep booksservice
```

See what’s located in the database:
```ruby
curl http://10.100.199.200/api/v1/books | jq .
```

Let’s add and see books to the backend-server!
```ruby
curl -H 'Content-Type: application/json' -X PUT -d '{"_id": 1, "title": "My First Book", "author": "John Doe", "description": "Not a very good book"}' http://10.100.199.200/api/v1/books | jq .

curl -H 'Content-Type: application/json' -X PUT -d '{"_id": 2, "title": "My Second Book", "author": "John Doe", "description": "Not a bad as the first book"}' http://10.100.199.200/api/v1/books | jq .

curl -H 'Content-Type: application/json' -X PUT -d '{"_id": 3, "title": "My Third Book", "author": "John Doe", "description": "Failed writers club"}' http://10.100.199.200/api/v1/books | jq .

curl http://10.100.199.200/api/v1/books | jq .
```

Deploy the front-end to bookservices (docker container)
```ruby
ansible-playbook /vagrant/ansible/books-fe.yml -i /vagrant/ansible/hosts/prod
```

See where it is deployed to  (repeat previous step, perform this step again and see containers moving)
```ruby
docker ps | grep booksservice
```

# Demo Jenkins in this cluster 
To allow infrastructure to login to your public docker repo:
```ruby
docker login 
```

Open Jenkins UI:
```ruby
http://10.100.199.200:8080/
```
In Jenkins setup a manual cd node (did not yet come to automate this):

- Click Manage Jenkins > Manage Nodes > New Node
- Name it cd, select Dumb Slave and click OK
- Type /data/jenkins/slaves/cd as Remote Root Directory
- Type 10.100.199.200 as Host
- Select Launch slave agents on Unix machines via SSH
- Click Add* next to **Credentials
- Use vagrant as both Username and Password and click Add
- Click Save





