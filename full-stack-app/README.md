This app was splitted in folders for better understanding and It's based on th following article: https://www.callicoder.com/deploy-spring-mysql-react-nginx-kubernetes-persistent-volume-secret/ 

Original repo for this app is: https://github.com/callicoder/spring-security-react-ant-design-polls-app 

We already have a nfs-provisiong configured, so we are going to use it for MySQL.  

kubectl create secret generic mysql-user-pass --from-literal=username=callicoder --from-literal=password=c@ll1c0d3r
secret/mysql-user-pass --namespace=full-stack-app

kubectl create secret generic mysql-db-url --from-literal=database=polls --from-literal=url='jdbc:mysql://polling-app-mysql:3306/polls?useSSL=false&serverTimezone=UTC&useLegacyDatetimeCode=false'
secret/mysql-db-url --namespace=full-stack-app