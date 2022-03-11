---

copyright:
  years: 2022
lastupdated: "2022-03-11"

subcollection: cloud-native-journey

---

{{site.data.keyword.attribute-definition-list}}

# Connectivity to services
{: #cloud-native-service-connectivity}

In this section, you will learn how to add {{site.data.keyword.cloud_notm}}  services to enhance your Kubernetes cluster with extra capabilities in areas such as Watson AI, data, security, and Internet of Things (IoT).


## Journey Map
{: #cloud-native-service-connectivity-map}

![Architecture](images/connectivity/journey-map.png){: class="center"}

## Add {{site.data.keyword.cloud_notm}}  services to IKS cluster 
{: #cloud-native-service-connectivity-to-ibm-cloud}

To add an {{site.data.keyword.cloud_notm}} service to your cluster:

1. Create an [instance of the {{site.data.keyword.cloud_notm}} service](https://{DomainName}/docs/account?topic=account-externalapp#externalapp).

    * Some {{site.data.keyword.cloud_notm}} services are available only in select regions. You can bind a service to your cluster only if the service is available in the same region as your cluster. In addition, if you want to create a service instance in the Washington DC zone, you must use the CLI.
    * **For IAM-enabled services**: You must create the service instance in the same resource group as your cluster. A service can be created in only one resource group that you can't change afterward.
    * Make sure that the service name is in the following regex format. Example permitted names are `myservice` or `example.com`. Unallowed characters include spaces and underscores.
        ```sh
        [a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*
        ```
        {: screen}

2. Check the type of service that you created and make note of the service instance **Name**.

    ```sh
    ibmcloud resource service-instances
    ```
    {: pre}

   Example output:

    ```sh
    NAME                          Location   State     Type               Tags
    <iam_service_instance_name>   <region>   active   service_instance
    ```
    {: screen}

3. Identify the cluster namespace that you want to use to add your service.
    ```sh
    kubectl get namespaces
    ```
    {: pre}

4. Bind the service to your cluster to create service credentials for your service that use the public cloud service endpoint and store the credentials in a Kubernetes secret. If you have existing service credentials, use the `--key` flag to specify the name of the credentials. For IAM-enabled services, the credentials are automatically created with the **Writer** service access role, but you can use the `--role` flag to specify a different service access role. If you use the `--key` flag, don't include the `--role` flag.

    If your service supports private cloud service endpoints, you can manually create the service credentials with the private cloud service endpoint, and then use the `--key` flag to specify the name of your credentials.
    {: tip}

    ```sh
    ibmcloud ks cluster service bind --cluster <cluster_name_or_ID> --namespace <namespace> --service <service_instance_name> [--key <service_instance_key>] [--role <IAM_service_role>]
    ```
    {: pre}

    When the creation of the service credentials is successful, a Kubernetes secret with the name `binding-<service_instance_name>` is created.  

    Example output:

    ```sh
    ibmcloud ks cluster service bind --cluster mycluster --namespace mynamespace --service cleardb
    Binding service instance to namespace...
    OK
    Namespace:         mynamespace
    Secret name:     binding-<service_instance_name>
    ```
    {: screen}

5. Verify the service credentials in your Kubernetes secret.
    1. Get the details of the secret and note the **binding** value. The **binding** value is base64 encoded and holds the credentials for your service instance in JSON format.
        ```sh
        kubectl get secrets binding-<service_instance_name> --namespace=<namespace> -o yaml
        ```
        {: pre}

        Example output
        ```yaml
        apiVersion: v1
        data:
        binding: <binding>
        kind: Secret
        metadata:
          annotations:
            service-instance-id: 1111aaaa-a1aa-1aa1-1a11-111aa111aa11
            service-key-id: 2b22bb2b-222b-2bb2-2b22-b22222bb2222
          creationTimestamp: 2018-08-07T20:47:14Z
          name: binding-<service_instance_name>
          namespace: <namespace>
          resourceVersion: "6145900"
          selfLink: /api/v1/namespaces/default/secrets/binding-mycloudant
          uid: 33333c33-3c33-33c3-cc33-cc33333333c
        type: Opaque
        ```
        {: screen}

    2. Decode the binding value.
        ```sh
        echo "<binding>" | base64 -D
        ```
        {: pre}

        Example output
        ```sh
        {"apikey":"<API_key>","host":"<ID_string>-bluemix.cloudant.com","iam_apikey_description":"Auto generated apikey during resource-key operation for Instance - crn:v1:bluemix:public:cloudantnosqldb:us-south:a/<ID_string>::","iam_apikey_name":"auto-generated-apikey-<ID_string>","iam_role_crn":"crn:v1:bluemix:public:iam::::serviceRole:Writer","iam_serviceid_crn":"crn:v1:bluemix:public:iam-identity::a/1234567890brasge5htn2ec098::serviceid:ServiceId-<ID_string>","password":"<ID_string>","port":443,"url":"https://<ID_string>-bluemix.cloudant.com","username":"123b45da-9ce1-4c24-ab12-rinwnwub1294-bluemix"}
        ```
        {: screen}

    3. Optional: Compare the service credentials that you decoded in the previous step with the service credentials that you find for your service instance in the {{site.data.keyword.cloud_notm}} dashboard.

6. Now that your service is bound to your cluster, you must configure your app to [access the service credentials in the Kubernetes secret](https://{DomainName}/docs/containers?topic=containers-service-binding#adding_app).