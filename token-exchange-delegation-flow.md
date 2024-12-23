## Token Exchange Delegation Flow

### Specification
https://datatracker.ietf.org/doc/html/rfc8693

### Use Case
Alice encounters an issue with the Acme User Application and wants to delegate authorization to the Administrator of the Acme User Application to reproduce the issue using the Acme Admin Application.

### OAuth Roles
- **Subject**: Alice (alice@acme.com). This is new OAuth role defined in https://datatracker.ietf.org/doc/html/rfc8693
- **Actor**: Administrator Bob (bob@acme.com). This is new OAuth role defined in https://datatracker.ietf.org/doc/html/rfc8693.

    Alice can authorize Administrator Bob (bob@acme.com) in different ways. She can authorize Bob individually, authorize anyone who belongs to the `admin` group, or authorize anyone with the `admin` role. As a result, Bob can either be an individual directly authorized by Alice, or have the `admin` role, or belong to the `admin` group, depending on the implementation.
- **Subject Client**: Acme User Application. This application consumes some Acme Backend APIs which are protected by OAuth.
- **Actor Client**: Acme Admin Application. This application consumes some Acme Backend APIs which are protected by OAuth.
- **Authorization Server/OIDC Provider**: Acme OAuth & OIDC Provider.

  ```
  Acme OAuth & OIDC Provider Configuration:
  
  "iss": "https://as.acme.com"
  
  // Typically, the `aud` claim represents the Resource Server. However, in the case of the `token_exchange`
  // flow, the `aud` claim for both `subject_token` and `actor_token` should be the Authorization Server.
  // Therefore, `https://as.acme.com` must be configured as one of the audiences.
  
  "aud": ["https://rs.acme.com", "https://as.acme.com"]
  ```

- **Resource Server**: Acme Backend APIs used by the Acme User Application and Acme Admin Application (`https://rs.acme.com`) which is protected by Acme OAuth & OIDC Provider.

### Solution
- When Alice is using Acme User Application in regular way then Acme User Application can consume Acme Backend APIs using regular `access_token` and Authorization Server would be able to authorize the `access_token` in regular OAuth way and `access_token` will have information about only Alice.
- Similarly, when Bob is using Acme Admin Application in regular way then Acme Admin Application can consume Acme Backend APIs using regular `access_token` and Authorization Server would be able to authorize the `access_token` in regular OAuth way and `access_token` will have information about only Bob.
- For Administrator Bob to reproduce the issue Alice is facing using Acme Admin Application, and for Acme Admin Application consume Acme Backend APIs as Alice delegated to Bob, Acme Admin Application would need a special access_token, say `delegate_access_token` (name `delegate_access_token` is not in spec) which has information about Alice, Bob and also information about Alice delegating to Bob.

Let's examine how Alice authorizes Bob and how the Acme Admin Application obtains a special `access_token` on behalf of Bob with Alice-to-Bob delegation.

### Subject Flow (Alice's Side)

- Alice logs in to the Acme User Application through the Acme OAuth & OIDC Provider using the standard OIDC `authorization_code` flow.
- During login, the Acme User Application receives both an `id_token` and an `access_token`. This regular `access_token` can be used at the Resource Server as needed, with the `aud` claim is set to `https://rs.acme.com`.
- Alice clicks "Give permission to reproduce," initiating a regular `authorization_code` flow to get the special accesss_token called `subject_token` which captures information about Alice delegating to Bob or `admin` group or `admin` role.
  - Alice needs to inform the Acme OAuth & OIDC Provider to whom the authorization is being delegated so that the provider can add the `may_act` claim to the `subject_token`. This can be achieved using the `claims` authorization request parameter. Alice can search for specific actors (e.g., Bob) via the Acme User Application interface, or the Acme User Application can assign a role or group in the background, depending on the implementation.
  - The `aud` claim in the `subject_token` must be set to the Authorization Server. This can be achieved using the `resource` authorization request parameter.

    **Authorization Request for Individual Actor Implementation**:
    
    ``` 
    GET https://as.acme.com/authorize
    ?response_type=code
    &client_id=<acme_user_app_client_id>
    &redirect_uri=https://user-app.acme.com/redirect
    &scope=<scopes>
    &code_challenge=<code_challenge>
    &state=<client_created_same_returned_by_as>
    &resource=https://as.acme.com
    &claims={"access_token":{"may_act":{"essential":true,"value":{"sub":"bob@acme.com"}}}}
    ```
    
    **Authorization Request for Group Actor Implementation**:
    
    ``` 
    same as above
    &claims={"access_token":{"may_act":{"essential":true,"value":{"groups":["admin-group"]}}}}
    ```
    
    **Authorization Request for Role Actor Implementation**:
    
    ``` 
    same as above
    &claims={"access_token":{"may_act":{"essential":true,"value":{"roles":["admin-role"]}}}}
    ```
    
- The Authorization Server validates the request and issues the `subject_token` (the `/token` request is not shown here for brevity).
- The Acme User Application stores the `subject_token` in the database to use it in the `token_exchange` flow to get the `delegate_token`, which is explained in later sections.
  
  **Example `subject_token` for Individual Actor Implementation**:
    
  ```json
  {
    "aud": "https://as.acme.com",
    "iss": "https://as.acme.com",
    "exp": "{exp}",
    "scope": "{scopes}",
    "sub": "alice@acme.com",
    "may_act": {
      "sub": "bob@acme.com"
    }
  }
  ```
  
  **Example `subject_token` for Group Actor Implementation**:
  
  ```json
  {
    same as above
    "may_act": {
      "groups": ["admin-group"] // only "sub" claim is defined in spec. "groups" is a custom claim.
    }
  }
  ```
  
  **Example `subject_token` for Role Actor Implementation**:
  
  ```json
  {
    same as above
    "may_act": {
      "roles": ["admin-role"] // only "sub" claim is defined in spec. "roles" is a custom claim.
    }
  }
  ```
  
### Actor Flow (Bob's Side)

- Bob logs in to the Acme Admin Application through the Acme OAuth & OIDC Provider using the standard OIDC Authorization Code Flow.
- During login, the Acme Admin Application receives both an `id_token` and an `access_token`. This regular `access_token` can be used at the Resource Server as needed, with the `aud` claim is set to `https://rs.acme.com`.
- Bob clicks "Reproduce issue," initiating an `authorization_code` flow to get a special access_token called `actor_token`.  
  - The `aud` claim in the `actor_token` must be set to the Authorization Server. This can be achieved using the `resource` authorization request parameter.

    ``` 
    GET https://as.acme.com/authorize
    ?response_type=code
    &client_id=<acme_admin_app_client_id>
    &redirect_uri=https://admin-app.acme.com/redirect
    &scope=<scopes>
    &code_challenge=<code_challenge>
    &state=<client_created_same_returned_by_as>
    &resource=https://as.acme.com
    ```

- The Authorization Server validates the request and issues the `actor_token` (the token request is not shown here for brevity).

  **Example `actor_token` for Individual Actor Implementation**:
  
  ```json
  {
    "aud": "https://as.acme.com",
    "iss": "https://as.acme.com",
    "exp": "{exp}",
    "scope": "{scopes}",
    "sub": "bob@acme.com"
  }
  ```
  
  **Example `actor_token` for Group Actor Implementation**:
  
  ```json
  {
    same as above
    "sub": "bob@acme.com",
    "groups": ["admin-group"] // only "sub" claim is defined in spec. "groups" is a custom claim.
  }
  ```
  
  **Example `actor_token` for Role Actor Implementation**:
  
  ```json
  {
    same as above
    "sub": "bob@acme.com",
    "roles": ["admin-role"] // only "sub" claim is defined in spec. "roles" is a custom claim.
  }
  ```

- The Acme Admin Application performs the `token_exchange` with both the `subject_token` from the database and the above `actor_token` to get a `delegate_token` that allows Bob to act on Alice's behalf.
- The Authorization Server verifies the signature, validates the `aud`, `exp`, and `nbf` claims of both the `actor_token` and the `subject_token`, and issues the `delegate_token`. The validation to be performed and the method to obtain the keys for validation are not defined in the specification and it is out to implementer.

  **Example `delegate_token` for Individual Actor Implementation**:
  
  ```json
  {
    "aud": "https://rs.acme.com",
    "iss": "https://as.acme.com",
    "exp": "{exp}",
    "scope": "{scopes}",
    "sub": "alice@acme.com",
    "act": {
      "sub": "bob@acme.com"
    }
  }
  ```
  
  **Example `delegate_token` for Group Actor Implementation**:
  
  ```json
  {
    same as above
    "sub": "alice@acme.com",
    "act": {
      "sub": "bob@acme.com",
      "groups": ["admin-group"] // custom claim
    }
  }
  ```
  
  **Example `delegate_token` for Role Actor Implementation**:
  
  ```json
  {
    same as above
    "sub": "alice@acme.com",
    "act": {
      "sub": "bob@acme.com",
      "roles": ["admin-role"] // custom claim
    }
  }
  ```

- Acme Admin Application will use the `delegate_access_token` to consume Acme Backend APIs to reproduce issue as Bob on behalf of Alice.

  **Note**: The Acme User Application and Acme Admin Application must share a database so that the Acme Admin Application can retrieve the `subject_token` to perform the `token_exchange` flow with both the `subject_token` and `actor_token`.

