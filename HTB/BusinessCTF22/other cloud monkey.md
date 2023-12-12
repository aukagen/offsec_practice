 ```
 spec:

   ...

-  ldap_cacert_secret: <resourcename>-ldap-ca-cert

+  ldap_cacert_secret: <resourcename>-custom-certs

+  bundle_cacert_secret: <resourcename>-custom-certs

 ```

	
	To create the secret, you can use the command below:

 

 ```sh

-# kubectl create secret generic <resourcename>-ldap-ca-cert --from-file=ldap-ca.crt=<PATH/TO/YOUR/CA/PEM/FILE>

+# kubectl create secret generic <resourcename>-custom-certs \

+    --from-file=ldap-ca.crt=<PATH/TO/YOUR/CA/PEM/FILE>  \

+    --from-fle=bundle-ca.crt=<PATH/TO/YOUR/CA/PEM/FILE>

 ```
 
 
 ```+        postgres_label_selector: "app.kubernetes.io/instance=postgres-{{ deployment_name }}"```
 
 
 certs
 ```
  

             ssl_certificate /etc/nginx/pki/web.crt;

             ssl_certificate_key /etc/nginx/pki/web.key;

+            ssl_session_cache shared:SSL:50m;

+            ssl_session_timeout 1d;

+            ssl_session_tickets off;

+            ssl_ciphers PROFILE=SYSTEM;

+            ssl_prefer_server_ciphers on;

         {% else %}

             listen 8052 default_server;

         {% endif %}
 ```
 
 
 __something to do with the outdated v1beta1 api__
 
 ```
 apiVersion: apps/v1
kind: Deployment
metadata:
  name: awx-operator
```
