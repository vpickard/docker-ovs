This is a fork from great work done by Dave Tucker. Thanks Dave!
================================================================
https://github.com/dave-tucker/docker-ovs

docker-ovs
==========

docker-ovs creates [Open vSwitch](http://openvswitch.org) containers for Docker.

The steps below will create a Docker container running OVS 2.4.0, with both
the ovsschema, and the hardwarevtep schema.


Steps to build and start ovs 2.4 docker image with both ovsschema and hardwarevtep schema
=========================================================================================

git clone https://github.com/vpickard/docker-ovs.git

docker pull socketplane/openvswitch:2.4.0

sudo docker build -t ovs-hwvtep


cid=$(docker run -idt --cap-add NET_ADMIN -p 6641:6640 ovs-hwvtep)

docker exec $cid ovsdb-tool create /etc/openvswitch/vtep.db /usr/local/share/openvswitch/vtep.ovsschema

docker exec $cid supervisorctl stop ovsdb-server

docker exec $cid supervisorctl start ovsdb-server-vtep

docker exec $cid ovs-vsctl add-br br-vtep

docker exec $cid ovs-vsctl add-port br-vtep eth0

docker exec $cid vtep-ctl add-ps br-vtep

docker exec $cid vtep-ctl add-port br-vtep eth0

docker exec $cid vtep-ctl set Physical_Switch br-vtep tunnel_ips=192.168.254.20

docker exec $cid supervisorctl start ovs-vtep

docker exec $cid vtep-ctl set-manager ptcp:6640

Some useful commands
=====================
docker exec $cid vtep-ctl get-manager

docker exec $cid ovsdb-client dump unix:/usr/local/var/run/openvswitch/db.sock hardware_vtep
