Pour créer un compte admin sur sharelatex :

kubectl exec <pod sharelatex> -- /bin/bash -c "cd /var/www/sharelatex; grunt user:create-admin --email yannick.libert@gmail.com"
