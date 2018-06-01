Drupal on Kubernetes
====================

Drupal manifests for kubernetes deployment.

1. [Ansible Template Generated Manifests](#ansible-template-generated-manifests)
2. [Ansible Hosts File Configuration](#ansible-hosts-file-configuration)
3. [Sample Data](#sample-data)
4. [Blazemeter Load Testing](#blazemeter-load-testing)
5. [VMware PKS LoadBalancer](#vmware-pks-loadbalancer)
6. [Tectonic Ingress](#tectonic-ingress)
7. [Rolling Deployments](#rolling-deployments)


## Ansible Template Generated Manifests

Uses ansible templates to generate __Drupal__ yaml manifests based on configuration information supplied via an ansible inventory file.

In the __deployments__ folder create a sub-folder that matches the name of the  deployment/stack name (see the example in the __drupal7-example__ folder).

Run the ansible playbook to generate the Kubernetes YAML deployment manifests:

    $ ansible-playbook -i deployments/my-deploy/drupal.conf drupal-template.yml

This will create a set of output manifest files in the __deployment package folder__ that can then be used for deployment to Kubernetes:

From within the __deployment package folder__ package folder apply the generated manifest:

    kubectl apply -f drupal.yml

> You can then access the drupal site via the __NodePort__ specified, using the load balanced ingress address to the k8s cluster (or any worker node): Eg. http://core-ingress.idstudios.io:30200

Front end proxying and ingress via Traefik, Nginx or Tectonic Ingress is left as an excercise for the user.

## Ansible Hosts File Configuration

Fairly self explanatory, adjust to your environment:

### CoreOS Example

    [all:vars]
    drupal_stack_name=drupal7
    drupal_stack_namespace=web
    drupal_docker_image=idstudios/drupal:plain
    drupal_nodeport=30200

    drupal_domain=drupal-internal.idstudios.io

    drupal_files_nfs_server=192.168.1.107
    drupal_files_nfs_path="/idstudios-files-drupal-test"
    drupal_files_volume_size=10Gi

    drupal_db_host=staging-galera-haproxy
    drupal_db_name=drupaldb 
    drupal_db_user=root 
    drupal_db_password=Fender2000

    [template_target]
    localhost

### VMware PKS Example

    [all:vars]
    drupal_stack_name=drupal7
    drupal_stack_namespace=web
    drupal_docker_image=idstudios/drupal:plain

    drupal_domain=drupal-nsx.onprem.idstudios.io
    drupal_service_port=8000

    drupal_files_nfs_server=192.168.1.107
    drupal_files_nfs_path="/idstudios-files-drupal-test"
    drupal_files_volume_size=10Gi

    drupal_db_host=pks-galera-haproxy
    drupal_db_name=drupaldb 
    drupal_db_user=root 
    drupal_db_password=Fender2000

    [template_target]
    localhost

This values supplied in this file will generate the resulting manifest using the ansible j2 template.

__Note__ that __drupal_domain__ is the site domain name used for host header routing via the ingress proxy.

> The __template_target__ inventory group should always contain __localhost__ as the location of the template generation takes place locally.  A future __Helm__ chart will deprecate the use of ansible.

## Sample Data

> Note that with the new image `drupal_docker_image=idstudios/drupal:plain` you do not need to perform the manual data loads, or unpack the sample image files to the NFS share, the image will take care of this automatically at startup.  Replicas has been set to 1 so that the intialization does not enter race, make sure to scale up for load tests.

In the __data__ folder there is a __drupadb_sample.sql__ file that will load a sample drupal data set into a __drupaldb__ database.  This can be done by connecting to the NodePort (CoreOS) or Load Balancer (PKS) that exposes the MariaDB:

    mysql -h <ingress url> --port <node port> -u <mysql user> -p < drupaldb_sample.sql

Eg.

    mysql -h core-ingress.idstudios.io --port 31306 -u root -p < drupaldb_sample.sql

The sample Drupal7 database data is paired with some sample image uploads, that are expected to reside on the shared NFS volume that holds the file uploads (shared among all of the front end instances of Drupal).  These sample files are located in the __data__ directory, as __drupal_files.tar.gz__.

Expand the __drupal_files.tar.gz__ on the host mounted NFS share and then ensure that the deployment configuration references that folder as the shared host mounted NFS share:

In the deployment configuration file, the settings for the NFS mapping are as follows:

    drupal_files_nfs_server=<nfs server>
    drupal_files_nfs_path="/<nfs path to drupal files folder>"
    drupal_files_volume_size=<size of volume allocation> (eg. 10Gi)

> Unfortunately this requires the manual step of pre-configuring the sample files on the target NFS share before drupal stack deployment.

## VMware PKS LoadBalancer

A sample __VMware PKS__ _LoadBalancer_ manifest is created that will create a dedicated load balancer exposing __drupal_service_port__ (SSL termination coming soon).

    kubectl apply -f drupal-pks-lb.yml

And then check for the assigned IP address:

    kubectl get svc

You should see the assigned Drupal LB hosted on the designated __drupal_service_port__.

## Tectonic Ingress

A sample __Tectonic Ingress__ manifest is created that will do host header routing for the __drupal_domain__ over port 80 (SSL termination coming soon).

    kubectl apply -f drupal-tectonic-ingress.yml

## Blazemeter Load Testing

Example [Blazemeter](https://www.blazemeter.com/) scripts are created based on the configured __drupal_domain__ and sample data URLs to exert __moderate__ and __heavy__ load on the __drupal stack__.

    bzt loadtests/med_load.yml

or

    bzt loadtests/heavy_load.yml

> These can be adjusted and extended as needed and serve as examples.

## Rolling Deployments

Simply updating the image to the __themed__ variant is a good way to test rolling deployments.  Manually edit the generated __drupal.yml__ file and change the __image/tag__:

Eg.

        containers:
            - name: test-drupal
              image: idstudios/drupal:themed


And then simply apply the changes:

    kubectl apply -f drupal.yml

> You can use the __idstudios/drupal:themed__ variant to test a deployment.

The deployment will begin a rolling update, as this is what __Deployments__ do by default in Kubernetes.

> Combining the __Blazemeter__ load testing with a __rolling deployment__ really demonstrates what makes Kubernetes so amazing!