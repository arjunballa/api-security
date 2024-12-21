## Token Exchange Delegation Flow

Alice encounters an issue with the Acme User Application and wants to delegate authorization to the Administrator of the Acme User Application to reproduce the issue using the Acme Admin Application.

### Solution

#### OAuth Roles
- **Subject**: Alice (alice@acme.com).
- **Actor**: Administrator Bob (bob@acme.com). Bob either has the `admin` role or belongs to the `admin` group, depending on the implementation.
- **Subject Client**: Acme User Application.
- **Actor Client**: Acme Admin Application.
- **Authorization Server/OIDC Provider**: Acme OAuth & OIDC Provider.

**Configuration**

```
iss: https://as.acme.com

// Typically, the `aud` claim represents the Resource Server. However, in the case of the `token_exchange`
// flow, the `aud` claim for both `subject_token` and `actor_token` should be the Authorization Server.
// Therefore, `https://as.acme.com` must be configured as one of the audiences.

aud: [https://rs.acme.com, https://as.acme.com]
```

- **Resource Server**: Acme Backend Service used by the Acme User Application and Acme Admin Application (`https://rs.acme.com`).

**Note**: The Acme User Application and Acme Admin Application must share a database so that the Acme Admin Application can retrieve the `subject_token` to perform the `token_exchange` flow with both the `subject_token` and `actor_token`.

---

### Subject Flow (Alice's Side)

- Alice logs in to the Acme User Application through the Acme OAuth & OIDC Provider using the standard OIDC `authorization_code` flow.
- During login, the Acme User Application receives both an `id_token` and an `access_token`. This regular `access_token` can be used at the Resource Server as needed, with the `aud` claim set to `https://rs.acme.com`.
- Alice clicks "Give permission to reproduce," initiating a regular `authorization_code` flow to get the `subject_token`.
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
    &claims={"access_token":{"may_act":{"essential":true,"value":{"sub":"bob@acme.com","groups":["admin-group"]}}}}
    ```
    
    **Authorization Request for Role Actor Implementation**:
    
    ``` 
    same as above
    &claims={"access_token":{"may_act":{"essential":true,"value":{"sub":"bob@acme.com","roles":["admin-role"]}}}}
    ```
    
- The Authorization Server validates the request and issues the `subject_token` (the token request is not shown here for brevity).
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
      "sub": "bob@acme.com",
      "groups": ["admin-group"]
    }
  }
  ```
  
  **Example `subject_token` for Role Actor Implementation**:
  
  ```json
  {
    same as above
    "may_act": {
      "sub": "bob@acme.com",
      "roles": ["admin-role"]
    }
  }
  ```

---

### Actor Flow (Bob's Side)

- Bob logs in to the Acme Admin Application through the Acme OAuth & OIDC Provider using the standard OIDC Authorization Code Flow.
- During login, the Acme Admin Application receives both an `id_token` and an `access_token`. This regular `access_token` can be used at the Resource Server as needed, with the `aud` claim set to `https://rs.acme.com`.
- Bob clicks "Reproduce issue," initiating an `authorization_code` flow to get the `actor_token`.  
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
    "groups": ["admin-group"]
  }
  ```
  
  **Example `actor_token` for Role Actor Implementation**:
  
  ```json
  {
    same as above
    "sub": "bob@acme.com",
    "roles": ["admin-role"]
  }
  ```

- The Acme Admin Application performs the `token_exchange` with both the `subject_token` from the database and the above `actor_token` to get a `delegate_token` that allows Bob to act on Alice's behalf.
- The Authorization Server verifies the signature, validates the `aud`, `exp`, and `nbf` claims of both the `actor_token` and the `subject_token`, and issues the `delegate_token`.

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
      "groups": ["admin-group"]
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
      "roles": ["admin-role"]
    }
  }
  ```
