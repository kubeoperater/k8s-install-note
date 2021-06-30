package=fengmq-console-task  
mkdir -p /tmp/${package}  
cd /tmp/${package}/  
rm -f ${package}.war  
cp ${WORKSPACE}/${package}/target/${package}.war ./  
echo "from hub-dev.example.com/base/centos6.7:tomcatV2" > /tmp/${package}/Dockerfile  
echo "ADD ./${package}.war /export/server/tomcat/webapps/" >> /tmp/${package}/Dockerfile  
if /usr/local/bin/docker build -t  hub-dev.example.com/k8s-test/${package}:$BUILD_NUMBER .   
	then    
				/usr/local/bin/docker login -u jenkins-upload -p passwd hub-dev.example.com      
				/usr/local/bin/docker push hub-dev.example.com/k8s-test/${package}:$BUILD_NUMBER    
				/usr/local/bin/docker rmi  hub-dev.example.com/k8s-test/${package}:$BUILD_NUMBER
fi
