How To resolve SSL issue with hosted IP.

SSL issues occurr since you are using the default WSO2IS package which comes with a self-signed certificate.

The default certificate's CN is localhost, but you are communicating over an IP with the server. Therefore, directly importing the WSO2IS certificate into the .Net application will not address the issue.

To overcome this issue in your development environment, you can create a self-signed certificate with a custom hostname and use it to communicate with WSO2IS.

In the server where WSO2IS is running, goto <IS_HOME>/repository/resources/security directory and execute the following commands.

keytool -genkey -alias wso2carbon -keyalg RSA -keysize 2048 -keystore newkeystore.jks -dname "CN=localhost,OU=WSO2,O=WSO2,L=Mountain View,ST=CA,C=US" -storepass wso2carbon -keypass wso2carbon -validity 3600 -ext SAN=dns:wso2is.dev

keytool -importkeystore -srckeystore newkeystore.jks -destkeystore newkeystore.p12 -srcstoretype JKS -deststoretype PKCS12 -deststorepass wso2carbon -srcstorepass wso2carbon

keytool -export -alias wso2carbon -keystore wso2carbon.jks -file newCert.pem -storepass wso2carbon

keytool -import -alias newcert -file newCert.pem -keystore client-truststore.jks -storepass wso2carbon
The last command might prompt the following message, type "yes" and press enter for it.

Certificate already exists in keystore under alias <wso2carbon>
Do you still want to add it? [no]:
Now open the <IS_HOME>/repository/conf/deployment.toml file and find the section with "[keystore.primary]". You have to replace all 03 lines there, with following block.

[keystore.primary]
file_name = "newkeystore.jks"
password = "wso2carbon"

[keystore.internal]
file_name = "wso2carbon.jks"
Now Restart WSO2 IS.

Now go to the machine where your .Net application is running and follow the steps provided in [1] to import the certificate of WSO2IS into that machine. The relevant certificate store can be found at <IS_HOME>/repository/resources/security/newkeystore.p12. If it asks for a password while importing the store, you can use wso2carbon.

Later you have to add a hostname mapping for wso2is.dev hostname with your server IP. You can follow the steps provided in [2].

e.g: if your server IP is 52.22.2.3, the following line can be added to the hosts file.

52.22.2.3 wso2is.dev
Now configure your .Net application to use wso2is.dev as the hostname for connecting to WSO2IS instead of the IP.

Then please test your application again.


[1] https://www.thewindowsclub.com/manage-trusted-root-certificates-windows
[2] https://gist.github.com/zenorocha/18b10a14b2deb214dc4ce43a2d2e2992