Drupal7 on Kubernetes
=====================

Drupal manifests for kubernetes deployment.

## Ansible Template Approach

Uses ansible templates to generate __Drupal7__ yaml manifests based on configuration information supplied via an ansible inventory file.

In the __deployments__ folder create a sub-folder that matches the name of the  deployment/stack name (see the example in the __drupal7-example__ folder).

Run the ansible playbook to generate the Kubernetes YAML deployment manifests:

    $ ansible-playbook -i deployments/my-deploy/drupal.conf drupal-template.yml

This will create a set of output manifest files in the __deployment package folder__ that can then be used for deployment to Kubernetes:

From within the __deployment package folder__ package folder apply the generated manifest:

    kubectl apply -f drupal.yml

> You can then access the drupal site via the __NodePort__ specified, using the load balanced ingress address to the k8s cluster (or any worker node): Eg. http://core-ingress.idstudios.io:30200

Front end proxying and ingress via Traefik, Nginx or Tectonic Ingress is left as an excercise for the user.

## Ansible Drupal Template Configuration File

Fairly self explanatory, adjust to your environment:

    [all:vars]
    drupal_stack_name=drupal7
    drupal_stack_namespace=web
    drupal_docker_image=idstudios/drupal7-docker:plain
    drupal_nodeport=30200

    drupal_files_nfs_server=192.168.1.107
    drupal_files_path="/idstudios-files-drupal-test"
    drupal_files_volume_size=10Gi

    drupal_db_host=staging-galera-haproxy
    drupal_db_name=drupaldb 
    drupal_db_user=root 
    drupal_db_password=Fender2000

    [template_target]
    localhost

This values supplied in this file will generate the resulting manifest using the ansible j2 template.