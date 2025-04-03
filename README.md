keytool -import -trustcacerts -keystore "$JAVA_HOME/lib/security/cacerts" -storepass changeit -noprompt -alias cloudinary -file cloudinary_cert.pem
