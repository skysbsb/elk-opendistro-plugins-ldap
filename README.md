# elk-opendistro-plugins-ldap
Baixei do https://opendistro.github.io/for-elasticsearch-docs/assets/examples/ldap-example.zip 

Doc: https://opendistro.github.io/for-elasticsearch-docs/docs/security/configuration/ldap/

# LDAP
You can access the administration tool at https://localhost:6443. Acknowledge the security warning and log in using cn=admin,dc=example,dc=org and changethis.

* psantos is in the Administrator and Developers groups. jroe and jdoe are in the Developers group. The security plugin loads these groups as backend roles.
* roles_mapping.yml maps the Administrator and Developers LDAP groups (as backend roles) to security roles so that users gain the appropriate permissions after authenticating.
* curl -XPUT https://localhost:9200/new-index/_doc/1 -H 'Content-Type: application/json' -d '{"title": "Spirited Away"}' -u psantos:password -
* curl -XGET https://localhost:9200/new-index/_search?pretty -u jroe:password -k
