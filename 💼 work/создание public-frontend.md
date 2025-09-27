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