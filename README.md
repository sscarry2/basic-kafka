Run Kafka
===

1. Enter workshop directory
$ cd chapter01-basic-knowledges/1.5-basic-kafka/01-run-kafka

2. Create namespace
$ kubectl apply -f 00-namespace.yml 
namespace/basic-kafka created

3. Check namespace exists
$ kubectl get ns
NAME              STATUS   AGE
basic-kafka       Active   21s
default           Active   23h
ingress-nginx     Active   15h
kube-node-lease   Active   23h
kube-public       Active   23h
kube-system       Active   23h

4. Create zookeeper deployment
$ kubectl apply -f 01-deployment-zk.yml 
deployment.apps/zk1 created
deployment.apps/zk2 created
deployment.apps/zk3 created

5. Check zookeeper deployment
$ kubectl get po -n basic-kafka
NAME                   READY   STATUS              RESTARTS   AGE
zk1-76cc547698-jhngx   0/1     ContainerCreating   0          21s
zk2-7bb59d6788-rc8s5   0/1     ContainerCreating   0          21s
zk3-566db54d6b-g579s   0/1     ContainerCreating   0          21s

* Wait until STATUS = Running and READY = 1/1
zk1-76cc547698-jhngx   1/1     Running   0          57s
zk2-7bb59d6788-rc8s5   1/1     Running   0          57s
zk3-566db54d6b-g579s   1/1     Running   0          57s

6. Create zookeeper service
$ kubectl apply -f 02-service-zk.yml
service/zk1 created
service/zk2 created
service/zk3 created

7. Check zookeeper service
$ kubectl get svc -n basic-kafka
NAME   TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                      AGE
zk1    ClusterIP   None         <none>        2181/TCP,2888/TCP,3888/TCP   26s
zk2    ClusterIP   None         <none>        2181/TCP,2888/TCP,3888/TCP   26s
zk3    ClusterIP   None         <none>        2181/TCP,2888/TCP,3888/TCP   26s

8. Create kafka deployment
$ kubectl apply -f 03-deployment-kfk.yml
deployment.apps/kfk1 created
deployment.apps/kfk2 created
deployment.apps/kfk3 created

9. Check kafka deployment
$ kubectl get po -n basic-kafka
NAME                    READY   STATUS              RESTARTS   AGE
kfk1-86886b6b84-xkfh2   0/1     ContainerCreating   0          14s
kfk2-5b69dfcdb4-kwwk2   0/1     ContainerCreating   0          14s
kfk3-6d4c8874c6-ll7nh   0/1     ContainerCreating   0          14s
zk1-76cc547698-jhngx    1/1     Running             0          12m
zk2-7bb59d6788-rc8s5    1/1     Running             0          12m
zk3-566db54d6b-g579s    1/1     Running             0          12m

* Wait until STATUS = Running and READY = 1/1
NAME                    READY   STATUS    RESTARTS   AGE
kfk1-86886b6b84-xkfh2   1/1     Running   0          69s
kfk2-5b69dfcdb4-kwwk2   1/1     Running   0          69s
kfk3-6d4c8874c6-ll7nh   1/1     Running   0          69s
zk1-76cc547698-jhngx    1/1     Running   0          13m
zk2-7bb59d6788-rc8s5    1/1     Running   0          13m
zk3-566db54d6b-g579s    1/1     Running   0          13m

10. Create kafka service
$ kubectl apply -f 04-service-kfk.yml
service/kfk1 created
service/kfk2 created
service/kfk3 created

11. Check kafka service
$ kubectl get svc -n basic-kafka
NAME   TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                      AGE
kfk1   ClusterIP   None         <none>        9092/TCP                     28s
kfk2   ClusterIP   None         <none>        9092/TCP                     28s
kfk3   ClusterIP   None         <none>        9092/TCP                     28s
zk1    ClusterIP   None         <none>        2181/TCP,2888/TCP,3888/TCP   14m
zk2    ClusterIP   None         <none>        2181/TCP,2888/TCP,3888/TCP   14m
zk3    ClusterIP   None         <none>        2181/TCP,2888/TCP,3888/TCP   14m

12. Create client-util pod
$ kubectl apply -f 05-client-util.yml
pod/client-util created

13. Exec into client-util pod
$ kubectl exec -it client-util -n basic-kafka -- bash

14. Run kafkacat -L to list brokers and topics
# kafkacat -b "kfk1,kfk2,kfk3" -L
Metadata for all topics (from broker -1: kfk1:9092/bootstrap):
 3 brokers:
  broker 2 at kfk2:9092
  broker 3 at kfk3:9092
  broker 1 at kfk1:9092
 0 topics:

Play Consumer-Producer
===

Kafkacat URL
https://github.com/edenhill/kafkacat

1. Start Producer using kafkacat, and send some messages
$ kafkacat -P -b "kfk1,kfk2,kfk3" -t "mytopic"

$ message1
$ message2
$ message3

2. Open another tab in terminal, and exec into client-util
$ kubectl exec -it client-util -n basic-kafka -- bash
root@client-util:/#

3. Start Consumer in the open terminal
$ kafkacat -C -b "kfk1,kfk2,kfk3" -t "mytopic"

4. Exit Consumer by (Ctrl + c)

5. Exit client-util 
$ exit

Play Consumer Group
===

Kafkacat URL
https://github.com/edenhill/kafkacat

1. Start Producer and send some messages
$ kafkacat -P -b "kfk1,kfk2,kfk3" -t "mytopic1"

$ message1
$ message2
$ message3

2. Open another tab in terminal, then exec into client-util shell
$ kubectl exec -it client-util -n basic-kafka -- bash
root@client-util:/#

3. Start Consumer 1 in new terminal
$ kafkacat -C -b "kfk1,kfk2,kfk3" -G mygroup mytopic1

4. Open another tab in terminal, then exec into client-util shell
$ kubectl exec -it client-util -n basic-kafka -- bash
root@client-util:/#

5. Start Consumer 2 in new terminal
$ kafkacat -C -b "kfk1,kfk2,kfk3" -G mygroup mytopic1

6. Send some more messages in Producer
$ message4
$ message5
$ message6
$ message7
$ message8

7. Check each Consumers will receive new messages

8. Exit from Consumer using (Ctrl + c)

9. When Consumer exit, see the rebalance process happen

10. Exit from all consumer using (Ctrl + c)

11. Exit from all client-util
$ exit

12. Cleanup workshop
$ kubectl delete ns basic-kafka






