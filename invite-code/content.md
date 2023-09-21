# Consider a reservation system using invite code and an email to reserve a spot 

# Required Items 

###  1. invite codes needs to be have a limit on usage (once or multiple times) 
- **Problem List**:
    - How many daily users are expected?
    - How should we choose the appropriate database?
    - For how long should multiple usage codes exist? Do they have an expiration criterion?
    
- **Solution & Idea:** 
    -  We could set up an **invite code** generation service and initially create users without any additional information.
    
    - Next, we could initiate a cron job to check if the sum of users and the number from the config is less than the required columns.
    
    - It might be best to use a relational database.
        -  Aurora DB is based on the MySQL InnoDB engine and offers fast backup times
        -  Postgres is also a good choice but has a longer recovery time when the server shuts down
        -  Given the importance of the SLA for the users table, I'm leaning towards Aurora DB.
        
    - If the daily user increased, we could scale our `invite service`,  import a kind of message queue as we insert or modify invite codes list, expiration and activities.As we import Message Queue for handle huge reservation:
        -  Kafka can manage 100 thousand to 1 million requests, scales easily, and provides atomic transactions to prevent data loss (if the ACK level is set to 'all' or '-1'). However, setting up its infrastructure can be challenging and requires many nodes (at least 6 with Zookeeper or 3 with KRaft).
        - Nats is lightweight and user-friendly, but there's a risk of losing some messages.
        -  Rabbit MQ is an excellent choice due to its inbuilt delay queue feature, robust AMQP, and is apt for reservations. However, it can handle only up to 10 thousand requests.
    - If the user count exceeds 10-100 million, we might consider sharding our database. If we aim for more flexibility with the **invite_codes table**, we can easily transition from Aurora to a NoSQL DB like Dynamo or Scylla.

-  Click the sequence diagram below to access the mermaid online editor. For a clearer view, select FULL SCREEN.

[![](https://mermaid.ink/img/pako:eNqNVNtum0AQ_ZURT4nkCxcbG6ty1cR56HMqVaosRQsMsDXs0t3FNo3y750F4kuaVuEB7c59zpzZZyeRKTorR-OvBkWCG85yxSrYCsNNicDFnhsEawVZKQ9bwRojRVPFqLYC6GNpxQUZCC3Jfrxev_poVHue4IoOBpBkSooKhYE9U5zFJWrYc0baPSpWwoG1Gj7FaroGbwKaVzWFK7lojl0kVDbGq4U_gd1SQ4p1KVsbVMOgCSbQ1Cmj_PRjMdP9AZhIoaHL_eYeal4jRcbb0eD0vUBhyxhBSbV2haPS1FRTplBIAwpLydKugr7r6x67tr80SioGm7sVqTMwBQL0QHXJLW7jHIUNAlxbVLr0sK4V1kxdYa0hkwpiZpKCxJTFwBkbaoOKMxbCMxy981Pv3Over3T894C44AYSGg_8lLF302JJk4a4lMnutitE4OGKCbqpaToW9xEcpNrZeqwdsSBplCIitVQly7EvQWFCEObxjT-fj6D_ue7tR5B8NIx639zBt-PQa04TsigUVGLFRDvAgUeuzRmOk_YCF3IVRDWD6QVZLvRPiaIjcRNYNx7DRf7vcb8FMWFl0pSWeKfcQ5Yr5IqOU_FFMZ8_goNGopCyS0pdGmkRsYAP3KDTQPoeDMq7hgeR9rj18SnAu9NwXfp5_5nG20a_y27eh4ITOfsVIfrSc8GJQHE_j68b3feeFJjsQJINrZe2FjK7BsTYsgekhi0l4ugLpHQrEjuSMwvfjOvUoDNyKlQV4yk9ac9WvHVoDyvcOis6ppixpjRbZyteyNRu5CPFdlZGNThyegiHF9BZZazUJ-lDyo1UJ6F9DZCuz45pa_t-5sQ-CkkLkPHcyhtVkrgwptar6dSqJzk3RRNPEllNNU8LwqzYR-E09MMl8wMMFwGbB0GaxF60zPyZl6UL1_OZ8_IycmombNSjsxr77nLi-_Nl4EW-N4-8xXzktCRfLCZ0m4WzKIwWXhTMyO23lNSJNwmWi8glp2jph2Ho-l3AH53S9v7yB-f3A14?type=png)](https://mermaid.live/edit#pako:eNqNVNtum0AQ_ZURT4nkCxcbG6ty1cR56HMqVaosRQsMsDXs0t3FNo3y750F4kuaVuEB7c59zpzZZyeRKTorR-OvBkWCG85yxSrYCsNNicDFnhsEawVZKQ9bwRojRVPFqLYC6GNpxQUZCC3Jfrxev_poVHue4IoOBpBkSooKhYE9U5zFJWrYc0baPSpWwoG1Gj7FaroGbwKaVzWFK7lojl0kVDbGq4U_gd1SQ4p1KVsbVMOgCSbQ1Cmj_PRjMdP9AZhIoaHL_eYeal4jRcbb0eD0vUBhyxhBSbV2haPS1FRTplBIAwpLydKugr7r6x67tr80SioGm7sVqTMwBQL0QHXJLW7jHIUNAlxbVLr0sK4V1kxdYa0hkwpiZpKCxJTFwBkbaoOKMxbCMxy981Pv3Over3T894C44AYSGg_8lLF302JJk4a4lMnutitE4OGKCbqpaToW9xEcpNrZeqwdsSBplCIitVQly7EvQWFCEObxjT-fj6D_ue7tR5B8NIx639zBt-PQa04TsigUVGLFRDvAgUeuzRmOk_YCF3IVRDWD6QVZLvRPiaIjcRNYNx7DRf7vcb8FMWFl0pSWeKfcQ5Yr5IqOU_FFMZ8_goNGopCyS0pdGmkRsYAP3KDTQPoeDMq7hgeR9rj18SnAu9NwXfp5_5nG20a_y27eh4ITOfsVIfrSc8GJQHE_j68b3feeFJjsQJINrZe2FjK7BsTYsgekhi0l4ugLpHQrEjuSMwvfjOvUoDNyKlQV4yk9ac9WvHVoDyvcOis6ppixpjRbZyteyNRu5CPFdlZGNThyegiHF9BZZazUJ-lDyo1UJ6F9DZCuz45pa_t-5sQ-CkkLkPHcyhtVkrgwptar6dSqJzk3RRNPEllNNU8LwqzYR-E09MMl8wMMFwGbB0GaxF60zPyZl6UL1_OZ8_IycmombNSjsxr77nLi-_Nl4EW-N4-8xXzktCRfLCZ0m4WzKIwWXhTMyO23lNSJNwmWi8glp2jph2Ho-l3AH53S9v7yB-f3A14)


## DB Schema

```sql 
 CREATE TABLE users (
    `id` bigint(20) PRIMARY KEY ,
    `email` varchar(30) COMMENT '',
    `password` varchar(64) COMMENT 'using bcrypt algo',
    `invitation_code` varchar(10),
    `on_chain_address` varchar(64) COMMENT 'mapping on chain data',
    `state` tinyint(1) COMMENT '0: init 1: processing activation 
     2: activated 6: deactivated 9: suspend',
    `created_at` int(11), 
    `updated_at` int(11), 
    `integration_priv_key` dedault '' COMMENT 'encrypted, using as private key for ECDH verification request between components',
    `current_auth_admin` bigint(20) COMMENT 'a public key of users in admin table, and it can be updated by admin service, or random select an virtual admin user for update..'
     ...
 )
 
  CREATE TABLE invite_codes (
    `id` bigint(20) PRIMARY KEY ,
    `invite_code` varchar(10),
    `referrer` varchar(6) COMMENT 'referrer of this invite_code',
    `remain_times` int(11),
    `update_version` int(11) COMMENT 'optimistic locking for update with timestamp'
    `state` tinyint(1) COMMENT '0: init 1: processing authentication 
     2: activated 6: deactivated 8: suspend',
    `prcessing_activity_ids` varchar(30) COMMENT 'string list for activate ids',
    `created_at` int(11), 
    `updated_at` int(11), 
     ...
 )
 
 CREATE TABLE invite_code_criteria (
    `id` bigint(20) PRIMARY KEY ,
    `invitation_code_id` bigint(20),
    `invitation_code` varchar(10),
    `state` tinyint(1) COMMENT '0: init 1: processing authentication 
     2: activated 6: deactivated 8: suspend',
    `created_at` int(11), 
    `updated_at` int(11), 
     ...
 )

  CREATE TABLE admin (
     `id` bigint(20) PRIMARY KEY ,
     `acoount`   varchar(30),
     `password`  varchar(64),
     `role` tinyint(1),
     `state` tinyint(1),
     `admin_pub_key` varchar(64) COMMENT 'ECDH for sign message of auth service, with AES(secret store in config/k8s secret/vault...etc ) encrypted',
     `admin_pri_key` varchar(64) COMMENT 'ECDH for sign message of auth service, with AES encrypted',
     ...
 )
```



---

### 2. invite codes should be hard to guess, but also not too hard to type

- **Problem List**:
    - How can we design user-friendly invite codes?
    - How can we make the invite code challenging to guess?
    
- **Solution & Idea:**
    - Consider using a combination of letters (excluding 'O' and 'I') and numbers (2-9). The formula (8+24) ^ 6 results in 1,073,741,824 possible combinations.
    - We could consider encrypting with a salt or using simple encryption methods like the Caesar Cipher.
    - ref1. https://medium.com/@siddhusingh/referral-code-generation-architecture-contention-free-scalable-approach-68ea44ee5fb0
    - ref2. https://developer.aliyun.com/article/843394


--- 

### 3. invite codes are associated with a unique email address - each email can only use one invite code.
- **Problem List:**
     - How can we uniquely associate invite codes with email addresses?
     - What are the expected peak daily users per second?
     - Could operations used to query potentially lead to database deadlocks?
     - Should invite codes be used like a primary or foreign key?
     
- **Solution & Idea:** 
    -  A **relational database** appears optimal as it inherently supports unique indexes, eliminating the need to handle duplicate items or design another normalized email address.
    - Given that user states and on-chain addresses and keys might not change often, the likelihood of encountering a deadlock is minimized.
    - While NoSQL databases like MongoDB offer similar utilities, versions below 4.2 don't support them.
    - If the users table grows, we can consider migrating to a NoSQL DB and sharding tables..

- Additionally, we could migrate invite code information to NoSQL databases like ScyllaDB or DynamoDB, setting invite codes as indices, then map them into Aurora DB. 

---

### 4. invite codes should be trackable, admin should be able to know who the referrer is
   - **Problem List:**
       - It appears we require some form of application performance monitoring and error tracking.
       - How can we optimally establish a relationship between the users table and the invite code table?

   - **Solution & Idea:**
      - Store the referrer's invite code directly in the DB. To retrieve it, simply query the invite_codes referrer column.
      - Implement an Application Performance Management & Monitoring solution.
      - For tracing, consider incorporating open-tracing tools such as Signoz, Loki, or Sentry
      - Personally, Sentry offers superior tracing experiences, is suitable for testing cron jobs, and provides detailed error stacks per request.

---
### 5. design should be able to withstand high traffic and concurrency
- **Problem List**
    - What's the expected Transactions Per Second (TPS)?
    - Which concurrency patterns are optimal?
    - Is there a need for scaling? Should we consider a serverless approach?
    
- **Solution & Idea**
    -  the daily user count isn't exceptionally high, a well-designed EC2 combined with a load balancer (be it AWS, Nginx, Envoy, or APISIX) might be a straightforward solution. This means even if one server goes down, we can still round-robin requests to the remaining active nodes.
    - On the other hand, if we experience significant peak usage, scaling our services becomes essential. A few direct approaches include:

        - Re-evaluating and potentially adjusting our rate limit.
        - Integrating a message queue to manage peak requests. This system would produce messages for peak periods, and dedicated consumers would handle these messages.
        - Incorporating Kubernetes (K8s). With a well-structured service design, scaling becomes feasible when there's a need for additional replicas.
    - The email sending service can also be scaled as needed.
    
---
### 6. design should be hard to hack
- **Problem List**: 
    - There's a necessity to establish services with robust authentication.
    - We must acknowledge server nodes, leveraging both AES256 for encryption and ECDSA for verification.
    - The nodes should have firewall protections in place, such as Linux iptables on EC2, AWS inbound/outbound rules, or node groups in K8s that exclusively accept our service IPs on specific ports.
    
- **Solution & Idea**
    - The authentication service should handle requests and sign them using the admin's key.
    - It's crucial to ensure message protection and verification for security.

[![](https://mermaid.ink/img/pako:eNrNU8uSmzAQ_BWVLrkQjME8K-UqO7tfsLcUFxkNoApIRA9viMv_nhHGrmyyyTmcRM90q-ehC20UB1pRA98cyAaeBOs0G0ktrbADkIOzPWnVay2Zs0q68QS6lgQ_A_oMmhzIx_3-luYR0UCFB8mJ9orGkk8nvdkTq3wYPON44_9KeUcDzeh5ssQZIbtVhDx_fno5EDZ0Sgvbjyt8Ukh8RYAwPgpJGN5-izSDAGk_EEMmLc7MAvkK81-uX-s5rvZtD4sHtLCKIXUtY8kj5E0fjr-JcLj5X3wtAveGeTuoFdw704MMSAeWTGweFOOLf2eACGvWHCM6yazTECwusMZ7Lhq63_RPOz2K4jg1NCDO8JjOO5y3Y9AwKYle_uuZHR4rZ25u_9i5Qy1pQEfQIxMc1_3iBWuKvR-hphUeObTMDbamtbxiqt_2l1k2tLLaQUDdxNHK-jpo1bLBPNBnLqzSD9DPBfD3Qu08-bfVCWNRslGyFZ3HnR4Q7q2dTLXZ-HDYYSvcKWzUuDGC90zb_lxmmyzOChYnkOUJS5OEN6dtWbTxbtvyPNrGjF6vAZ2Y9KrfabXdJWGUF0WZRlGWpbsyCehMqzgJ8zhNy2JXFFEW5XGGrB9KYSFRWMZ5sk2iuMyzLCrifNH7sgR96defIxJZ9A?type=png)](https://mermaid.live/edit#pako:eNrNU8uSmzAQ_BWVLrkQjME8K-UqO7tfsLcUFxkNoApIRA9viMv_nhHGrmyyyTmcRM90q-ehC20UB1pRA98cyAaeBOs0G0ktrbADkIOzPWnVay2Zs0q68QS6lgQ_A_oMmhzIx_3-luYR0UCFB8mJ9orGkk8nvdkTq3wYPON44_9KeUcDzeh5ssQZIbtVhDx_fno5EDZ0Sgvbjyt8Ukh8RYAwPgpJGN5-izSDAGk_EEMmLc7MAvkK81-uX-s5rvZtD4sHtLCKIXUtY8kj5E0fjr-JcLj5X3wtAveGeTuoFdw704MMSAeWTGweFOOLf2eACGvWHCM6yazTECwusMZ7Lhq63_RPOz2K4jg1NCDO8JjOO5y3Y9AwKYle_uuZHR4rZ25u_9i5Qy1pQEfQIxMc1_3iBWuKvR-hphUeObTMDbamtbxiqt_2l1k2tLLaQUDdxNHK-jpo1bLBPNBnLqzSD9DPBfD3Qu08-bfVCWNRslGyFZ3HnR4Q7q2dTLXZ-HDYYSvcKWzUuDGC90zb_lxmmyzOChYnkOUJS5OEN6dtWbTxbtvyPNrGjF6vAZ2Y9KrfabXdJWGUF0WZRlGWpbsyCehMqzgJ8zhNy2JXFFEW5XGGrB9KYSFRWMZ5sk2iuMyzLCrifNH7sgR96defIxJZ9A)

```sql
 CREATE TABLE admin (
     `id` bigint(20) PRIMARY KEY ,
     `acoount`   varchar(30),
     `password`  varchar(64),
     `role` tinyint(1),
     `state` tinyint(1),
     `admin_pub_key` varchar(64) COMMENT 'ECDH for sign message of auth service, with AES encrypted',
     `admin_pri_key` varchar(64) COMMENT 'ECDH for sign message of auth service, with AES encrypted',
     ...
 )
```

- ref1. https://cryptobook.nakov.com/digital-signatures/ecdsa-sign-verify-messages
- ref2. https://github.com/EasonWang01/Introduction-to-cryptography/blob/master/3.3%20ECDSA.md
- ref3. https://gist.github.com/fmolliet/f542f9f2a89f112c54838ea4ccc2f3e9
- ref4. https://gist.github.com/santhosh77h/4ad6f2dfcaad5e23997976c69d2aa7ac
---
# Please design this system, document and **provide reasoning behind** each design decision (and options considered).


## Preliminary Considerations
Before diving into the specific solutions for invite codes, it's essential to evaluate the suitability of the existing database design. The following are my considerations.

## Design Principles
- Each domain should have distinct functionalities.
- Every API or message should be idempotent (冪等) – ensuring that operations can be repeated without changing the result beyond the initial application.
- The design should be scalable and facilitate easy migration.

## Domain Modules, Components
[![](https://mermaid.ink/img/pako:eNp9km9vmzAQxr_Kya_TpIRCApoqZck6ddq0Spk0aaQvDtuANbCRbdqxNN99NvnX5MV4Yc7PPffznewtoYpxkpKiVq-0Qm3hx2ojwX_LbPH0CJ_R8lfs4XmvLuHm5v4N3mCVfVWlkLDm-kVQ_i4NayHLmsOis9UxDR8mub6H77LugaquZoCUcmM4g7yHBWscyYH3UTasB-JD9pPnz0NyuVdWe6ejZxdHrCnWnB3Khoz3uWkOoFPZlePhav8NRe0myA7_qwmP6iXsUb4Ii1YomZ1DWGq3fFH5safDmGC4tY5hBsjZf-nyeK00_q_2Y63o72WFQl634a1Uc3d5wJvW9gCd4dpcY8-AbFjPLXtAq-phVifRIcvQIqBkYNwpnXnXJRmRhusGBXOPaevhG2Ir3vANSV3IeIFdbTdkI3fOip1V615Sklrd8RHpWkfmK4GlxoakBdbmpH5iwip9EmuFjLvtlti-9S-3FMY6JFWyEKXXO107ubK2Nelk4tPjUtiqy8dUNRMjmH_m1UsST-JpPMdpyONZiFEYMpoHybyY3gUFm90GUyS73Yi0KD31D0nj2XgWhmEwi-IoSZIwGZGepNF8HNzFSXwbhXEYzOPI1fxVyo0RDMW_htjPufsHndoUBw?type=png)](https://mermaid.live/edit#pako:eNp9km9vmzAQxr_Kya_TpIRCApoqZck6ddq0Spk0aaQvDtuANbCRbdqxNN99NvnX5MV4Yc7PPffznewtoYpxkpKiVq-0Qm3hx2ojwX_LbPH0CJ_R8lfs4XmvLuHm5v4N3mCVfVWlkLDm-kVQ_i4NayHLmsOis9UxDR8mub6H77LugaquZoCUcmM4g7yHBWscyYH3UTasB-JD9pPnz0NyuVdWe6ejZxdHrCnWnB3Khoz3uWkOoFPZlePhav8NRe0myA7_qwmP6iXsUb4Ii1YomZ1DWGq3fFH5safDmGC4tY5hBsjZf-nyeK00_q_2Y63o72WFQl634a1Uc3d5wJvW9gCd4dpcY8-AbFjPLXtAq-phVifRIcvQIqBkYNwpnXnXJRmRhusGBXOPaevhG2Ir3vANSV3IeIFdbTdkI3fOip1V615Sklrd8RHpWkfmK4GlxoakBdbmpH5iwip9EmuFjLvtlti-9S-3FMY6JFWyEKXXO107ubK2Nelk4tPjUtiqy8dUNRMjmH_m1UsST-JpPMdpyONZiFEYMpoHybyY3gUFm90GUyS73Yi0KD31D0nj2XgWhmEwi-IoSZIwGZGepNF8HNzFSXwbhXEYzOPI1fxVyo0RDMW_htjPufsHndoUBw)

### API Gateway
- Receives client requests.
### Login Service
- Forwards the login request to the authentication service.
- Sends back a JWT token upon successful authentication.
### Invitation Service
- Manages pre-generated invitation codes.
- Runs a cron job to validate user codes against updated criteria.
### Auth Service
- Every request must be signed for added security.
- Provides authentication tokens, stored in Redis.
- Token format: {USERID}:{FUNCTION}:{AUTH_TOKEN}:{AUTH_STATE} for multi-stage authentication.
### Mailing Service
- Operates a worker pool to handle email requests from services such as the login and admin services.
- Can leverage AWS SES or other third-party SMTP solutions.
### Admin Service
- Especially for login authentication and config settings purposes.
### Chain Service
- Continuously polling data from the blockchain.
- Potentially scalable based on user ID; ensures atomic operations.
### Web Service
- Utilizes Server-Side Rendering (SSR) frontend frameworks to facilitate communication with the auth service.

# Actual mailing sequence diagram
[![](https://mermaid.ink/img/pako:eNrNVcFu2zAM_RVCl17SxLEbxzGGAFnbw4YOKJYCA4ZcFIu2hdqSK8npsqL_Pspx0qRdC-y25GJTfOTjI0U_sUwLZCmz-NCiyvBK8sLwGlbKSVchLFpXQq4fV4q3Tqu2XqNZKaDf4vZLwR0-8i2cz-fwHYW0KVyWmN2DbEAbkDlIBeuKk6WS1n2Es-jA-EeoNLl_WpvRHJyGxuAGlQPRNpXMCOcDl22B5E2MrbPw17A3upAqhSUqAdc1lxUlemj3HLrT4_QFpb81OkNr4U7foxrsGHCCb9DIfNtV44AIbnglxUmc8_lR-tR75pQRheeqtIOHliC5RDEg1q41CtAYOjPYaGXxHU72NafXfl1vLJqNzNC7E9delBf9vhERqQoSonPrtToCHmdsG-EVPkkK97gF68j-DvaUBc2Q2TYOWuuz7mjA9eXVcgG8KrSRrqx781oT8JEMwEVNJXmtdydZJannZ2Cp_XLjKRGJd9L3Bfb1uxI7CsSgj-Xpkw4eggY-Qz8uPewkgsAd947TDt2J7YlQmMFe1ZLGo5uYhm8rzUXHvLVI82F7HysLxanTOOgIUHV7X-Kyz_OGyfJ6CYdG2m42XjpJFaAvYfEG9g_dO0Kd9q0fRfg_m9zf5pLQlb_5GcoNHqb97b2gy8UPglBnzJmlHuTa6-hfLTi-rqg5j6XMSn-pC1RoyFtAbnRNzhtJqkmt9jw-uKRff9x1Oud0pf0A9gndqz2yW3HcauVzw93dzRviR0vkMABd6wmOfoutlP-zAavR0Lugzf3ko6wYJa5xxVJ6FJjztnIrtlLP5OoX93KrMpY60-KA7WTpFz1Lc17Zg_VaSKfNwegnFun1iblt4z8TBa1xCplplcvC21tTkbl0rrHpaOSPhwV1u10PM12PrBQlN67czOJRHMYJDyOMpxGfRJHI1uNZkocX41xMg3HI2fPzgDVc-ai_WDq-iIbBNElmkyCI48nFLBqwLUvDaDgNJ5NZcpEkQRxMw5hQv7WmQoLhLJxG4ygIZ9M4DpIw6eL97A596c9_AJPWS3s?type=png)](https://mermaid.live/edit#pako:eNrNVcFu2zAM_RVCl17SxLEbxzGGAFnbw4YOKJYCA4ZcFIu2hdqSK8npsqL_Pspx0qRdC-y25GJTfOTjI0U_sUwLZCmz-NCiyvBK8sLwGlbKSVchLFpXQq4fV4q3Tqu2XqNZKaDf4vZLwR0-8i2cz-fwHYW0KVyWmN2DbEAbkDlIBeuKk6WS1n2Es-jA-EeoNLl_WpvRHJyGxuAGlQPRNpXMCOcDl22B5E2MrbPw17A3upAqhSUqAdc1lxUlemj3HLrT4_QFpb81OkNr4U7foxrsGHCCb9DIfNtV44AIbnglxUmc8_lR-tR75pQRheeqtIOHliC5RDEg1q41CtAYOjPYaGXxHU72NafXfl1vLJqNzNC7E9delBf9vhERqQoSonPrtToCHmdsG-EVPkkK97gF68j-DvaUBc2Q2TYOWuuz7mjA9eXVcgG8KrSRrqx781oT8JEMwEVNJXmtdydZJannZ2Cp_XLjKRGJd9L3Bfb1uxI7CsSgj-Xpkw4eggY-Qz8uPewkgsAd947TDt2J7YlQmMFe1ZLGo5uYhm8rzUXHvLVI82F7HysLxanTOOgIUHV7X-Kyz_OGyfJ6CYdG2m42XjpJFaAvYfEG9g_dO0Kd9q0fRfg_m9zf5pLQlb_5GcoNHqb97b2gy8UPglBnzJmlHuTa6-hfLTi-rqg5j6XMSn-pC1RoyFtAbnRNzhtJqkmt9jw-uKRff9x1Oud0pf0A9gndqz2yW3HcauVzw93dzRviR0vkMABd6wmOfoutlP-zAavR0Lugzf3ko6wYJa5xxVJ6FJjztnIrtlLP5OoX93KrMpY60-KA7WTpFz1Lc17Zg_VaSKfNwegnFun1iblt4z8TBa1xCplplcvC21tTkbl0rrHpaOSPhwV1u10PM12PrBQlN67czOJRHMYJDyOMpxGfRJHI1uNZkocX41xMg3HI2fPzgDVc-ai_WDq-iIbBNElmkyCI48nFLBqwLUvDaDgNJ5NZcpEkQRxMw5hQv7WmQoLhLJxG4ygIZ9M4DpIw6eL97A596c9_AJPWS3s)
## Design database for storage usages
### Relational Database (RDB) for User Table
- RDB systems offer strong consistency and maintain ACID properties.
- Frequent querying and adjustments make an RDB a suitable choice. 
- AWS-related services are preferred for integration ease.
    
#### MySQL (Aurora DB)
- **Pros**: 
    - Faster recovery speed (according to AWS documentation).
    - Supports multiple replicas.
    - Excellent query speeds with parallel querying in extensive tables.
    - Efficient recovery rate and time.
- **Cons**:
    - Parallel queries can dramatically increase IOPS, potentially leading to higher costs.

#### Postgres (RDS)
- **Pros**: 
    - More cost-effective compared to Aurora.
    - Supports most SQL syntax.
- **Cons**:
    - Recovery time needs attention

**Ref.** https://aws.amazon.com/blogs/database/is-amazon-rds-for-postgresql-or-amazon-aurora-postgresql-a-better-choice-for-me/




#### NoSQL 
- **Redis(Elastic Cache)**:  
    - Powerful with an RDB mode for persistence.
- **Scylla DB** (Touted as the fastest NoSQL database with the potential for theoretically infinite scalability. )


### DB schema

```sql
  CREATE TABLE users (
    `id` bigint(20) PRIMARY KEY ,
    `email` varchar(30) COMMENT '',
    `password` varchar(64) COMMENT 'using bcrypt algo',
    `invitation_code` varchar(10),
    `on_chain_address` varchar(64) COMMENT 'mapping on chain data',
    `on_chain_data` JSON COMMENT 'It could also use NoSQL DB', 
    `state` tinyint(1) COMMENT '0: init 1: processing activation 
     2: activated 6: deactivated 9: suspend',
    `created_at` int(11), 
    `updated_at` int(11), 
    `integration_key` dedault '' COMMENT 'using as private key for ECDSA verification request between components', 
     ...
 )
 
  CREATE TABLE invite_codes (
    `id` bigint(20) PRIMARY KEY ,
    `invite_code` varchar(10),
    `referrer` varchar(6) COMMENT 'referrer of this invite_code',
    `remain_times` int(11),
    `update_version` int(11) COMMENT 'optimistic locking for update with timestamp'
    `state` tinyint(1) COMMENT '0: init 1: processing authentication 
     2: activated 6: deactivated 8: suspend',
    `prcessing_activity_ids` varchar(30) COMMENT 'string list for activate ids',
    `created_at` int(11), 
    `updated_at` int(11), 
     ...
 )
 
 CREATE TABLE invite_code_criteria (
    `id` bigint(20) PRIMARY KEY ,
    `invitation_code_id` bigint(20),
    `invitation_code` varchar(10),
    `state` tinyint(1) COMMENT '0: init 1: processing authentication 
     2: activated 6: deactivated 8: suspend',
    `created_at` int(11), 
    `updated_at` int(11), 
     ...
 )
 
 CREATE TABLE activity (
    `id` bigint(20) PRIMARY KEY ,
    `invitation_code_id` bigint(20),
    `invitation_code` varchar(10),
    `kind` tinyint(1) COMMENT '',
     ...
 )
 
  
  -- For api-gateway usage
  CREATE TABLE rate_limit ( 
      `id` bigint(20) PRIMARY KEY ,
      `route` varchar(20),
      `limit_count` int(11),
      ...
  )
 )
 ```


## Tech stacks
### Infra
- RDBMS
    - `AWS Aurora DB`, Chosen for its fast recovery speed and ability to support multiple replicas.
- NoSQL:
    - `Redis(Elastic cache)` Used as a caching service.
    - `Scylla DB` Preferred for its SQL-like query abilities.
#### Option1: EC2 with ELB
  - **props**
      - Quick deployment.
  - **cons**
      - Configuration settings can be more challenging compared to Kubernetes.
#### Option2: Ingress Nginx + EKS
  - **props**
      - User-friendly configuration settings, typically defined in .yaml format. 
      - Multiple smooth deployment strategies available.

## Backend
### API gateway
- Languages like Golang or Rust are suitable for handling heavy request loads.
- Scalable across multiple API gateway pods.
### Behind gRPC or Restful API
- Node.js: Enables rapid development.
- Golang: Known for its efficient concurrency patterns.


## Frontend

### Web development
- Use an SSR (Server Side Rendering) frontend framework.
- Implement encryption for credentials and cryptographic signatures.
- Options: React (Next.js), Vue (Nuxt.js), or other server-based frameworks.

### Authentication app (optional)
- Ability to communicate with hardware keys via Bluetooth, FIDO, and other protocols.


### Logs
 - For the service layer, ELK (or its fork, OpenSearch) is a solid choice.
 - Operational logs, such as login activities, can be stored in Scylla DB.

