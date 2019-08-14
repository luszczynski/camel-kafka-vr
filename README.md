Exposing Services integrating with Kafka using Camel
===========================

This Project uses Camel to integrate with Kafka and showcases different replay capabilities by handling offsets to move forward or backwards. It incorporates 3D views of the Kafka topic and Camel consumer to visually follow events and replays.

![Alt text](assets/swagger.png?raw=true "Title")
![Alt text](assets/3d-viewer.png?raw=true "Title")


To run locally:

1. Download KAFKA
		
2. Start Zookeeper

		bin/zookeeper-server-start.sh config/zookeeper.properties
		
3. Start Kafka Broker

		bin/kafka-server-start.sh config/server.properties
	
4. Create a topic called test
		
		bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
	
5. Run the App
    	
    	mvn spring-boot:run

6. View REST operations with swagger UI    
    	
    	http://localhost:8080/webjars/swagger-ui/2.1.0/index.html?url=/camel/api-docs#/
    
    
7. To clean topic, delete and recreate with no consumers running
    	
		bin/kafka-topics.sh --zookeeper localhost:2181 --delete --topic test

8. Graphical view

		(local)
		http://localhost:8290/
		http://localhost:8290/topicview.html
    

To run in OpenShift:

1. Create a minishift instance

		minishift --profile amq-streams-demo start

2. Login and access my project

		oc login -u system:admin
		oc project myproject

3. Create the necessary secrets for pulling from registry.io

		oc create secret generic registry-io --from-file=.dockerconfigjson=/home/gustavo/.docker/config.json --type=kubernetes.io/dockerconfigjson -n myproject
		oc secrets link default registry-io --for=pull -n myproject
		oc secrets link builder registry-io -n myproject

4. Install strimzi operator

		oc apply -f install/cluster-operator -n myproject
		oc apply -f examples/templates/cluster-operator -n myproject
		oc secrets link strimzi-cluster-operator registry-io --for=pull -n myproject

5. Create a kafka cluster

		oc apply -f kafka/kafka-ephemeral-single.yaml -n myproject
		oc secrets link my-cluster-zookeeper registry-io --for=pull -n myproject

6. Create a new topic

		oc apply -f ocp/kafkatopic.yml -n myproject

7. Update files with your minishift ip

		sed -i "s/REPLACE-ME/$(minishift ip)/" src/main/fabric8/topicview-route.yml
		sed -i "s/REPLACE-ME/$(minishift ip)/" src/main/fabric8/swagger-std-route.yml
		sed -i "s/REPLACE-ME/$(minishift ip)/" src/main/resources/application.properties

8. Make sure fuse image stream is installed

		BASEURL=https://raw.githubusercontent.com/jboss-fuse/application-templates/application-templates-2.1.fuse-000099-redhat-5/
		oc apply -n openshift -f ${BASEURL}/fis-image-streams.json

9. Deploy the VR app

		mvn -P ocp fabric8:deploy

10. Cleaning up 

To clean up, run:

		oc delete all -l app=camel-kafka-vr -n myproject

6. Now Open:

		oc get route --no-headers | grep topicview | awk '{ print $2 }'
		oc get route --no-headers | grep vr-myproject | awk '{ print $2 }'
		oc get route --no-headers | grep vr-svc-strimzi | awk '{ print $2 "/webjars/swagger-ui/2.1.0/index.html?url=/camel/api-docs"}'
