# Guide to setup self-managed dashboards in EC2 hosted container

## Prerequisite
An AWS managed OpenSearch domain with SAML authentication enabled. [Reference here](https://docs.aws.amazon.com/opensearch-service/latest/developerguide/saml.html
)

Always set up self-managed dashboards version [2.14.0](https://hub.docker.com/layers/opensearchproject/opensearch-dashboards/2.14.0/images/sha256-22692a567107d4d12418ce37c30953dc6975b98c18db82a09f200bb76cc797b4?context=explore) to disable the version check that is causing connectivity issues during B/G. Once the docker container with version 2.14.0 is running, add the following setting to the `opensearch_dashboards.yml` file to ensure the version check is disabled.

```
opensearch.ignoreVersionMismatch: true
```

## Steps to spin up a self-managed dashboards
1. Create an EC2 instance within the identical VPC where the managed service OpenSearch domain is operating to run the self-managed dashboards and capture its endpoint.
2. Create a new Application in your `IDP` with the self-managed dashboards endpoint which would generate a new IDP metadata
3. Copy the new IDP metadata of the newly created application and paste it into the IDP metadata text box found in the `Configure identity provider (IdP)` section within the security configuration tab of the managed service domain in aws console
4. Replace the self-managed dashboards url in the security configuration file with the self-managed dashboards’ endpoint. The purpose of this is to guarantee that after the user is authenticated in IDP, the redirection occurs to the self-managed dashboards instead of the managed service dashboards.

> [!IMPORTANT]
> Customers do not have access to modify the security configuration file hence raise a support ticket to request a change to the self-managed URL endpoint
[Refer here](https://opensearch.org/docs/latest/security/authentication-backends/saml/#minimal-configuration-example). By running this API call ```_opendistro/_security/api/securityconfig```, customer can validate the `kibana_url` changes in security configuration file.

5. Install docker and its dependencies on the EC2 instance
6. Use the attched `docker-compose.yml` file and run the self-managed opensearch dashboards
7. After the container is up and running, access it by using the command `docker exec -it <CONTAINER ID> bash` and then modify the `opensearch_dashboards.yml` file by adding the SAML specific attributes. Once the modifications are made, restart the container using `docker restart <CONTAINER ID>`. Find the sample `opensearch_dashboards.yml` attached for reference. [Additional Reference here](https://opensearch.org/docs/latest/security/authentication-backends/saml/#opensearch-dashboards-configuration)
8. Post container restart you can access the self-managed OpenSearch Dashboards by hitting the EC2 endpoint with port 5601. By doing so, you can conveniently view and interact with all the saved objects in accordance with the Fine-Grained Access Control settings and SAML authentication.
> [!CAUTION]
> If the endpoint is transitioned to self-managed dashboards and the customer intends to revert to the managed service dashboards endpoint, they must repeat the same procedure, which involves altering the kibana_url in the security configuration file back to the managed service dashboards endpoint. Until this change is made, the managed service dashboards endpoint will remain inaccessible.

> [!NOTE]
> When using docker in EC2 instance, the self-managed OpenSearch Dashboards cannot be accessed over the internet. It is only accessible by machines within the same VPC.

## SAML Reference Links
* https://docs.aws.amazon.com/opensearch-service/latest/developerguide/saml.html
* https://opensearch.org/docs/latest/security/authentication-backends/saml/
* https://www.youtube.com/watch?v=TgnHBz4i63M
* https://www.youtube.com/watch?v=liJO_jOiIF8
* https://opster.com/guides/opensearch/opensearch-security/how-to-set-up-single-sign-on-using-saml/
