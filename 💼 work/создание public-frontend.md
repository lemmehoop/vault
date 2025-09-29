/admin/realms/{realm}/authentication/flows/{flowAlias}/executions/flow


```json
{
            "id": "sub-flow",
            "alias": "sub-flow",
            "description": "",
            "providerId": "basic-flow",
            "topLevel": false,
            "builtIn": false,
            "authenticationExecutions": [
                {
                    "authenticator": "idp-auto-link",
                    "authenticatorFlow": false,
                    "requirement": "REQUIRED",
                    "priority": 1,
                    "flowAlias": null,
                    "userSetupAllowed": false,
                    "autheticatorFlow": false
                }
            ]
        }
```


```python
async def main():  
    c = dishka.make_async_container(  
        InfrastructureProvider(),  
        context={Config: Config()},  
    )  
  
    async with c() as rc:  
        client_fetcher = await rc.get(interfaces.ClientFetcher)  
        policy_fetcher = await rc.get(interfaces.PolicyFetcher)  
        policy_updater = await rc.get(PolicyUpdaterAdapter)  
        print(type(policy_fetcher))  
        client = await client_fetcher.fetch_client("devP", "new")  
        print("here")  
        policy = await policy_fetcher.fetch_policy("devP", client.id, "policy-new")  
        policy.description = "from code"  
        policy.config = {  
            "clients": f'["public-frontend", "public-frontend-test"]',  
        }  
        await policy_updater.update_policy(realm_name="devP", client_uuid=client.id, policy=policy)  
  
        # await fetcher.fetch_client_uuid("new")  
  
  
asyncio.run(main())
```



```python
[
    {
        'id': '4c20a21d-4953-4573-a043-cf0ab2daeaaf',
        'requirement': 'REQUIRED',
        'displayName': 'ver',
        'description': '',
        'requirementChoices': [
            'REQUIRED',
            'ALTERNATIVE',
            'DISABLED',
            'CONDITIONAL'
        ],
        'configurable': False,
        'authenticationFlow': True,
        'flowId': '63e081d9-abf9-4a3d-8f13-7e11b5198c5f',
        'level': 0,
        'index': 0
    },
    {
        'id': 'e921ca33-4887-44a6-9a97-43920b1b549a',
        'requirement': 'ALTERNATIVE',
        'displayName': 'Automatically set existing user',
        'requirementChoices': ['REQUIRED', 'ALTERNATIVE', 'DISABLED'],
        'configurable': False,
        'providerId': 'idp-auto-link',
        'level': 1,
        'index': 0
    }
]try:  
    response = await self._client.post(  
        url=self.CREATE_URl.format(realm_name=realm_name),  
        json=TypeAdapter(type=AuthenticationFlowDM).dump_python(flow),  
    )  
    response.raise_for_status()  
except httpx.HTTPError as error:  
    logger.error(f"{self} Http request to keycloak failed: {error}")  
    raise KeycloakError(f"Http error on auth flow creation with {flow.alias=}") from error
```


```yaml
---  
- name: Configuring DevPlatform realm authentication flow  
  hosts: localhost  
  
  vars_files:  
    - config.yaml  
  
  vars:  
    keycloak_auth: &keycloak_auth  
      auth_keycloak_url: "{{ auth.url }}"  
      auth_client_id: "admin-cli"  
      auth_realm: "{{ auth.realm }}"  
      auth_username: "{{ auth.username }}"  
      auth_password: "{{ auth.password }}"  
  
  tasks:  
#    - name: Realm configuring  
#      community.general.keycloak_realm:  
#        <<: *keycloak_auth  
#  
#        realm: "{{ config.realm }}"  
#        enabled: true  
#  
#        state: present  
#    - name: Authentication flow configuring  
#      community.general.keycloak_authentication:  
#        <<: *keycloak_auth  
#  
#        realm: "{{ config.realm }}"  
#        alias: "{{ config.authentication.flowAlias }}"  
#        providerId: "basic-flow"  
#        authenticationExecutions:  
#          - displayName: "{{ config.authentication.subflowAlias }}"  
#            subFlowType: "basic-flow"  
#            requirement: "REQUIRED"  
#            index: "0"  
#          - providerId: "idp-auto-link"  
#            flowAlias: "verify-subflow"  
#            requirement: "ALTERNATIVE"  
#  
#        state: present  
#    - name: IDP configuring  
#      community.general.keycloak_identity_provider:  
#        <<: *keycloak_auth  
#  
#        realm: "{{ config.realm }}"  
#        alias: "{{ config.idp.alias }}"  
#        displayName: "{{ config.idp.displayName }}"  
#        enabled: true  
#        providerId: "oidc"  
#        trustEmail: true  
#        firstBrokerLoginFlowAlias: "{{ config.authentication.flowAlias }}"  
#        config:  
#          fromUrl: "{{ config.idp.config.fromUrl }}"  
#          clientAuthMethod: "client_secret_post"  
#          clientId: "{{ config.idp.config.clientId }}"  
#          clientSecret: "{{ config.idp.config.clientSecret }}"  
#          useJwksUrl: true  
#          validateSignature: true  
#          hideOnLoginPage: true  
#          syncMode: "FORCE"  
#        mappers:  
#          - name: "Username to Login"  
#            identityProviderMapper: "oidc-user-attribute-idp-mapper"  
#            config:  
#              claim: "login"  
#              user.attribute: "username"  
#              syncMode: "FORCE"  
#  
#        state: present  
#      register: idp  
#    - name: Debug result  
#      ansible.builtin.debug:  
#        msg: "idp.resource.{{ idp.end_state.internalId }}"  
#    - name: Permission configure  
#      community.general.keycloak_authz_permission:  
#        <<: *keycloak_auth  
#  
#        realm: "{{ config.realm }}"  
#        client_id: "new"  
#        name: "token-exchange.permission.idp.{{ idp.end_state.internalId }}"  
#        permission_type: resource  
##        scopes:  
##          - "token-exchange"  
#        resources:  
#          - "idp.resource.{{ idp.end_state.internalId }}"  
#        policies:  
#          - Default Policy  
#  
#        state: present  
#    - name: Configure Realm Management client permission for IDP  
#      community.general.keycloak_client:  
#        <<: *keycloak_auth  
#  
#        realm: "{{ config.realm }}"  
#        clientId: "new"  
#        name: "simple new client"  
#        protocol: "openid-connect"  
#        publicClient: false  
#        alwaysDisplayInConsole: true  
#        authorizationServicesEnabled: true  
#        serviceAccountsEnabled: true  
#        implicitFlowEnabled: false  
#        directAccessGrantsEnabled: false  
#        standardFlowEnabled: true  
#        frontchannelLogout: true  
#        authorizationSettings:  
#          clientId: "new"  
#          name: "Authorization settings"  
#          allowRemoteResourceManagement: true  
#          policyEnforcementMode: "ENFORCING"  
#          decisionStrategy: "UNANIMOUS"  
#          resources:  
#            - name: "simple"  
#              displayName: "simple_1"  
#              ownerManagedAccess: true  
#            - name: "idp.resource.{{ idp.end_state.internalId }}"  
#              displayName: "idp.resource.{{ idp.end_state.internalId }}"  
#              ownerManagedAccess: true  
#              type: "IdentityProvider"  
#              scopes:  
#                - name: "token-exchange"  
#  
#        state: present  
    - name: Fetch keycloak token  
      ansible.builtin.uri:  
        url: "{{ auth.url }}/realms/{{ auth.realm }}/protocol/openid-connect/token"  
        method: POST  
        body_format: form-urlencoded  
        body:  
          grant_type: "password"  
          client_id: "admin-cli"  
          username: "{{ auth.username }}"  
          password: "{{ auth.password }}"  
        status_code: 200  
      register: login  
    - name: Debug result  
      ansible.builtin.debug:  
        msg: "{{ login.json.access_token }}"  
    - name: Enable permission for IDP  
      ansible.builtin.uri:  
        url: "{{ auth.url }}/admin/realms/{{ auth.realm }}/identity-provider/instances/{{ /management/permissions"  
        method: POST  
        body_format: form-urlencoded  
        body:  
          grant_type: "password"  
          client_id: "admin-cli"  
          username: "{{ auth.username }}"  
          password: "{{ auth.password }}"  
        status_code: 200
```