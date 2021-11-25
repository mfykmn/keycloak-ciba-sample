# Keycloak CIBA Sample

## 環境構築
```bash
user@host: ~/workspace $ docker compose up -d
user@host: ~/workspace $ open http://localhost:8088/
```

`taro`というユーザーを手動で作成する

## CIBAフロー

```bash
$ AUTH_REQ_ID=$(curl -X POST \
   -H "Content-Type:application/x-www-form-urlencoded" \
   -H "Authorization:Basic dGVzdC1jbGllbnQ6MmFlNmQ4MWQtYmM3OC00MzRlLWE3YjgtOTg1OTIxZTBkNDdi" \
   -d "scope=openid" \
   -d "login_hint=taro" \
   -d "binding_message=hogehuga" \
 'http://localhost:8088/auth/realms/SampleRealm/protocol/openid-connect/ext/ciba/auth' | jq -r .auth_req_id)
$ docker logs authn-server
172.21.0.3 [31/Jul/2021:12:30:49 +0000] POST /status/201 HTTP/1.1 201 Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJOcVJYMjhaX3JsWC02eWFKTlJNbFh0bnZwNUxYRnl3aHUxU1pZaXlkeURvIn0.eyJleHAiOjE2Mjc3MzQ3NjgsImp0aSI6IjAyNDA2OWNhLTI2YmEtNGI5ZC1iNzIxLWQxYmYyMmY5MDRkMCIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6ODA4OC9hdXRoL3JlYWxtcy9TYW1wbGVSZWFsbSIsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6ODA4OC9hdXRoL3JlYWxtcy9TYW1wbGVSZWFsbSIsInN1YiI6IjE2ZTZiODYxLTkxN2UtNDJkOC1hOWJjLTYyNTZkZmRkYTFkZCIsInR5cCI6IkJlYXJlciIsImF6cCI6InRlc3QtY2xpZW50In0.mEhUPxtX-6N-oWkYY-svBzurgF-3q7fn_k20IyT5b-yceD5jk2mUyaNYMO34Qr7NrZmHqFNuK1txeY3TRYu3q2_8cIM9rr_A2OZpVElPRMcEX8pdQPggJIdYU1VKnMOGrWehLmo1nU7YDFJNvRcQVqF2quaK7QmhA5uxJFgfnPhxwHySVER7OR-X5zr8WridWigs6so-WaQOmwN8DxBTlPQuZW5M2DNBAE-Mo_t1WXTGKFsii4aV6qh-9TcycAXJ2K7HwjJAFKCGdtKlgyZcynHTiK-4iYt-2xA6CbkJzPCm_qJqXjbwLM-Cmy6xZdF3HIBuuJT3WhwP5VyAXIp64Q

$ CREDS=eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJOcVJYMjhaX3JsWC02eWFKTlJNbFh0bnZwNUxYRnl3aHUxU1pZaXlkeURvIn0.eyJleHAiOjE2Mjc3MzQ3NjgsImp0aSI6IjAyNDA2OWNhLTI2YmEtNGI5ZC1iNzIxLWQxYmYyMmY5MDRkMCIsImlzcyI6Imh0dHA6Ly9sb2NhbGhvc3Q6ODA4OC9hdXRoL3JlYWxtcy9TYW1wbGVSZWFsbSIsImF1ZCI6Imh0dHA6Ly9sb2NhbGhvc3Q6ODA4OC9hdXRoL3JlYWxtcy9TYW1wbGVSZWFsbSIsInN1YiI6IjE2ZTZiODYxLTkxN2UtNDJkOC1hOWJjLTYyNTZkZmRkYTFkZCIsInR5cCI6IkJlYXJlciIsImF6cCI6InRlc3QtY2xpZW50In0.mEhUPxtX-6N-oWkYY-svBzurgF-3q7fn_k20IyT5b-yceD5jk2mUyaNYMO34Qr7NrZmHqFNuK1txeY3TRYu3q2_8cIM9rr_A2OZpVElPRMcEX8pdQPggJIdYU1VKnMOGrWehLmo1nU7YDFJNvRcQVqF2quaK7QmhA5uxJFgfnPhxwHySVER7OR-X5zr8WridWigs6so-WaQOmwN8DxBTlPQuZW5M2DNBAE-Mo_t1WXTGKFsii4aV6qh-9TcycAXJ2K7HwjJAFKCGdtKlgyZcynHTiK-4iYt-2xA6CbkJzPCm_qJqXjbwLM-Cmy6xZdF3HIBuuJT3WhwP5VyAXIp64Q
```

###
```bash
$ curl -X POST \
   -H "Content-Type:application/x-www-form-urlencoded" \
   -H "Authorization:Basic dGVzdC1jbGllbnQ6MmFlNmQ4MWQtYmM3OC00MzRlLWE3YjgtOTg1OTIxZTBkNDdi" \
   -d "grant_type=urn:openid:params:grant-type:ciba" \
   -d "auth_req_id=${AUTH_REQ_ID}" \
 'http://localhost:8088/auth/realms/SampleRealm/protocol/openid-connect/token'
```
認証が成功している状態で行うとアクセストークンが返ってくる

### デバイス側で許可するときにkeycloakに渡るリクエストをCLIで渡す
```bash

$ curl -i -X POST \
   -H "Content-Type:application/json" \
   -H "Authorization:Bearer ${CREDS}" \
   -d '{"status":"SUCCEED"}' \
 'http://localhost:8088/auth/realms/SampleRealm/protocol/openid-connect/ext/ciba/auth/callback'

$ ACCESS_TOKEN=eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJOcVJYMjhaX3JsWC02eWFKTlJNbFh0bnZwNUxYRnl3aHUxU1pZaXlkeURvIn0.eyJleHAiOjE2Mjc3MzUwNjUsImlhdCI6MTYyNzczNDc2NSwianRpIjoiNmMzYTJhN2MtNzI1Yy00ODRhLTg4Y2QtOGU1NzAzOTIyNTQ3IiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDg4L2F1dGgvcmVhbG1zL1NhbXBsZVJlYWxtIiwic3ViIjoiMTZlNmI4NjEtOTE3ZS00MmQ4LWE5YmMtNjI1NmRmZGRhMWRkIiwidHlwIjoiQmVhcmVyIiwiYXpwIjoidGVzdC1jbGllbnQiLCJzZXNzaW9uX3N0YXRlIjoiZGRiYjNmY2YtMTFjOC00NzQ1LWJmMTktY2IyZGYxOWVkZWZjIiwiYWNyIjoiMSIsInNjb3BlIjoib3BlbmlkIn0.SiZuP_IWLif90_JUKAvjjLstxC4JnRRqXew-gZF9tdOniDvDHiKFp4kMQNQpx14psZwLaX1m_oG_Ca5RTqghPkPrX-LMRLz1dXXkNZSiVGLnjI11Ehf2fkMMvGILcwBAD2bC_adFIZBc3rwEWXNws8J8QLdNUKwQ_yN0p63KAbIAOP_YABacUohatg-LdZ2gmOddfGfTG_qTzAxEFS6Z5XTDm6OZD-Kgon48tdAi3bY1EP4qf5qmSIsb5HN0ePSNxl94vAnJZFwwRWEjfU5cEr1G537vZHIeskOQ-oXtpE02i-OIXhB30Q6COYL1GaPbhrhEDGfnMIi9BIwHv1BpFA","expires_in":300,"refresh_expires_in":1800,"refresh_token":"eyJhbGciOiJIUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICIwZWI1YjA0Yy02MmUwLTQyOTktYjdlNy1jNTJkYmIxZTUxN2EifQ.eyJleHAiOjE2Mjc3MzY1NjUsImlhdCI6MTYyNzczNDc2NSwianRpIjoiYzQxM2QwNmYtM2M1My00NDAzLWE5ZTUtOGZjMGU4ZTA4NTc4IiwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDg4L2F1dGgvcmVhbG1zL1NhbXBsZVJlYWxtIiwiYXVkIjoiaHR0cDovL2xvY2FsaG9zdDo4MDg4L2F1dGgvcmVhbG1zL1NhbXBsZVJlYWxtIiwic3ViIjoiMTZlNmI4NjEtOTE3ZS00MmQ4LWE5YmMtNjI1NmRmZGRhMWRkIiwidHlwIjoiUmVmcmVzaCIsImF6cCI6InRlc3QtY2xpZW50Iiwic2Vzc2lvbl9zdGF0ZSI6ImRkYmIzZmNmLTExYzgtNDc0NS1iZjE5LWNiMmRmMTllZGVmYyIsInNjb3BlIjoib3BlbmlkIn0.bxaFMjg1Dh3tJW8C8ySsiUDblhCucOXeuGqAepMB878","token_type":"Bearer","id_token":"eyJhbGciOiJSUzI1NiIsInR5cCIgOiAiSldUIiwia2lkIiA6ICJOcVJYMjhaX3JsWC02eWFKTlJNbFh0bnZwNUxYRnl3aHUxU1pZaXlkeURvIn0.eyJleHAiOjE2Mjc3MzUwNjUsImlhdCI6MTYyNzczNDc2NSwiYXV0aF90aW1lIjowLCJqdGkiOiJjYTRlNTg0My04MzEzLTRiNzMtODU3Yy1kNTZkNTkyZDMwNzEiLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwODgvYXV0aC9yZWFsbXMvU2FtcGxlUmVhbG0iLCJhdWQiOiJ0ZXN0LWNsaWVudCIsInN1YiI6IjE2ZTZiODYxLTkxN2UtNDJkOC1hOWJjLTYyNTZkZmRkYTFkZCIsInR5cCI6IklEIiwiYXpwIjoidGVzdC1jbGllbnQiLCJzZXNzaW9uX3N0YXRlIjoiZGRiYjNmY2YtMTFjOC00NzQ1LWJmMTktY2IyZGYxOWVkZWZjIiwiYXRfaGFzaCI6InhjRWhQTW9xXy1oejNPT1VGbGtaekEiLCJhY3IiOiIxIn0.nuhtZROTcoxtswyx619FwIE_57uDenFQBsjRXbNC8LzzCMVW9iiJIoT8CsN35QrACHa1nnGWyeZiI8rB4gJX74-nQ9PNW1Vj-ML9HlXGuXCc8nyXBbADyDdLaW8sykxIy45qaGQ2IGznhqRdyHpRipGZtrnd-rENsPgscp4DksgCMHYEp32u-FfFb50XtGmp8okcfu1nIxQ5kXMpPBlvV2ztwNlgxmC74GFhIFKIWWDYuRRVe5kRnHRwaWU_ZL1_D2rOQMU57YVAJmw3GOAKwBwD0dRN6sOLHxcGbIvig0Vr5mgt9hI1NKPA2yUhwES7ejw4UkuU7XX6ZUTVvuhlBw
```