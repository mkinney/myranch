# Create a local private registry
Mainly used the steps provided from the book "Getting Started with Containerization" for the first parts. I am setting affinity to "top" server so all registry files stay there. Really should get some better shared storage.

1. Create secrets directory

    `mkdir secrets`

2. Create certs

    `openssl req -newkey rsa:4096 -nodes -sha256 -keyout secrets/domain.key -x509 -days 365 -out secrets/domain.crt`

3. Create http secret

    `openssl rand -hex -out secrets/http.secret 8`

4. Create httpasswd


    ```
    # change XXX to password and YYY to username
    docker run -i httpd /bin/bash -c 'echo XXX | /usr/local/apache2/bin/htpasswd -nBi YYY' > secrets/registry_passwd
    ```

5. Add secrets to RKE

    `kubectl create secret generic registry-secrets --from-file secrets/`

Note: Can see the secrets like this:

    `kubectl describe secret registry-secrets`

6. Add the PV/PVC/Deployment:

    `kubectl create -f private_registry_pv.yml -f private_registry_pvc.yml -f private_registry_d.yml`

7.  Go into Docker Desktop (mac) under "Preferences", "Daemon" and add a new private registry for `top.mikekinney.org:30500` (Note: This will restart docker.)

8. Make sure you can login to your private registry:

    `docker login top.mikekinney.org:30500`


9. Pull a docker image:

    `docker pull ubuntu:bionic`

10. Tag image:

    `docker tag ubuntu:bionic top.mikekinney.org:30500/ubuntu:bionic`

11. Push image:

    `docker push top.mikekinney.org:30500/ubuntu:bionic`
